---
tags:
  - computer-vision
  - image-processing
  - edge-detection
  - sobel
  - canny
  - exam-prep
---
# Exercise 1. (8 points)
#### Describe the Sobel filter for edge detection. How does it differ from the Canny edge detector?

## What is an edge?
An edge is a place in the image where intensity changes sharply.

## The Sobel filter
It approximates the image gradient in both directions using two simple 3x3 kernels:
$$
S_x =
\begin{bmatrix}
-1 & 0 & 1 \\
-2 & 0 & 2 \\
-1 & 0 & 1
\end{bmatrix}
\qquad
S_y =
\begin{bmatrix}
-1 & -2 & -1 \\
0 & 0 & 0 \\
1 & 2 & 1
\end{bmatrix}
$$
- $S_x$ detects **vertical edges** (horizontal intensity change)
- $S_y$ detects **horizontal edges** (vertical intensity change)

The central row/column has weight 2: this is a mild built-in Gaussian smoothing, making Sobel slightly more robust to noise than a plain derivative filter.

## How to use it
Convolve the image with each kernel:

$$
G_x = Im * S_x
\qquad
G_y = Im * S_y
$$

Then combine into a single **gradient magnitude**:

$$
|G| = \sqrt{G_x^2 + G_y^2}
$$

(or approximated as $|G_x| + |G_y|$)

And optionally the **gradient direction**:

$$
\theta = \arctan\left(\frac{G_y}{G_x}\right)
$$

## What Sobel gives you

A **gradient magnitude image**: each pixel value tells you "how much of an edge is here." To get actual edges, you threshold this map: pixels above a threshold are edges, the rest are not.

## The problem with Sobel alone

- Edges come out **thick** (multiple adjacent pixels all have high gradient)
- Sensitive to noise (a noisy flat region can produce false edge responses)
- The threshold is a hard on/off decision with **no refinement**
## The Canny edge detector

Canny is a **multi-step pipeline** that builds on the gradient idea but fixes all of Sobel's weaknesses. The goal: edges that are **thin, accurate, and complete**.

### Step 1: Gaussian smoothing

Before computing any gradients, blur the image with a Gaussian filter $(G_{sigma})$:

$$
I' = G_\sigma * Im
$$

This removes noise first, so the gradient step doesn't amplify it.

> Sobel has mild built-in smoothing; Canny makes this explicit and controllable via $\sigma$.

### Step 2: Compute gradients (like Sobel)

Apply Sobel (or similar) to $I'$ to get magnitude $|G|$ and direction $theta$ at every pixel.

### Step 3: Non-maximum suppression (NMS)

This is what makes edges thin.

For each pixel, look at its two neighbors along the gradient direction. If the current pixel is not the strongest of the three, suppress it (set to 0):

```text
If pixel is not a local maximum along gradient direction -> set to 0
```

Result: only the single sharpest pixel along each edge survives: 1 pixel thin edges.

### Step 4: Double thresholding

Instead of one threshold, use two:

- High threshold $(T_H)$: pixels definitely are edges (strong edges).
- Low threshold ($T_L$): pixels might be edges (weak edges).
- Below ($T_L$): definitely not edges -> discard.

$$
T_L < T_H \qquad \text{(typically } T_H \approx 2\text{–}3 \times T_L \text{)}
$$

### Step 5: Edge tracking by hysteresis

The final step connects weak edges to strong edges:

- Start from every strong edge pixel.
- Walk along connected weak edge pixels and keep them (they're part of a real edge).
- Discard any weak pixels not connected to a strong one (they're likely noise).

```text
Strong edge pixel --connected--> Weak pixel -> KEEP (real edge)
Weak pixel -> DISCARD (isolated noise)
```

This prevents broken edges and removes isolated noise spikes.

## Sobel vs Canny: Direct comparison

| Property | Sobel | Canny |
|---|---|---|
| Output | Gradient magnitude image | Binary edge map |
| Edge thickness | Thick (multiple pixels) | 1 pixel thin |
| Noise handling | Mild (built-in smoothing) | Explicit Gaussian pre-smoothing |
| Thresholding | Single threshold | Double threshold + hysteresis |
| Edge connectivity | No | Yes (hysteresis links edges) |
| Complexity | Simple, fast | Multi-step, slower |
| Accuracy | Lower | Higher |

## Exam-ready answer

> The Sobel filter detects edges by computing the image gradient using two 3×3 kernels, one for horizontal changes ($S_x$) and one for vertical ($S_y$). Combining them gives a gradient magnitude ($|G| = \sqrt{G_x^2 + G_y^2}$), where high values indicate edges. Pixels above a threshold are marked as edges.
>
> Its main weaknesses are that edges come out thick, and the single threshold is brittle. Canny improves on this with a four-step pipeline: first it smooths the image with a Gaussian to remove noise; then it computes gradients like Sobel; then it applies non-maximum suppression to thin edges down to one pixel by keeping only local maxima along the gradient direction; finally it uses a double threshold with hysteresis. Strong edges are kept, weak edges are kept only if connected to a strong one, otherwise they are discarded as noise.
>
> The key differences are that Sobel is a single filter producing a thick gradient map, while Canny is a full pipeline producing thin, clean, binary edges with better noise robustness.



