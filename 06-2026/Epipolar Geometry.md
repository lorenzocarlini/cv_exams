## Exercise 3 (8 Points)

#### **Question:** Explain the concept of epipolar geometry between two images. Discuss: a) the meaning of epipolar lines and epipoles, b) how the fundamental matrix relates corresponding points in two images, c) why the fundamental matrix has rank 2, d) how epipolar geometry can reduce the search space in stereo matching.

---

### a) Epipolar Lines and Epipoles

Epipolar geometry describes the geometric relationship between two views of the same 3D scene taken from different camera positions. The key idea is that a point seen in one image cannot match anywhere in the second image: its correspondence must lie on a specific line called the **epipolar line**. This reduces the matching problem from a 2D search over the whole image to a 1D search along a line.

For a 3D point \(X\), the camera centers and the point define an **epipolar plane**. When this plane intersects each image plane, it produces two epipolar lines, one in each image. The point where the baseline connecting the two camera centers meets an image plane is the **epipole**; it is the common intersection point of all epipolar lines in that image.

---

### b) The Fundamental Matrix

The **fundamental matrix** \(F\) encodes the epipolar geometry between two uncalibrated images. If \(x\) is a point in the first image and \(x'\) is the corresponding point in the second image, then they satisfy the epipolar constraint:

$$
x'^T F x = 0
$$

This equation means that once a point \(x\) is known in one image, the matching point \(x'\) must lie on the epipolar line:

$$
l' = F x
$$

Similarly, the corresponding epipolar line in the first image is:

$$
l = F^T x'
$$

So the fundamental matrix does not give the exact match directly, but it gives the line on which the match must lie.

---

### c) Why \(F\) Has Rank 2

The fundamental matrix has rank 2 because it is singular: it has a one-dimensional null space on both sides corresponding to the epipoles. In particular, the epipole \(e'\) in the second image satisfies:

$$
F e = 0 \qquad \text{and} \qquad F^T e' = 0
$$

This means \(F\) is singular, so its determinant is zero:

$$
\det(F) = 0
$$

A \($3 \times 3$\) matrix with zero determinant has rank at most 2. For a valid stereo setup it is typically exactly rank 2, not lower. Geometrically, the rank deficiency reflects the fact that all epipolar lines pass through a single epipole, so the constraints are not fully independent.

---

### d) How It Reduces Stereo Search

Without epipolar geometry, matching a point in one image to the other would require searching over the whole 2D image. With epipolar geometry, the correspondence must lie on the epipolar line, so the search becomes one-dimensional. In rectified stereo, epipolar lines are horizontal, so the search is even simpler: points are matched along the same image row.

This greatly reduces computation and also makes matching more reliable because impossible matches outside the epipolar line can be discarded immediately. In practice, this is one of the main reasons stereo vision is computationally feasible.

---

### Exam-Ready Answer

> Epipolar geometry describes the relationship between two images of the same 3D scene. A 3D point and the two camera centers define an epipolar plane, and its intersection with each image gives an epipolar line. The camera center projection in each image is called the epipole, and all epipolar lines pass through it.
>
> The fundamental matrix \(F\) encodes this geometry through the epipolar constraint \($x'^T F x = 0$\), where corresponding points \(x\) and \(x'\) must satisfy it. This means that if one image point is known, its match in the other image must lie on a specific epipolar line.
>
> The matrix \(F\) has rank 2 because it is singular: the epipoles lie in its null spaces, so \($det(F)=0$\). This rank deficiency reflects the geometric structure of the two-view setup.
>
> Epipolar geometry reduces stereo matching from a 2D search to a 1D search along an epipolar line, and after rectification the search becomes horizontal along the same scanline.