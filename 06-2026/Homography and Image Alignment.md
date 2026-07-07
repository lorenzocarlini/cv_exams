## Exercise 3. (8 Points)

#### **Question:** Explain the concept of homography and its use in image alignment. 
#### Discuss: a) what a homography matrix represents geometrically,b) under which assumptions a homography can correctly map points between two images, c) how corresponding feature points can be used to estimate a homography.

---

### a) What a Homography Represents Geometrically

A homography is a $3 \times 3$ matrix $H$ that maps points from one image
plane to another under **projective transformation**. Given a point $p$ in
image 1, its corresponding point $p'$ in image 2 is:

$$
p' = H\, p
$$

In homogeneous coordinates with $p = (x, y, 1)^T$:

$$
\begin{bmatrix} x' \\ y' \\ w' \end{bmatrix}
=
\begin{bmatrix}
h_{11} & h_{12} & h_{13} \\
h_{21} & h_{22} & h_{23} \\
h_{31} & h_{32} & h_{33}
\end{bmatrix}
\begin{bmatrix} x \\ y \\ 1 \end{bmatrix}
$$

The actual image coordinates are recovered by dividing through by $w'$:

$$
x'_{\text{actual}} = \frac{x'}{w'} \qquad y'_{\text{actual}} = \frac{y'}{w'}
$$

Geometrically, $H$ can represent any combination of:

- **Translation** and **rotation** (rigid motion)
- **Scale** and **shear** (affine transforms)
- **Perspective distortion** (the full projective case)

$H$ has 8 degrees of freedom (9 entries, defined up to scale), so it can
warp one image plane onto another with full perspective correction.

---

### b) When a Homography Correctly Maps Points

A homography is an exact model only under specific geometric conditions.
It correctly maps all points between two images when either:

**Condition 1: The scene is planar**
All observed points lie on a single flat surface (e.g. a wall, a floor,
a document). In this case every point satisfies a single projective
relationship between the two views.

**Condition 2: Pure camera rotation (no translation)**
The camera rotates about its optical centre with no translation between
the two views. In this case all points, regardless of their depth, map
correctly through a single $H$.

If neither condition holds, that is if the scene has depth variation and
the camera also translates, then a single homography cannot correctly map
all points simultaneously. Different depths would require different
homographies, which is why general 3D scenes require the fundamental
matrix instead.

---

### c) Estimating a Homography from Corresponding Points

#### Minimum Requirements

$H$ has 8 degrees of freedom, so a minimum of **4 point correspondences**
are needed (each correspondence gives 2 equations).

---

#### Setting Up the Linear System

Each correspondence $(p_i, p'_i)$ with $p = (x, y, 1)^T$ and
$p' = (x', y', 1)^T$ expands into two linear equations in the entries
of $H$:

$$
x' = \frac{h_{11}x + h_{12}y + h_{13}}{h_{31}x + h_{32}y + h_{33}}
\qquad
y' = \frac{h_{21}x + h_{22}y + h_{23}}{h_{31}x + h_{32}y + h_{33}}
$$

Rearranging into a homogeneous form stacks into:

$$
A\mathbf{h} = 0
$$

where $\mathbf{h}$ is the 9-entry vector of $H$ and $A$ is a $2n \times 9$
matrix.

---

#### Solving via SVD

The solution $\mathbf{h}$ is the last column of $V$ from the SVD of $A$,
corresponding to the smallest singular value. Reshape into the $3 \times 3$
matrix $H$.

---

#### RANSAC for Robustness

In practice, feature matches (e.g. from SIFT) contain outliers. RANSAC
wraps the estimation:

```
Repeat N times:
    1. Sample 4 random correspondences
    2. Compute H via the linear system and SVD
    3. Count inliers: points where ||p' - Hp|| < epsilon
    4. Keep H with the largest inlier set

Final step: recompute H using all inliers of the best iteration
```

This makes $H$ robust to wrong matches.

---

#### Use in Image Alignment

Once $H$ is estimated, one image can be warped onto the other using
inverse mapping: for each pixel in the output image, apply $H^{-1}$ to
find the corresponding source pixel and interpolate its value. This is
the basis of:

- **Image stitching** (panoramas): align multiple images of a scene
  taken with camera rotation
- **Document rectification**: flatten a photographed flat document
- **Augmented reality**: project a texture onto a detected planar surface

---

### Exam-Ready Answer

> A homography is a $3 \times 3$ projective transformation matrix $H$ that
> maps points from one image to another via $p' = Hp$. It has 8 degrees of
> freedom and can represent any combination of translation, rotation, scale,
> and perspective distortion between two image planes.
>
> A homography correctly maps all points between two images under two
> conditions: when the scene is planar (all points lie on a flat surface),
> or when the camera undergoes pure rotation with no translation. If the
> scene has depth variation and the camera also translates, a single
> homography is not sufficient.
>
> To estimate $H$, at least 4 point correspondences are needed. Each
> correspondence provides 2 linear equations in the 9 entries of $H$,
> forming a homogeneous system $A\mathbf{h} = 0$ solved via SVD. In
> practice, correspondences come from feature matching (e.g. SIFT) and
> RANSAC is used to robustly estimate $H$ by repeatedly sampling 4 random
> matches, computing $H$, and keeping the model with the largest set of
> geometrically consistent inliers.
> ( We are using DIrect Linear Transformation (DLT) with a minimum of 4 correspondences )