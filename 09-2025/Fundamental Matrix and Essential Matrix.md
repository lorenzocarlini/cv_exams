---
tags:
    - computer-vision
    - epipolar-geometry
    - fundamental-matrix
    - essential-matrix
    - calibration
    - exam-prep
---
# Exercise 4. (8 points)
#### a) What is the relationship between the fundamental matrix F and the essential matrix E? Explain how one can be derived from the other.
#### b) Describe how the fundamental matrix F can be estimated from corresponding points in two images. What method(s) are commonly used for this computation?

---

### 4a. Relationship Between F and E

#### The Core Difference

Both matrices encode the epipolar geometry between two cameras, but they
differ in what they assume about the cameras:

|                  | Fundamental Matrix $F$                   | Essential Matrix $E$            |
| ---------------- | ---------------------------------------- | ------------------------------- |
| Camera knowledge | Works with **any** camera (uncalibrated) | Requires **calibrated** cameras |
| Coordinates      | Pixel (image) coordinates                | Normalised camera coordinates   |
| DOF              | 7                                        | 5                               |
| Rank             | 2                                        | 2                               |

The fundamental matrix works in raw pixel space. The essential matrix works
in a normalised coordinate system where the camera intrinsics have been
removed.

---

#### The Relationship

If $K$ and $K'$ are the intrinsic calibration matrices of camera 1 and
camera 2 respectively, then:

$$E = K'^T F K$$

and going the other way:

$$F = K'^{-T} E K^{-1}$$

The calibration matrix $K$ encodes the internal camera parameters (focal
length, principal point, skew):

$$K = \begin{bmatrix} f_x & 0 & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{bmatrix}$$

---

#### Intuition

A pixel coordinate $p$ in image space can be converted to a normalised
camera coordinate $\hat{p}$ by applying $K^{-1}$:

$$\hat{p} = K^{-1} p$$

The epipolar constraint in pixel space is $p'^T F p = 0$. Substituting
$p = K\hat{p}$ and $p' = K'\hat{p}'$:

$$(K'\hat{p}')^T F (K\hat{p}) = 0$$
$$\hat{p}'^T \underbrace{K'^T F K}_{E} \hat{p} = 0$$

So $E$ is simply $F$ expressed in normalised coordinates after removing the
effect of the camera intrinsics.

---

#### What E Gives You That F Does Not

Because $E$ operates in calibrated space, it encodes the **metric** relative
pose between the cameras: from $E$ you can recover the actual rotation $R$
and translation $t$ (up to scale) between the two cameras via SVD
decomposition. From $F$ alone you can only recover the projective geometry,
not the metric one.

---

### 4b. Estimating F From Corresponding Points

#### Setup

Each pair of corresponding points $(p_i, p'_i)$ must satisfy:

$$p'^T_i F p_i = 0$$

Expanding $p = (x, y, 1)^T$ and $p' = (x', y', 1)^T$, this gives one
linear equation in the 9 entries of $F$:

$$x'x f_{11} + x'y f_{12} + x' f_{13} + y'x f_{21} + y'y f_{22} + y' f_{23} + x f_{31} + y f_{32} + f_{33} = 0$$

Each correspondence contributes one such equation. Stacking $n$
correspondences gives:

$$A\mathbf{f} = 0$$

where $\mathbf{f}$ is the 9-entry vector of $F$ and $A$ is an $n \times 9$
matrix.

---

#### The Eight-Point Algorithm

The most common method. Requires at least 8 correspondences.

**Step 1: Normalise the points**
Scale and translate points so they have zero mean and average distance
$\sqrt{2}$ from the origin. This is critical for numerical stability.

$$\tilde{p} = T p \qquad \tilde{p}' = T' p'$$

**Step 2: Build the matrix $A$**
Stack one row per correspondence as shown above.

**Step 3: Solve via SVD**
Compute the SVD of $A = U \Sigma V^T$. The solution $\mathbf{f}$ is the
last column of $V$ (the right singular vector corresponding to the smallest
singular value).

**Step 4: Enforce rank-2 constraint**
The recovered $F$ will generally have rank 3. Set the smallest singular
value of $F$ to zero:

$$F = U \begin{bmatrix} \sigma_1 & 0 & 0 \\ 0 & \sigma_2 & 0 \\ 0 & 0 & 0 \end{bmatrix} V^T$$

**Step 5: Denormalise**
Undo the normalisation:

$$F = T'^T \tilde{F} T$$

---

#### RANSAC for Robustness

In practice, feature matches contain many outliers. The eight-point algorithm
is run inside a RANSAC loop:

```
Repeat N times:
    1. Sample 8 random correspondences
    2. Compute F using the eight-point algorithm
    3. Count inliers: points where |p'^T F p| < epsilon
    4. Keep the F with the largest inlier set

Final step: recompute F using all inliers of the best iteration
```

This makes the estimation robust to wrong matches.

---

### Exam-Ready Answer

> The essential matrix $E$ and fundamental matrix $F$ both encode epipolar
> geometry but differ in coordinate space. $F$ operates in raw pixel
> coordinates and requires no knowledge of the camera internals. $E$ operates
> in normalised camera coordinates and requires known calibration matrices $K$
> and $K'$. They are related by $E = K'^T F K$, meaning $E$ is simply $F$
> after removing the effect of the intrinsics. From $E$, the actual rotation
> and translation between cameras can be recovered via SVD; from $F$ alone
> only projective geometry is recoverable.
>
> $F$ is estimated from at least 8 point correspondences using the eight-point
> algorithm. Each correspondence provides one linear equation in the 9 entries
> of $F$; stacking these gives a homogeneous system $A\mathbf{f} = 0$ solved
> via SVD, where the solution is the right singular vector corresponding to
> the smallest singular value. The rank-2 constraint is then enforced by
> zeroing the smallest singular value of the resulting matrix. Point
> coordinates are normalised before this step for numerical stability.
> In practice, RANSAC wraps this computation to handle outlier matches,
> sampling 8 points at random, computing $F$, counting inliers by
> reprojection error, and keeping the best model.