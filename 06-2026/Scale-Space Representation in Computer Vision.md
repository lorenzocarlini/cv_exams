## Exercise 1. (8 points)

#### **Question:** Explain the role of scale-space representation in computer vision. 
#### Discuss: a) why analyzing an image at multiple scales can be useful, b) how Gaussian filtering is used to construct a scale-space representation, c) the difference between smoothing at a single scale and building a Gaussian pyramid, d) one application where multi-scale analysis is important.

---

### a) Why Analyzing at Multiple Scales is Useful

Real-world images contain structures at many different sizes. A cat in the
foreground occupies hundreds of pixels; the same cat far away occupies
tens. A single fixed-scale analysis will either miss large structures or
miss fine details, depending on what scale it is tuned to.

Multi-scale analysis addresses this by asking: **what is present at this
location across all scales?** Features that are stable and distinctive
across scales are more reliable than those that only appear at one
specific resolution.

Concretely:

| Scale | What is visible |
| ----------- | --------------------------------------- |
| Fine (small $\sigma$) | Texture, noise, fine edges |
| Medium | Object parts, local structures |
| Coarse (large $\sigma$) | Overall shape, large regions |

The key insight is that the **characteristic scale** of a structure is
the scale at which it produces the strongest response. Finding this scale
automatically is the basis of scale-invariant detection.

---

### b) Gaussian Filtering to Construct Scale-Space

The scale-space of an image $I$ is defined by convolving it with Gaussians
of increasing standard deviation $\sigma$:

$$
L(x, y, \sigma) = G(x, y, \sigma) * I(x, y)
$$

where the Gaussian kernel is:

$$
G(x, y, \sigma) = \frac{1}{2\pi\sigma^2} e^{-\frac{x^2 + y^2}{2\sigma^2}}
$$

Increasing $\sigma$ progressively suppresses finer detail, leaving only
coarser structures. The result is a continuous family of increasingly
smoothed images parameterised by $\sigma$.

**Why Gaussian specifically?**

The Gaussian is the only kernel that satisfies the scale-space axioms:
it does not introduce new structures as scale increases (no spurious
edges or blobs appear). Any other smoothing kernel can create false
features at coarser scales.

---

### c) Single-Scale Smoothing vs Gaussian Pyramid

#### Single-Scale Smoothing

Applying one Gaussian with a fixed $\sigma$ to the original image gives
one smoothed version. This is useful for noise removal but provides no
information about how structures behave across scales. The choice of
$\sigma$ is arbitrary and may miss features at other scales.

---

#### Gaussian Pyramid

A Gaussian pyramid builds a **stack of progressively smoothed and
subsampled images**:

```
Original (full resolution)
    |  blur with G_sigma, then downsample by 2
    v
Octave 1 (half resolution)
    |  blur with G_sigma, then downsample by 2
    v
Octave 2 (quarter resolution)
    |
    v
   ...
```

Each level represents the image at a coarser scale. Subsampling after
blurring is valid because the Gaussian has already removed the
high-frequency content that would cause aliasing.

| | Single-Scale Smoothing | Gaussian Pyramid |
| -------------- | ----------------------- | ---------------------------------- |
| Scales covered | One fixed $\sigma$ | Many, across octaves |
| Resolution | Unchanged | Decreases at each level |
| Use case | Noise removal | Multi-scale detection and matching |
| Information | One smoothed image | Full scale-space representation |

The pyramid is more efficient than storing all smoothed versions at full
resolution: each octave halves the image size, so the total storage is
bounded by a geometric series converging to twice the original image size.

---

### d) Application: SIFT Keypoint Detection

SIFT is the canonical example of multi-scale analysis being essential.

The goal is to detect the same physical point in two images taken at
different distances (different scales). A detector operating at a single
scale will find the feature at one distance but miss it at another.

SIFT builds a Gaussian pyramid and computes the **Difference of Gaussians
(DoG)** between adjacent levels:

$$
D(x, y, \sigma) = L(x, y, k\sigma) - L(x, y, \sigma)
$$

Keypoints are located at extrema of the DoG volume across both space and
scale. Each keypoint is assigned its **characteristic scale** (the scale
at which it is most prominent), which is then used to make the descriptor
scale-invariant.

Without multi-scale analysis this is impossible: there is no way to detect
the characteristic scale of a feature from a single smoothed image.

---

### Exam-Ready Answer

> Scale-space representation allows an image to be analysed at multiple
> levels of detail simultaneously. Since real scenes contain structures at
> many sizes, a single fixed scale will always miss some of them. The
> scale-space is constructed by convolving the image with Gaussians of
> increasing $\sigma$, producing $L(x,y,\sigma) = G_\sigma * I$. The
> Gaussian is the only valid smoothing kernel because it does not introduce
> new structures as scale increases.
>
> Single-scale smoothing applies one fixed $\sigma$ and produces one
> smoothed image, discarding all scale information. A Gaussian pyramid
> instead builds a stack of smoothed and progressively subsampled images
> across multiple octaves, efficiently representing the full scale-space.
>
> The most important application is SIFT keypoint detection. SIFT computes
> the Difference of Gaussians across pyramid levels and finds extrema in
> both space and scale, assigning each keypoint its characteristic scale.
> This makes keypoints detectable regardless of the distance at which the
> image was captured, enabling robust matching across scale changes.