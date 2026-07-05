# Exercise 3. (8 points)
#### a. Explain the concept of fundamental matrix. How is it computed, and what information does it convey about the relationship between two images? 
#### b. What is the geometric meaning of the epipoles in the two images? How are they related to the fundamental matrix (algebraically)?



### 3a. The Fundamental Matrix

#### Core Idea

When two cameras look at the same scene from different positions, a point
visible in both images must obey a precise geometric constraint. The
**fundamental matrix** $F$ is a $3 \times 3$ matrix that encodes this
constraint purely from geometry, without needing to know where the 3D point
actually is.

Given a point $p$ in image 1 and its correspondence $p'$ in image 2:

$$
p'^{\,T} F\, p = 0
$$

This means $F$ does not directly tell you where the match is, but it tells
you where it must lie: on a specific line in image 2 called the
**epipolar line**:

$$
l' = F\, p
$$

This reduces correspondence search from a 2D problem to a 1D search along
a line, which is very useful for stereo matching.

---

#### What Information Does F Convey?

- The **relative geometry** between the two cameras (rotation and
  translation), encoded implicitly
- For every point in image 1, the **epipolar line** in image 2 where its
  match must lie
- It works without knowing the internal camera parameters (focal length,
  sensor size etc.), making it a purely projective relationship

---

#### How is F Computed?

You need at least **8 point correspondences** between the two images.
Each pair $(p_i, p'_i)$ gives one linear equation in the 9 entries of $F$.
Stacking 8 or more gives a homogeneous system:

$$
A\mathbf{f} = 0
$$

solved via **SVD**, where the solution is the last column of $V$.

The **rank-2 constraint** is then enforced by zeroing the smallest singular
value of the result:

$$
F = U
\begin{bmatrix}
\sigma_1 & 0 & 0 \\
0 & \sigma_2 & 0 \\
0 & 0 & 0
\end{bmatrix}
V^T
$$

In practice, **RANSAC** wraps this computation to handle outlier matches
robustly.

---

### 3b. The Epipoles

#### Geometric Meaning

The **epipole** is the projection of one camera's optical centre onto the
other camera's image plane.

- $e$ in image 1: projection of camera 2's centre onto image 1
- $e'$ in image 2: projection of camera 1's centre onto image 2

All epipolar lines in a given image pass through its epipole. The epipole
is the **vanishing point** of the direction of the baseline between the
two cameras.

```
         3D point P
        /          \
       /            \
   Camera 1        Camera 2
   (centre C1)     (centre C2)
       \            /
        e           e'
   (C2 projected   (C1 projected
    into image 1)   into image 2)
```

> Intuition: if you are driving forward and filming, all motion trails
> converge toward a single point on the horizon. That point is the epipole.

---

#### Algebraic Relationship with F

The epipoles are the **null vectors** of $F$:

$$
F\, e = 0 \qquad F^T e' = 0
$$

- $e$ is the **right null vector** of $F$
- $e'$ is the **left null vector** of $F$

This is why $F$ must have **rank 2**: a full-rank $3 \times 3$ matrix has
no null vector, but the epipoles must always exist geometrically, so $F$
must be singular:

$$
\det(F) = 0
$$

---

### Exam-Ready Answer

> The fundamental matrix $F$ encodes the epipolar geometry between two
> views. For any correspondence $(p, p')$ across two images it satisfies
> $p'^T F p = 0$, meaning that given a point in image 1, its match in
> image 2 must lie on the epipolar line $l' = Fp$, reducing 2D search to
> 1D. $F$ is computed from at least 8 point correspondences using the
> eight-point algorithm: each pair gives one linear equation in the 9
> entries of $F$, the system is solved via SVD, and the rank-2 constraint
> is enforced by zeroing the smallest singular value. RANSAC is used in
> practice to handle outlier matches. $F$ conveys the relative projective
> geometry between the two cameras without requiring knowledge of their
> internal parameters.
>
> The epipoles are the projections of each camera's optical centre onto
> the other camera's image plane. All epipolar lines in an image fan out
> from its epipole, making it the vanishing point of the camera baseline
> direction. Algebraically, the epipoles are the null vectors of $F$:
> $Fe = 0$ and $F^T e' = 0$, which is why $F$ must have rank 2 and
> satisfy $\det(F) = 0$.