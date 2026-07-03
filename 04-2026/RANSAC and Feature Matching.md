---
tags:
   - computer-vision
   - feature-matching
   - ransac
   - homography
   - exam-prep
---
# Exercise 2. (8 points)

#### Explain feature matching and how the RANSAC (Random Sample Consensus) algorithm can be used in conjunction with feature matching to improve the accuracy of homography estimation between two images.

## Feature Matching

After detecting keypoints (e.g. with SIFT), each one has a 128-dimensional descriptor vector. Matching means finding, for each keypoint in image 1, the keypoint in image 2 whose descriptor is most similar, usually by nearest-neighbour search with Euclidean distance.

The problem is that many matches are wrong. These are **outliers**. Good matches are **inliers**. A common filter is Lowe's ratio test: accept a match only if the best match is much closer than the second-best match. If the two closest descriptors are too similar, the match is ambiguous and discarded.

Even after this, many outliers can remain. This is where RANSAC comes in.

## What is a Homography?

A homography is a 3×3 matrix \(H\) that maps points from one image to corresponding points in another, assuming the scene is planar or the camera center stays fixed:

$$
p' = Hp
$$

To compute \(H\), at least 4 point correspondences are needed. The challenge is deciding which noisy matches to trust.

## RANSAC

### Core idea

Instead of using all matches, RANSAC repeatedly selects a small random subset, fits a model, and checks how many other points agree with it.

### Steps

1. Randomly sample the minimum number of matches needed.
   - Pick 4 random matches.
   - Compute the homography \(H\) from these 4 pairs.

2. Compute the reprojection error for all other matches.
   - For every other match (($p, p'$)), apply ($H$) to ($p$) and measure how far the result is from \(p'\):

$$
\text{error} = \|p' - Hp\|
$$

3. Count inliers.
   - Any match whose error is below a threshold ($epsilon$) is counted as an inlier.
   - This set is called the **consensus set**.

4. Repeat.
   - Go back to step 1 and repeat \(N\) times, typically hundreds of iterations.

5. Keep the best model.
   - The iteration with the largest consensus set wins.
   - Recompute \(H\) using all inliers from that best iteration for a more accurate final estimate.
---
### Example

- Iteration 1: sample 4 → compute \(H\) → 12 inliers
- Iteration 2: sample 4 → compute \(H\) → 47 inliers ← best
- Iteration 3: sample 4 → compute \(H\) → 8 inliers

Final \(H\): recomputed using all 47 inliers.

---
### Why it works

A homography computed from 4 correct matches will be consistent with the other correct matches. A homography computed from even one wrong match will usually fit poorly, so very few other matches will agree with it.

Across many iterations, the correct model accumulates a large consensus set while wrong models do not. RANSAC uses this difference to separate inliers from outliers without knowing in advance which matches are correct.

## Exam-ready answer

>Feature matching assigns each detected keypoint a descriptor, such as SIFT's 128-dimensional vector, and finds corresponding keypoints across two images by nearest-neighbour search in descriptor space. Since many matches are wrong, RANSAC is used to robustly estimate the homography despite outliers.

>RANSAC works iteratively: at each iteration, 4 matches are sampled at random and a homography ($H$) is computed from them. ($H$) is then evaluated by projecting all other matched points and counting how many fall within a small error threshold. These are the inliers. This process is repeated many times, and the homography with the largest inlier set is kept. The final homography is then recomputed using all inliers from the best iteration, producing a clean estimate that is not affected by outlier matches.

>The key insight is that a homography built from correct matches will be consistent with other correct matches, while one built from wrong matches will produce large reprojection errors. RANSAC exploits this to separate inliers from outliers.