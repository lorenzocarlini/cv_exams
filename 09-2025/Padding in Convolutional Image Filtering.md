---
tags:
  - computer-vision
  - image-processing
  - convolution
  - padding
  - edge-detection
  - exam-prep
---
# Exercise 2. (8 points)

#### Explain the concept of padding in the context of convolutional image filtering. How does the choice of padding method influence edge detection results, especially near the image boundaries?

---

## What is Padding?

When you convolve a filter (e.g., 3x3) over an image, the filter needs to be centered on each pixel. For pixels near the border, part of the filter hangs outside the image. Padding is how you handle those missing values. Without padding, you have two options: skip border pixels entirely (shrinking the output) or invent values for the missing region. The second approach is padding.

---

## Why it matters

For a filter of size $k \times k$, the border region where the problem occurs has width $p = \lfloor k/2 \rfloor$ pixels. For a 3x3 filter that is 1 pixel; for a 5x5 filter it is 2 pixels. Without padding, each convolution layer shrinks the image, which is problematic for deep pipelines. More importantly for edge detection, the choice of padding directly affects what values get multiplied with the filter near the image boundary, which changes the detected edges there.

---

## Common Padding Methods

Consider this original image $Im$:

$$Im = \begin{bmatrix} 3 & 1 & 2 & 0 \\ 0 & 2 & 1 & 5 \\ 1 & 0 & 1 & 3 \\ 1 & 2 & 0 & 5 \end{bmatrix}$$

### Zero padding (most common)

Fill the border region with zeros before applying the filter:

$$Im_{\text{zero}} = \begin{bmatrix} 0 & 0 & 0 & 0 & 0 & 0 \\ 0 & 3 & 1 & 2 & 0 & 0 \\ 0 & 0 & 2 & 1 & 5 & 0 \\ 0 & 1 & 0 & 1 & 3 & 0 \\ 0 & 1 & 2 & 0 & 5 & 0 \\ 0 & 0 & 0 & 0 & 0 & 0 \end{bmatrix}$$

**Effect on edge detection:** The artificial zeros create a strong intensity contrast at the image border. Sobel or Canny will detect a false edge all around the image boundary, since the gradient between real pixel values and the surrounding zeros is large. 

### Replicate padding (edge padding)

The border pixels of the image are repeated outward:

$$Im_{\text{rep}} = \begin{bmatrix} 3 & 3 & 1 & 2 & 0 & 0 \\ 3 & 3 & 1 & 2 & 0 & 0 \\ 0 & 0 & 2 & 1 & 5 & 5 \\ 1 & 1 & 0 & 1 & 3 & 3 \\ 1 & 1 & 2 & 0 & 5 & 5 \\ 1 & 1 & 2 & 0 & 5 & 5 \end{bmatrix}$$

**Effect on edge detection:** No artificial gradient is introduced at the border since the padded values match the edge pixels. Boundary edges are suppressed rather than amplified. This is more honest but means real edges at the image border may be missed. 

### Reflect padding

Values are mirrored around the border pixel:

$$Im_{\text{ref}} = \begin{bmatrix} 2 & 1 & 3 & 1 & 2 & 0 \\ 2 & 1 & 3 & 1 & 2 & 0 \\ 1 & 0 & 0 & 2 & 1 & 5 \\ 2 & 1 & 1 & 0 & 1 & 3 \\ 2 & 1 & 1 & 2 & 0 & 5 \\ 2 & 1 & 1 & 2 & 0 & 5 \end{bmatrix}$$

**Effect on edge detection:** Even smoother transition than replicate padding. The gradient at the boundary is minimised, so false edges are avoided and real boundary edges are also suppressed. 

### Wrap padding (circular)

Values from the opposite edge are used to fill the border.

**Effect on edge detection:** Only makes physical sense for periodic signals. On natural images it introduces severe false edges at the boundary where visually unrelated regions are placed next to each other.

---

## Direct impact on edge detection

| Padding | False boundary edges | Real boundary edges preserved | Use case |
| :--- | :--- | :--- | :--- |
| **Zero** | Yes, strong | Partially | General default |
| **Replicate** | No | Weakly | When boundaries matter |
| **Reflect** | No | Weakly | Smooth boundary behaviour |
| **Wrap** | Severe | No | Periodic signals only |

The core tradeoff is:
* **Zero padding** preserves output size and is simple, but pollutes the boundary with artificial gradients.
* **Replicate/Reflect** avoids false edges but may suppress real ones near the border.
* For most edge detection tasks, **replicate or reflect padding** produces cleaner results at the boundaries.

---

## Exam-ready answer

> Padding fills the region outside the image border so that a convolution filter can be applied to every pixel including those at the edges, preserving the output size. Without padding, each convolution would shrink the image by $\lfloor k/2 \rfloor$ pixels on each side.
> 
> The choice of padding directly affects edge detection near image boundaries. Zero padding introduces artificial zeros around the image, creating a strong intensity contrast that Sobel or Canny will interpret as edges, producing false detections along the entire image border. Replicate padding repeats the border pixel values outward, avoiding artificial gradients and therefore suppressing false boundary edges, at the cost of potentially missing real edges near the border. Reflect padding mirrors values around the border and behaves similarly to replicate but with an even smoother transition.
> 
> In summary, zero padding is the most common default and preserves output dimensions, but for applications where boundary accuracy matters, replicate or reflect padding produces more reliable edge detection results near the image borders.