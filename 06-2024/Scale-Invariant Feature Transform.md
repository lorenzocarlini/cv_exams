---
tags:
  - computer-vision
  - feature-detection
  - feature-description
  - sift
  - surf
  - exam-prep
---

# Exercise 2. (8 Points)

#### a. Explain the Scale-Invariant Feature Transform (SIFT) algorithm. Outline the steps involved in detecting and describing features using SIFT. How does it achieve scale and rotation invariance?
#### b. Compare SIFT with the Speeded-Up Robust Features (SURF) algorithm. Discuss the differences in

## 2a. The SIFT Algorithm

### Core idea

SIFT detects distinctive points in an image and describes them in a way that
is stable across changes in scale, rotation, and illumination. The same
physical corner or blob should produce the same descriptor regardless of how
the image was taken.

---

### Step 1: Build a scale space

The image is blurred with Gaussians of increasing $\sigma$ across multiple
octaves. Each octave halves the image resolution. This simulates viewing the
scene from different distances.

$$
L(x, y, \sigma) = G(x, y, \sigma) * I(x, y)
$$

---

### Step 2: Difference of Gaussians (DoG)

Consecutive blurred images within each octave are subtracted:

$$
D(x, y, \sigma) = L(x, y, k\sigma) - L(x, y, \sigma)
$$

This approximates the Laplacian of Gaussian and highlights blob-like
structures at each scale.

---

### Step 3: Find keypoint candidates

Local extrema are found by comparing each point to its 26 neighbours:
8 in the same layer, 9 in the layer above, 9 in the layer below.

```
For each point in DoG:
    compare to 26 neighbours across 3 layers
    if local max or min -> candidate keypoint
```

---

### Step 4: Filter bad keypoints

Two types of candidates are discarded:

- **Low contrast points:** unstable under noise, removed by thresholding
  the DoG response
- **Edge points:** poorly localised along the edge direction, removed using
  the ratio of principal curvatures from the Hessian matrix

---

### Step 5: Assign orientation (rotation invariance)

For each keypoint, a histogram of gradient orientations is computed in the
surrounding neighbourhood. The dominant orientation becomes the reference
direction. The descriptor is then computed relative to this direction,
making it **rotation invariant**.

$$
\theta = \arg\max_\theta \ \text{histogram of} \ \nabla I \ \text{directions}
$$

---

### Step 6: Build the descriptor

A $16 \times 16$ patch around the keypoint is divided into a $4 \times 4$
grid of cells. In each cell an 8-bin gradient orientation histogram is
computed. All histograms are concatenated:

$$
4 \times 4 \times 8 = 128 \ \text{dimensional vector}
$$

The vector is $L_2$-normalised to reduce sensitivity to illumination changes.

---

### How invariances are achieved

| Invariance | Mechanism |
| ------------ | --------------------------------------------------------------- |
| Scale | Keypoints detected across the full scale pyramid |
| Rotation | Descriptor aligned to dominant gradient orientation |
| Illumination | Descriptor uses relative gradients and is $L_2$-normalised |

---

## 2b. SIFT vs SURF

### Key idea behind SURF

SURF approximates the computations in SIFT using simpler operations that run
much faster, mainly by exploiting **integral images**: a precomputed lookup
table where any rectangular pixel sum takes $O(1)$ time regardless of region
size.

---

### Scale space construction

| | SIFT | SURF |
| ------- | ------------------------------------ | --------------------------------------- |
| Method | Gaussian blur + DoG subtraction | Box filter approximation of Laplacian |
| Tool | Standard convolution | Integral images |
| Speed | Slower | Much faster |

---

### Keypoint detection

| | SIFT | SURF |
| --------- | ------------------------------ | ---------------------------------- |
| Detector | DoG extrema | Determinant of Hessian matrix |
| Finds | Blob-like structures | Blob-like structures |

---

### Descriptor

| | SIFT | SURF |
| --------- | --------------------------------------- | -------------------------------------- |
| Size | 128 dimensions | 64 dimensions |
| Built from | Gradient orientation histograms | Haar wavelet responses |
| Speed | Slower | Roughly 3x faster |

---

### Rotation invariance

| | SIFT | SURF |
| --------- | --------------------------------------- | -------------------------------------------- |
| Method | Gradient orientation histogram | Haar wavelet responses in sliding window |
| Accuracy | More precise | Slightly less precise, faster |

---

### Summary

| Property | SIFT | SURF |
| -------------------- | ---------- | ------------------- |
| Accuracy | Higher | Slightly lower |
| Speed | Slower | 3 to 7x faster |
| Descriptor size | 128 | 64 |
| Scale invariance | Yes | Yes |
| Rotation invariance | Yes | Yes (approximate) |
| Key trick | DoG + gradient histograms | Integral images + Haar wavelets |

---

## Exam-ready answer

> SIFT detects keypoints by building a Gaussian scale pyramid and computing
> the Difference of Gaussians across scales. Local extrema in the DoG volume
> are candidate keypoints; low-contrast and edge responses are then discarded.
> Scale invariance is achieved because the same structure produces an extremum
> at its characteristic scale regardless of image resolution. Rotation
> invariance is achieved by aligning the descriptor to the dominant gradient
> orientation in the keypoint neighbourhood. The descriptor is a
> 128-dimensional vector of gradient orientation histograms over a
> $4 \times 4$ grid of cells, normalised to reduce illumination sensitivity.
>
> SURF achieves similar invariances at significantly higher speed by
> approximating Gaussian derivatives with box filters evaluated using integral
> images, reducing expensive convolutions to $O(1)$ rectangular sums. Its
> descriptor uses 64-dimensional Haar wavelet responses instead of gradient
> histograms, making it roughly 3x faster to compute and match. The tradeoff
> is a slight reduction in accuracy and rotation precision. SIFT is preferred
> when accuracy is critical; SURF when speed is the priority.