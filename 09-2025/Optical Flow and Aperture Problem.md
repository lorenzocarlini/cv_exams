## Exercise 3 (8 Points)
#### Describe a method for estimating optical flow in image sequences. What is the aperture problem, and how does it affect motion estimation? In which

---
## Optical Flow Estimation

### What is Optical Flow?

Optical flow is the apparent per-pixel motion between two consecutive frames
of a video. For every pixel we want to estimate a velocity vector $(u, v)$
describing how that point moved from frame $t$ to frame $t+1$.

---

### Brightness Constancy Assumption

The core assumption: a pixel keeps the same intensity as it moves:

$$I(x, y, t) = I(x+u, y+v, t+1)$$

---

### The Optical Flow Constraint Equation

Taking a first-order Taylor expansion and assuming small motion:

$$I_x u + I_y v + I_t = 0$$

where:
- $I_x, I_y$ = spatial gradients
- $I_t$ = temporal gradient
- $(u, v)$ = unknown flow vector

**The problem:** one equation, two unknowns. Unsolvable at a single pixel.

---

### Lucas-Kanade: Solving the Ambiguity

Assume flow $(u, v)$ is constant within a small local window (e.g. 5x5).
This gives one equation per pixel in the window, all sharing the same unknowns,
solved via least squares:

$$M \begin{bmatrix} u \\ v \end{bmatrix} = -\begin{bmatrix} \sum I_x I_t \\ \sum I_y I_t \end{bmatrix}$$

where $M$ is the Harris structure matrix:

$$M = \begin{bmatrix} \sum I_x^2 & \sum I_x I_y \\ \sum I_x I_y & \sum I_y^2 \end{bmatrix}$$

The system is only reliably solvable when both eigenvalues of $M$ are large,
which happens at corners. This is exactly why corners are the best points
to track.

---

### The Aperture Problem

#### What it is

Imagine watching a moving edge through a small window. You can only see
the motion component **perpendicular to the edge**. The component **parallel
to the edge** is invisible because sliding along the edge produces no visible
change in the window.

```
   ─────────────
   ─────────────   sliding sideways looks IDENTICAL to no motion
   ─────────────   → that component is invisible
```

#### Why it happens mathematically

The constraint equation only fixes the component of $(u,v)$ along the
gradient direction $(I_x, I_y)$, i.e. perpendicular to the edge. Motion
parallel to the edge produces no change in $I_x$, $I_y$, or $I_t$, so it
vanishes from the equation entirely.

---

### Which Direction is Ambiguous?

Optical flow **cannot be reliably estimated along the direction tangent to
the edge**, i.e. parallel to the isophote (the line of constant intensity).
Only the component **along the gradient direction**, normal to the edge, can
be recovered.

| Eigenvalues of $M$ | Region | Flow estimation |
|---|---|---|
| Both large | Corner | Both components recoverable |
| One large, one small | Edge | Only normal component, aperture problem |
| Both small | Flat | Undefined, no information |

---

### Exam-Ready Answer

> Optical flow estimates the apparent per-pixel motion $(u,v)$ between
> consecutive frames using the brightness constancy assumption. Linearising
> this gives the constraint $I_x u + I_y v + I_t = 0$, which is one equation
> in two unknowns and is therefore underdetermined at any single pixel.
>
> The Lucas-Kanade method resolves this by assuming constant flow within a
> local window, producing an overdetermined system solved via least squares
> using the structure matrix $M$. The system is only reliable where both
> eigenvalues of $M$ are large, i.e. at corners.
>
> The aperture problem arises because the constraint equation only fixes the
> component of motion perpendicular to the local edge. The component parallel
> to the edge produces no measurable intensity change and is therefore
> invisible. Optical flow is consequently ambiguous along the direction
> tangent to edges, and can only be fully recovered at corner-like points
> where intensity changes significantly in both directions.