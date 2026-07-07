## Exercise 2. (8 Points)

**Question:** Describe the Harris corner detector. Explain: a) the
intuition behind detecting corners using local intensity variations, b)
the role of the second-moment matrix, c) how the corner response function
is computed and interpreted, d) whether the detector is robust to
rotation, intensity changes, and scale changes.

---

### a) Intuition Behind Corner Detection

A corner is a point where intensity changes significantly in **two
directions at once**. The key idea is to ask: if I shift a small window
around this pixel, how much does the image content change?

```
Flat region:   shift in any direction   -> no change   -> not interesting
Edge:          shift along edge         -> no change   -> ambiguous
               shift across edge        -> large change
Corner:        shift in any direction   -> large change -> uniquely localizable
```

This is formalised by the **auto-correlation function**, which measures
the intensity change $E(u,v)$ when a window is shifted by $(u,v)$:

$$
E(u,v) = \sum_{x,y} w(x,y)\, [I(x+u, y+v) - I(x,y)]^2
$$

where $w(x,y)$ is a Gaussian weighting window. For small shifts, a
first-order Taylor expansion gives:

$$
E(u,v) \approx \begin{bmatrix} u & v \end{bmatrix} M \begin{bmatrix} u \\ v \end{bmatrix}
$$

where $M$ is the second-moment matrix.

---

### b) The Second-Moment Matrix

The second-moment matrix (also called the structure tensor) is:

$$
M = \sum_{x,y} w(x,y)
\begin{bmatrix}
I_x^2 & I_x I_y \\
I_x I_y & I_y^2
\end{bmatrix}
$$

where $I_x$ and $I_y$ are the image gradients. The eigenvalues
$\lambda_1$ and $\lambda_2$ of $M$ describe the intensity variation along
the two principal directions of the local window:

| Eigenvalues | Interpretation |
| ------------------------------ | --------------- |
| Both small | Flat region |
| One large, one small | Edge |
| Both large | Corner |

The eigenvectors give the directions of maximum and minimum intensity
change; the eigenvalues give the magnitude of change in those directions.
A corner has large curvature in both directions, meaning $E(u,v)$ is
large for shifts in any direction.

---

### c) Corner Response Function

To avoid computing eigenvalues explicitly (expensive), Harris proposed the
**corner response function**:

$$
R = \det(M) - k\, (\text{trace}(M))^2
$$

where:

$$
\det(M) = \lambda_1 \lambda_2 \qquad
\text{trace}(M) = \lambda_1 + \lambda_2 \qquad
k \approx 0.04\text{--}0.06
$$

#### Interpretation

| Value of $R$ | Region type |
| -------------- | ----------- |
| $R \approx 0$ | Flat region (both eigenvalues small) |
| $R < 0$ | Edge (one eigenvalue dominates) |
| $R \gg 0$ | Corner (both eigenvalues large) |

#### Selection Steps

**Step 1: Threshold**
Keep only pixels where $R > \tau$ for some threshold $\tau$.

**Step 2: Non-maximum suppression**
Among surviving pixels, keep only local maxima of $R$ within a
neighbourhood. This avoids clusters of duplicate detections on the same
corner.

---

### d) Robustness Analysis

#### Rotation

**Fully robust.**

$M$ transforms under rotation as $M' = R_\theta M R_\theta^T$, which is a
similarity transformation. Similarity transformations preserve eigenvalues,
so $\det(M)$ and $\text{trace}(M)$ are unchanged:

$$
\det(M') = \det(M) \qquad \text{trace}(M') = \text{trace}(M) \implies R' = R
$$

The response $R$ is identical after rotation. Only the eigenvectors
(principal directions) rotate, not the eigenvalues.

---

#### Intensity Shifts (Additive: $I' = I + b$)

**Fully robust.**

Adding a constant $b$ does not change the gradient:

$$
I'_x = \frac{\partial(I+b)}{\partial x} = I_x \qquad I'_y = I_y
$$

So $M$ is completely unchanged and $R$ is identical.

---

#### Intensity Scaling (Multiplicative: $I' = aI$)

**Robust in location, not in threshold.**

Gradients scale by $a$, so $M$ scales by $a^2$:

$$
\det(M') = a^4 \det(M) \qquad \text{trace}(M') = a^2\, \text{trace}(M)
$$

$$
R' = a^4 R
$$

Since $a^4 > 0$ the relative ordering of corner strengths is preserved, so
corner **locations** are unchanged. However a fixed threshold $\tau$ is no
longer valid since all responses are scaled uniformly.

---

#### Scale Changes

**Not robust.**

Harris operates at a single fixed window size. A corner that is clearly
detectable at one scale may look like a flat region or an edge at a
different scale. This is the main limitation of Harris and the motivation
for scale-invariant detectors like SIFT, which detect corners across a
full Gaussian pyramid.

---

### Exam-Ready Answer

> The Harris detector finds corners by measuring how much image content
> changes when a small window is shifted in any direction. It builds the
> second-moment matrix $M$ from the squared and cross gradients $I_x^2$,
> $I_y^2$, $I_xI_y$ summed over a local window. The eigenvalues of $M$
> describe intensity variation along the principal directions: both large
> means corner, one large means edge, both small means flat region.
>
> To avoid computing eigenvalues, the corner response $R = \det(M) - k\,\text{trace}(M)^2$ is used. Pixels with $R \gg 0$ are corners, $R < 0$  are edges, $R \approx 0$ are flat. Non-maximum suppression is applied to  keep only the strongest local peaks.
>
> Harris is fully robust to rotation because eigenvalues are invariant under
> similarity transformations. It is fully robust to additive intensity shifts
> because derivatives of a constant are zero. It is robust to multiplicative
> intensity scaling in terms of corner locations (responses scale by $a^4$,
> preserving ranking) but a fixed threshold is no longer valid. It is not
> robust to scale changes since it operates at a single fixed window size,
> which is the main motivation for multi-scale detectors like SIFT.