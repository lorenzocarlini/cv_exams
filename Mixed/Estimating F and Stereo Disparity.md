markdown


## Estimating F and Stereo Disparity

**Question:** How can we estimate the Fundamental Matrix? Describe the
process of computing a disparity map and its relation to depth.

---

### Estimating the Fundamental Matrix

#### Setup

The fundamental matrix $F$ satisfies the epipolar constraint for any pair
of corresponding points $p$ and $p'$ across two images:

$$
p'^{\,T} F\, p = 0
$$

To estimate $F$, we need at least **8 point correspondences**.

---

#### The Normalised Eight-Point Algorithm (Hartley, 1995)

##### Step 1: Normalise the Points

Centre the image data at the origin and scale it so that the **mean squared
distance between the origin and the data points is 2 pixels**. If $T$ and
$T'$ are the normalising transformations for image 1 and image 2:

$$
\tilde{p} = T p \qquad \tilde{p}' = T' p'
$$

This step is critical for numerical stability. Without it, the system
$A\mathbf{f} = 0$ is poorly conditioned and the solution is unreliable.

---

##### Step 2: Build the System of Equations

Each normalised correspondence $(\tilde{p}_i, \tilde{p}'_i)$ with
$\tilde{p} = (x, y, 1)^T$ and $\tilde{p}' = (x', y', 1)^T$ expands the
epipolar constraint into one linear equation in the 9 entries of $F$:

$$
x'x\, f_{11} + x'y\, f_{12} + x'\, f_{13} +
y'x\, f_{21} + y'y\, f_{22} + y'\, f_{23} +
x\, f_{31} + y\, f_{32} + f_{33} = 0
$$

Stacking $n \geq 8$ correspondences gives a homogeneous system:

$$
A\mathbf{f} = 0
$$

---

##### Step 3: Solve via SVD

Compute $A = U \Sigma V^T$. The solution $\mathbf{f}$ is the last column
of $V$, corresponding to the smallest singular value. Reshape into a
$3 \times 3$ matrix $\tilde{F}$.

---

##### Step 4: Enforce the Rank-2 Constraint

The recovered $\tilde{F}$ will generally have rank 3. Take the SVD of
$\tilde{F}$ and zero the smallest singular value:

$$
\tilde{F} = U
\begin{bmatrix}
\sigma_1 & 0 & 0 \\
0 & \sigma_2 & 0 \\
0 & 0 & 0
\end{bmatrix}
V^T
$$

This enforces $\det(F) = 0$, which is required for the epipoles to exist.

---

##### Step 5: Denormalise

Transform the fundamental matrix back to original image coordinates using
the normalising transformations $T$ and $T'$:

$$
F = T'^{\,T}\, \tilde{F}\, T
$$

---

#### RANSAC for Robustness

In practice feature matches contain outliers. RANSAC wraps the normalised
eight-point algorithm:

```
Repeat N times:
    1. Sample 8 random correspondences
    2. Run the normalised eight-point algorithm to get F
    3. Count inliers: pairs where |p'^T F p| < epsilon
    4. Keep F with the largest inlier set

Final step: recompute F using all inliers of the best iteration
```

---

### Computing a Disparity Map

#### Setup

Two cameras are placed side by side with a known **baseline** $b$ and
shared **focal length** $f$. For any 3D point, it appears at slightly
different horizontal positions in the two images. This shift is the
**disparity**.

---

#### Step 1: Rectification

Warp both images so that epipolar lines become perfectly horizontal. After
rectification, corresponding points always lie on the **same row** in both
images, reducing the search from 2D to 1D.

---

#### Step 2: Correspondence Search

For each pixel $(x, y)$ in the left image, scan along row $y$ in the right
image to find the best matching patch. Similarity is measured using:

- **SSD** (Sum of Squared Differences)
- **SAD** (Sum of Absolute Differences)
- **NCC** (Normalised Cross-Correlation)

---

#### Step 3: Compute Disparity

$$
d(x, y) = x_{\text{left}} - x_{\text{right}}
$$

Repeat for every pixel to obtain the full **disparity map**: an image
where each pixel value encodes its horizontal shift between the two views.

```
Large disparity  ->  object is close
Small disparity  ->  object is far
```

---

#### Step 4: Recover Depth

$$
Z = \frac{f \cdot b}{d}
$$

| Symbol | Meaning                          |
| ------ | -------------------------------- |
| $Z$    | Depth (distance from camera)     |
| $f$    | Focal length                     |
| $b$    | Baseline between the two cameras |
| $d$    | Disparity                        |

Depth is **inversely proportional** to disparity. This is the same
principle the human visual system uses with two eyes.

---

#### Common Failure Cases

| Problem             | Cause                                             |
| ------------------- | ------------------------------------------------- |
| Textureless regions | No reliable patch match, disparity undefined      |
| Occlusions          | Some areas visible in one image but not the other |
| Reflective surfaces | Appearance changes between the two views          |

---

### Exam-Ready Answer

> The fundamental matrix (F, extrinsic data) is estimated using the normalised eight-point
> algorithm (Hartley, 1995). 
> > Center the image data at the origin, and scale it so the mean squared distance between the origin and the data points is 2 pixels
	- Use the eight-point algorithm to compute **F** from the normalized points
	- Enforce the rank-2 constraint (for example, take SVD of **F** and throw out the smallest singular value)
	-Transform fundamental matrix back to original units: if **T** and **T'** are the normalizing transformations in the two images, then the fundamental matrix in original coordinates is **T'T^T T F T
>
> A disparity map is computed from a rectified stereo pair by searching
> along each scanline for the best-matching patch between the left and right
> images. 
> The disparity $d = x_{\text{left}} - x_{\text{right}}$ is  recorded for every pixel. Depth is then recovered via 
> $Z = fb/d$, where $f$ is the focal length and $b$ is the baseline. Large disparity means the
> object is close; small disparity means it is far. Failure cases include
> textureless regions, occlusions, and reflective surfaces.