## Exercise 4. (8 Points)
#### **Question:** Describe the process of recovering 3D structure from two calibrated images. Include: a) the role of camera calibration and intrinsic/extrinsic parameters, b) how triangulation is used to estimate the 3D position of scene points.

---

### a) Camera Calibration and Intrinsic/Extrinsic Parameters

#### Why Calibration is Needed

A camera projects 3D world points onto a 2D image plane. To reverse this
process and recover 3D structure, we need to know exactly how the camera
performs that projection. This is what calibration provides.

The full projection of a 3D point $P$ to an image point $p$ is modelled as:

$$
p = K [R \mid t] P
$$

where $K$ is the intrinsic matrix and $[R \mid t]$ encodes the extrinsic
parameters.

---

#### Intrinsic Parameters

The intrinsic matrix $K$ encodes the internal properties of the camera:

$$
K = \begin{bmatrix}
f_x & 0   & c_x \\
0   & f_y & c_y \\
0   & 0   & 1
\end{bmatrix}
$$

| Parameter | Meaning |
| --------- | --------------------------------------- |
| $f_x, f_y$ | Focal length in pixels (x and y) |
| $c_x, c_y$ | Principal point (image centre offset) |

Knowing $K$ allows us to convert pixel coordinates into **normalised camera
coordinates**, removing the effect of the camera internals:

$$
\hat{p} = K^{-1} p
$$

---

#### Extrinsic Parameters

The extrinsic parameters describe the position and orientation of the
camera in the world:

- $R$: $3 \times 3$ rotation matrix (camera orientation)
- $t$: $3 \times 1$ translation vector (camera position)

Together $[R \mid t]$ transforms a point from world coordinates into
camera coordinates. For two cameras, the relative rotation $R$ and
translation $t$ between them define the **baseline** of the stereo system.

---

#### From F to E Using Calibration

When both cameras are calibrated, the fundamental matrix $F$ can be
converted to the essential matrix $E$:

$$
E = K'^{\,T} F K
$$

From $E$, the relative rotation $R$ and translation $t$ between the two
cameras can be recovered via SVD (up to a scale factor on $t$). This
gives the full camera geometry needed for triangulation.

---

### b) Triangulation

#### Core Idea

Once we know the camera matrices $P_1 = K[I \mid 0]$ and
$P_2 = K'[R \mid t]$ for the two views, and we have a correspondence
$(p, p')$ between them, we can recover the 3D point $X$ by intersecting
the two **rays** back-projected from each camera through those image points.

```
Camera 1                    Camera 2
    o --------> ray 1           o --------> ray 2
                    \         /
                     \       /
                      X (3D point)
```

---

#### Why Rays Do Not Intersect Perfectly

In practice, due to noise in the detected keypoints and imperfect
calibration, the two rays are **skew** (they do not exactly meet in 3D).
The 3D point is therefore estimated as the point that **minimises the
distance** to both rays simultaneously.

---

#### Linear Triangulation (DLT)

Each image point gives two equations. Writing $p = P_1 X$ and
$p' = P_2 X$ in homogeneous coordinates and cross-multiplying to eliminate
the scale:

$$
p \times (P_1 X) = 0 \qquad p' \times (P_2 X) = 0
$$

This gives a system $AX = 0$ with 4 equations in 4 unknowns (homogeneous
3D coordinates of $X$). Solved via SVD: the solution is the last column
of $V$.

---

#### Full Pipeline Summary

```
1. Detect and match keypoints across both images (e.g. SIFT + RANSAC)
2. Estimate F from correspondences (eight-point algorithm)
3. Compute E = K'^T F K using known calibration
4. Decompose E via SVD to recover R and t
5. For each correspondence (p, p'):
       back-project rays from both cameras
       triangulate to find 3D point X
6. Output: sparse 3D point cloud of the scene
```

---

### Exam-Ready Answer

> Recovering 3D structure from two calibrated images requires knowing the
> camera projection model $p = K[R \mid t]P$. The intrinsic matrix $K$
> encodes focal length and principal point, and allows pixel coordinates to
> be converted into normalised camera coordinates. The extrinsic parameters
> $R$ and $t$ describe the relative pose between the two cameras. When
> calibration is known, the fundamental matrix $F$ estimated from point
> correspondences can be converted to the essential matrix $E = K'^T F K$,
> from which $R$ and $t$ are recovered via SVD.
>
> With the camera matrices fully known, 3D points are recovered by
> triangulation. For each pair of corresponding points $(p, p')$, a ray is
> back-projected from each camera through its image point. The 3D point $X$
> lies at the intersection of these two rays. In practice noise makes the
> rays skew, so the solution is found by solving a linear system $AX = 0$
> via SVD, minimising the distance to both rays simultaneously. This
> produces a sparse 3D point cloud of the scene from the two views.