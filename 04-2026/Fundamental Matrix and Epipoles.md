---
tags:
  - computer-vision
  - epipolar-geometry
  - fundamental-matrix
  - epipoles
  - exam-prep
---
# Exercise 3. (8 points)

#### a. Explain the concept of the fundamental matrix. How is it computed, and what information does it convey about the relationship between two images?
#### b. What is the geometric meaning of the epipoles in the two images? How are they related to the fundamental matrix algebraically?

## 3a. The Fundamental Matrix
### Core idea

When two cameras look at the same scene from different positions, a point visible in both images must satisfy a geometric constraint. The fundamental matrix ($F$) is a 3×3 matrix that encodes this constraint from geometry alone, without needing the 3D point location.

Given a point ($p$) in image 1 and its correspondence ($p'$) in image 2:

$$
p'^T F p = 0
$$

This does not directly tell where the match is, but it tells where it must lie: on a line in image 2 called the **epipolar line**:

$$
l' = Fp
$$

This reduces correspondence search from a 2D problem to a 1D search along a line, which is very useful for stereo matching.

---
### What information does $F$ convey?

- The relative geometry between the two cameras, such as rotation and translation, encoded implicitly.
- For every point in image 1, the epipolar line in image 2 where its match must lie.
- It works without knowing the internal camera parameters, so it is a purely projective relationship.

### How is $F$ computed?

You need at least 8 point correspondences between the two images. Each pair (($p_i$, $p'_i$)) gives one linear equation in the 9 entries of ($F$). Stacking 8 or more gives a system solved via SVD.

The rank-2 constraint, meaning ($F$) must be singular, is then enforced by zeroing the smallest singular value of the result.

In practice, RANSAC is applied around this step to handle outlier matches robustly.

## 3b. The Epipoles

### Geometric meaning

The epipole is the projection of one camera's optical centre onto the other camera's image plane.

- ($e$) in image 1: projection of camera 2's centre onto image 1.
- ($e'$) in image 2: projection of camera 1's centre onto image 2.

All epipolar lines in a given image pass through its epipole. The epipole is the vanishing point of the baseline direction between the two cameras.

Intuition: if you are driving forward and filming, all motion trails converge toward a single point on the horizon. That point is the epipole.

---

### Algebraic relationship with ($F$)

The epipoles are the null vectors of ($F$):

$$
Fe = 0 \qquad F^T e' = 0
$$

- ($e$) is the right null vector of ($F$).
- ($e'$) is the left null vector of ($F$).

This is why ($F$) must have rank 2. A full-rank 3×3 matrix has no null vector, but the epipoles must always exist geometrically, so ($F$) must be singular, with ($\det F = 0$).

---
## Exam-ready answer

> The fundamental matrix $F$ encodes the epipolar geometry between two views. For any correspondence $(p, p')$ across two images it satisfies $p'^T F p = 0$, meaning that given a point in image 1, its match in image 2 must lie on the epipolar line $l' = Fp$, reducing 2D search to 1D. $F$ is computed from at least 8 point correspondences using the eight-point algorithm, solved via SVD, with rank 2 enforced by zeroing the smallest singular value. RANSAC is used in practice to handle outliers.

> The epipoles are the projections of each camera's optical centre onto the other camera's image plane. All epipolar lines in an image pass through its epipole. Algebraically, the epipoles are the null vectors of $F$: $Fe = 0$ and $F^T e' = 0$, which is why $F$ must have rank 2 and be singular.