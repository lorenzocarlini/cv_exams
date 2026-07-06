## Exercise 1. (8 Points)
#### Discuss the role of image derivatives in edge and corner detection. In your answer, explain:
#### a) The meaning of first-order and second-order image derivatives
#### b) How gradient magnitude and gradient direction are used for edge detection
#### c) How second-order derivatives can be used to identify intensity transitions
#### d) Why are derivative-based methods sensitive to noise and how can mitigate that.

---

### a) First-Order and Second-Order Derivatives

#### First-Order Derivatives

The first-order derivative of an image measures how fast the intensity
changes at each pixel. In 2D this means computing partial derivatives in
both spatial directions:

$$
I_x = \frac{\partial I}{\partial x} \qquad I_y = \frac{\partial I}{\partial y}
$$

In practice, since images are discrete, derivatives are approximated by
finite differences or convolution with a filter (e.g. Sobel):

$$
I_x \approx I(x+1, y) - I(x-1, y)
$$

A large first-order derivative means intensity is changing rapidly at that
point, which is the signature of an edge.

---

#### Second-Order Derivatives

The second-order derivative measures how fast the first derivative itself
is changing:

$$
I_{xx} = \frac{\partial^2 I}{\partial x^2} \qquad
I_{yy} = \frac{\partial^2 I}{\partial y^2} \qquad
I_{xy} = \frac{\partial^2 I}{\partial x \partial y}
$$

The most common second-order operator is the **Laplacian**:

$$
\nabla^2 I = I_{xx} + I_{yy}
$$

While the first derivative is large at an edge, the second derivative
**crosses zero** at the edge peak. These zero crossings pinpoint the
exact location of the edge more precisely than thresholding the first
derivative alone.

---

### b) Gradient Magnitude and Direction for Edge Detection

From the first-order derivatives, two quantities are computed at every
pixel:

**Gradient magnitude:** how strong the intensity change is

$$
|G| = \sqrt{I_x^2 + I_y^2}
$$

**Gradient direction:** which direction the change is strongest

$$
\theta = \arctan\left(\frac{I_y}{I_x}\right)
$$

The gradient direction is always **perpendicular to the edge**. This is
used in Canny's non-maximum suppression step: a pixel is only kept as an
edge if it is the local maximum along the gradient direction, thinning
edges to one pixel wide.

| Value | Meaning |
| ---------------- | ------------------------------ |
| Large $|G|$ | Strong edge present |
| Small $|G|$ | Flat region, no edge |
| $\theta$ | Orientation of the edge normal |

---

### c) Second-Order Derivatives for Identifying Intensity Transitions

Second-order derivatives are more sensitive to rapid intensity changes
and can locate edges more precisely via **zero crossings**.

At an edge, the intensity profile looks like a ramp. The first derivative
shows a peak at the edge. The second derivative shows a zero crossing
exactly where the peak is:

```
Intensity:     ___/‾‾‾
First deriv:   __/\___     <- peak at edge
Second deriv:  _/‾\___     <- zero crossing at edge
                 ^
              exact edge location
```

The **Laplacian of Gaussian (LoG)** combines Gaussian smoothing with the
Laplacian to detect these zero crossings robustly:

$$
\text{LoG}(x,y) = \nabla^2 (G_\sigma * I)
$$

Second-order derivatives are also at the core of **corner detection**.
The **Hessian matrix** collects all second-order derivatives:

$$
H = \begin{bmatrix} I_{xx} & I_{xy} \\ I_{xy} & I_{yy} \end{bmatrix}
$$

The eigenvalues of $H$ describe the curvature of the intensity surface in
all directions. Large curvature in both directions indicates a corner.
This is the basis of blob and corner detectors like the Hessian detector
used in SURF.

---

### d) Sensitivity to Noise and Mitigation

#### Why Derivatives Are Sensitive to Noise

Differentiation amplifies high-frequency content. Noise in an image
consists of random intensity fluctuations at high spatial frequencies.
Taking a derivative makes these fluctuations larger, potentially creating
false edge responses where the image is actually flat.

Second-order derivatives are even more sensitive: differentiating twice
amplifies noise further, making the Laplacian unreliable on raw images.

---

#### How to Mitigate It

The standard solution is to **smooth the image before differentiating**,
typically with a Gaussian filter:

$$
I' = G_\sigma * I \qquad \text{then compute } \nabla I'
$$

This works because smoothing suppresses high-frequency noise before
derivatives are taken. The parameter $\sigma$ controls the tradeoff:

| $\sigma$ | Effect |
| --------- | ---------------------------------------------- |
| Small | Less smoothing, preserves fine detail, more noise |
| Large | More smoothing, robust to noise, blurs fine edges |

The **Gaussian and the derivative can be combined into a single
convolution** by differentiating the Gaussian kernel itself:

$$
\frac{\partial}{\partial x}(G_\sigma * I) = \left(\frac{\partial G_\sigma}{\partial x}\right) * I
$$

This is computationally efficient and is exactly what Canny does in its
first step. The LoG operator applies the same idea to second-order
derivatives.

---

### Exam-Ready Answer

> First-order image derivatives $I_x$ and $I_y$ measure how fast intensity
> changes spatially. Their magnitude $|G| = \sqrt{I_x^2 + I_y^2}$ indicates
> edge strength and their direction $\theta = \arctan(I_y / I_x)$ gives the
> edge orientation, perpendicular to the edge itself. Thresholding the
> gradient magnitude is the basis of edge detection.
>
> Second-order derivatives measure the rate of change of the first
> derivative. The Laplacian $\nabla^2 I = I_{xx} + I_{yy}$ produces zero
> crossings exactly at edge locations, allowing more precise localisation
> than first-order methods. The Hessian matrix of second-order derivatives
> captures intensity curvature in all directions and is used to detect
> corners and blobs: large curvature in both directions signals a corner.

>The Laplacian of Gaussian (LoG) filter is mathematically equivalent to the **second derivative of a Gaussian function**.
>
> Derivative operators are sensitive to noise because differentiation
> amplifies high-frequency content, and noise lives at high frequencies.
> The standard mitigation is to smooth the image with a Gaussian before
> differentiating, controlling the tradeoff between noise robustness and
> edge sharpness via $\sigma$. Since convolution is associative, the
> Gaussian and the derivative kernel can be merged into a single filter,
> which is the approach used by Canny and the LoG detector.