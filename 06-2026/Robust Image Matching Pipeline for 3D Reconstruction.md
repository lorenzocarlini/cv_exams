## Exercise 2. (8 Points)
#### You are given a set of images of the same building taken from different viewpoints, some images contain repeated structures such as windows, columns or similar textures. You need to build a robust image matching pipeline for 3D reconstruction.
#### Describe: a) Which local features and descriptors you would use and why. b) How you would match descriptors between image paris. c) How you would remove incorrect matches caused by repeated patterns.

---

### a) Local Features and Descriptors

#### Choice: SIFT

For a building with repeated structures and images taken from multiple
viewpoints, the descriptor must be robust to:

- **Scale changes:** the same window looks different sizes from near and far
- **Viewpoint changes:** columns look different from different angles
- **Illumination changes:** outdoor lighting varies across images

SIFT is the strongest classical choice because it satisfies all three
requirements. It detects keypoints at their characteristic scale in a
Gaussian pyramid, assigns a dominant orientation for rotation invariance,
and produces a 128-dimensional descriptor of local gradient histograms
that is normalised against illumination.

**Why not simpler descriptors?**

- Raw intensity patches fail under viewpoint and lighting changes
- Harris corners give locations but no descriptor
- SURF is faster but slightly less accurate, which matters when repeated
  structures already make matching ambiguous

---

#### Keypoint Selection Strategy

With repeated structures, many keypoints will look similar. It is important
to keep only high-quality, well-localised keypoints by applying strict
thresholds on:

- DoG contrast threshold (remove low-contrast responses)
- Edge response threshold (remove keypoints on window frames or column
  edges that are poorly localised along one direction)

---

### b) Matching Descriptors Between Image Pairs

#### Nearest Neighbour Matching

For each keypoint in image $A$, find the keypoint in image $B$ with the
most similar descriptor using Euclidean distance in the 128-dimensional
space:

$$
\text{match}(i) = \arg\min_j \| d_i^A - d_j^B \|_2
$$

#### Lowe's Ratio Test

Nearest neighbour alone produces many false matches, especially with
repeated structures where many descriptors look similar. The ratio test
rejects ambiguous matches:

$$
\frac{\|d_i^A - d_{\text{best}}^B\|}{\|d_i^A - d_{\text{second}}^B\|} < \tau
\qquad (\tau \approx 0.75)
$$

A match is only accepted if the best match is significantly closer than
the second best. If two candidates are similarly close, the match is
ambiguous and is discarded.

This is the most important step for repeated structures: a window keypoint
will have many similarly close matches (other windows), so the ratio will
be near 1 and the match will be correctly rejected.

---

### c) Removing Incorrect Matches Caused by Repeated Patterns

Even after the ratio test, many incorrect matches remain. Two geometric
verification steps are applied.

#### Step 1: RANSAC with Fundamental Matrix

Run RANSAC to estimate the fundamental matrix $F$ from the surviving
matches. At each iteration:

```
1. Sample 8 random matches
2. Compute F via the eight-point algorithm
3. Count inliers: matches where |p'^T F p| < epsilon
4. Keep the F with the largest inlier set
```

Incorrect matches caused by repeated patterns will not be geometrically
consistent with the true camera geometry, so they will fail the epipolar
constraint and be rejected as outliers. Only matches that satisfy
$p'^T F p \approx 0$ are kept.

#### Step 2: Homography Verification

Buildings contain many planar surfaces such as walls and facades. A
homography $H$ can be estimated for each dominant plane using RANSAC with
4 point correspondences. Matches consistent with a valid homography are
more likely to be correct, and this also helps separate matches from
different planes that would otherwise confuse the reconstruction.

---

#### Why Repeated Patterns Are Especially Hard

| Problem | Effect |
| ---------------------------------- | ----------------------------------------------- |
| Many similar descriptors | Ratio test rejects most, but some pass |
| Matches between wrong instances | Geometrically inconsistent with true $F$ |
| Multiple valid local geometries | RANSAC may converge to a wrong plane |

The combination of ratio test and RANSAC-based geometric verification
handles most of these cases.

---

### Exam-Ready Answer

> To build a robust matching pipeline for a building with repeated
> structures, SIFT is the descriptor of choice because it is invariant to
> scale, rotation, and illumination, and its 128-dimensional gradient
> histogram descriptor is distinctive enough to survive viewpoint changes.
> Strict keypoint filtering on contrast and edge response reduces the number
> of ambiguous candidates from repeated elements such as windows and columns.
>
> Matching is performed by nearest neighbour search in descriptor space,
> filtered by Lowe's ratio test: a match is accepted only if the closest
> descriptor is significantly closer than the second closest, with threshold
> $\tau \approx 0.75$. This is critical for repeated structures, where many
> descriptors look similar and the ratio will be close to 1, causing
> ambiguous matches to be discarded.
>
> Remaining incorrect matches are removed by RANSAC-based estimation of the
> fundamental matrix. Each iteration samples 8 matches, computes $F$ via
> the eight-point algorithm, and counts inliers using the epipolar
> constraint $|p'^T F p| < \epsilon$. Matches caused by repeated patterns
> are geometrically inconsistent with the true camera geometry and are
> rejected as outliers. The final set of verified inliers provides the
> reliable correspondences needed for accurate 3D reconstruction.