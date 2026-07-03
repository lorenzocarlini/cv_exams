---
tags:
  - image-processing
  - computer-vision
  - fourier-transform
  - frequency-domain
  - dft
  - filtering
  - exam-prep
aliases:
  - DFT in Image Filtering
---

# Exercise 1. (8 points)

#### Discuss the role of DFT (Discrete Fourier Transform) in image filtering and what information does it provide about an image. Provide an example of how DFT can be used to perform high-pass and low-pass filtering in the frequency domain.

## What is the DFT?

The 2D Discrete Fourier Transform converts an image from the spatial domain
(pixels) into the frequency domain. Each coefficient $F(u,v)$ encodes how
much of a particular sinusoidal pattern at frequency $(u,v)$ is present in
the image.

$$
F(u,v) = \sum_{x=0}^{M-1}\sum_{y=0}^{N-1} f(x,y)\, e^{-j2\pi\left(\frac{ux}{M}+\frac{vy}{N}\right)}
$$

The inverse (IDFT) reconstructs the image from its frequency representation:

$$
f(x,y) = \frac{1}{MN}\sum_{u=0}^{M-1}\sum_{v=0}^{N-1} F(u,v)\, e^{j2\pi\left(\frac{ux}{M}+\frac{vy}{N}\right)}
$$

---

## What information does the DFT provide?

After applying `fftshift` to center the spectrum, the position in the
frequency map tells you what kind of content is present:

| Frequency Region | What it Represents |
| ---------------- | ---------------------------------------------------- |
| Center (low) | Smooth regions, gradual changes, large structures |
| Periphery (high) | Edges, fine details, noise, sharp transitions |

Each coefficient $F(u,v)$ is a complex number encoding:

- **Magnitude** $|F(u,v)|$: how strongly that frequency is present
- **Phase** $\angle F(u,v)$: where that frequency pattern is located spatially

---

## Why filter in the frequency domain?

Because of the **Convolution Theorem**:

$$
f(x,y) * h(x,y) \longleftrightarrow F(u,v) \cdot H(u,v)
$$

Spatial convolution becomes pointwise multiplication in frequency space,
especially efficient with the FFT running in $O(N^2 \log N)$.

The filter $H(u,v)$ is called the **transfer function**.

---

## Low-pass filter (smoothing / blurring)

Keep low frequencies, suppress high ones. Removes noise and blurs edges.

Ideal LPF with cutoff radius $D_0$:

$$
H_{LP}(u,v) =
\begin{cases}
1 & \text{if } D(u,v) \leq D_0 \\
0 & \text{otherwise}
\end{cases}
$$

where $D(u,v) = \sqrt{u^2 + v^2}$ after shift.

In practice a **Gaussian LPF** is preferred to avoid ringing (Gibbs phenomenon):

$$
H_{LP}(u,v) = e^{-D(u,v)^2 / 2\sigma^2}
$$

**Result:** smooth image, edges softened. Equivalent to a spatial Gaussian blur.

---

## High-pass filter (edge detection / sharpening)

Suppress low frequencies, keep high ones. Enhances edges and fine detail.

Simply the complement of the LPF:

$$
H_{HP}(u,v) = 1 - H_{LP}(u,v)
$$

**Result:** edges and fine detail highlighted, flat regions go dark.
Equivalent to a spatial Laplacian or unsharp mask.

---

## Full Pipeline

```
Image f(x,y)
    |
    v
  2D FFT  -->  F(u,v)
                 |
                 v
         Multiply by H(u,v)    <- LPF or HPF mask
                 |
                 v
              G(u,v)
                 |
                 v
  2D IFFT -->  g(x,y)          <- filtered image
```

---

## Exam-ready answer

> The DFT transforms a spatial-domain image into its frequency-domain
> representation, where each coefficient $F(u,v)$ encodes the contribution
> of a sinusoidal component at frequency $(u,v)$. Low frequencies near the
> center of the shifted spectrum correspond to smooth regions and large
> structures; high frequencies at the periphery correspond to edges, fine
> details, and noise.
>
> Filtering in the frequency domain exploits the Convolution Theorem:
> spatial convolution with a kernel is equivalent to pointwise multiplication
> in the frequency domain, making it computationally efficient via the FFT.
>
> Low-pass filtering is achieved by multiplying $F(u,v)$ by a transfer
> function $H_{LP}$ that preserves frequencies within a radius $D_0$ of the
> origin and suppresses the rest, producing a blurred output. A Gaussian
> transfer function is preferred over a hard cutoff to avoid ringing.
> High-pass filtering uses the complementary mask $H_{HP} = 1 - H_{LP}$,
> retaining only high-frequency components to enhance edges and fine detail.
>
> In both cases the pipeline is: compute FFT, multiply pointwise by $H(u,v)$,
> apply IFFT to recover the filtered image.