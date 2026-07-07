## Exercise 4. (8 Points)

#### **Question:** A surveillance camera monitors a fixed outdoor area with strong illumination changes during the day. Design a traditional computer vision method to detect moving objects while reducing false detections caused by lighting variations. Describe: a) preprocessing steps, b) background modeling and foreground detection, c) how to distinguish real motion from illumination changes or shadows.

---

### a) Preprocessing Steps

Preprocessing reduces the impact of illumination changes before any
background modeling is applied.

#### Convert to Grayscale

Colour information is useful later for shadow removal, but working in
grayscale for motion detection reduces computational cost and avoids
colour-specific noise.

#### Gaussian Smoothing

Apply a Gaussian blur to each frame to suppress sensor noise and small
pixel-level fluctuations that would otherwise be mistaken for motion:

$$
I'(x,y) = G_\sigma * I(x,y)
$$

#### Histogram Normalisation

Apply adaptive histogram equalisation (CLAHE) to normalise the brightness
of each frame. This compresses the effect of global illumination shifts
such as a cloud passing in front of the sun.

#### Convert to HSV Colour Space

For later shadow detection, convert the frame to HSV. The **V (value)**
channel carries illumination; the **H (hue)** channel is largely
illumination-invariant. Separating them now makes shadow discrimination
easier in step c.

---

### b) Background Modeling and Foreground Detection

#### Why a Static Background Model Fails

A single reference background frame fails immediately under outdoor
conditions: gradual illumination drift, wind moving trees, and weather
changes all alter the background continuously. A static model would flag
every pixel as foreground by midday.

---

#### Gaussian Mixture Model (GMM)

The standard approach for outdoor scenes. Each pixel is modelled as a
mixture of $K$ Gaussians (typically $K = 3$ to $5$), each representing
one possible background state such as clear sky, overcast, or shadow:

$$
P(x_t) = \sum_{k=1}^{K} \pi_k\, \mathcal{N}(x_t \mid \mu_k, \sigma_k^2)
$$

At each frame, each pixel value $x_t$ is tested against its mixture. If
it fits one of the Gaussians within a threshold it is classified as
background; otherwise it is foreground.

The model updates continuously with a learning rate $\alpha$:

$$
\mu_k \leftarrow (1 - \alpha)\,\mu_k + \alpha\, x_t
$$

$$
\sigma_k^2 \leftarrow (1-\alpha)\,\sigma_k^2 + \alpha\,(x_t - \mu_k)^2
$$

This allows the background to adapt slowly to lighting drift while still
detecting fast-moving objects.

---

#### Foreground Mask Cleanup

Pixels classified as foreground form a binary mask. Raw masks are noisy
so morphological operations are applied:

```
1. Erode mask        -> removes isolated noise pixels
2. Dilate mask       -> fills small holes inside detected objects
3. Connected components -> group pixels into candidate blobs
4. Filter by area    -> discard blobs too small to be real objects
```

---

### c) Distinguishing Real Motion from Illumination Changes and Shadows

Even after GMM adaptation, two main sources of false positives remain:
global illumination changes and shadows cast by moving objects.

---

#### Global Illumination Changes

A sudden brightness change affects many pixels simultaneously across a
large region. Real moving objects affect a localised, compact region.

If the fraction of pixels flagged as foreground exceeds a threshold (e.g.
more than 30% of the image), the change is likely a global illumination
event rather than a real object. In this case the frame update is skipped
or the GMM adaptation rate is increased temporarily.

---

#### Shadow Detection

Shadows cause large dark regions that move with the object. If not
removed, they inflate the detected shape and produce false detections.

**HSV-based shadow discrimination (Cucchiara et al.):**

A foreground pixel is reclassified as shadow if it satisfies both
conditions:

**Condition 1: Value (brightness) is lower than background but not
extremely so**

$$
\alpha \leq \frac{V_t(x,y)}{V_b(x,y)} \leq \beta
\qquad \alpha = 0.4,\quad \beta = 0.9
$$

**Condition 2: Hue is approximately preserved**

$$
|H_t(x,y) - H_b(x,y)| < \tau_H
$$

A shadow does not change the colour of the surface, only its brightness.
If the hue matches the background but the value is lower, the pixel is a
shadow and is removed from the foreground mask. Real moving objects change
both hue and value and therefore survive this test.

---

#### Summary of False Positive Suppression

| Source | Suppression Method |
| ---------------------------- | ---------------------------------------- |
| Sensor noise | Gaussian pre-smoothing |
| Slow illumination drift | GMM adaptive background model |
| Sudden global change | Foreground fraction threshold |
| Shadows | HSV hue-value shadow discrimination |
| Small noise blobs | Morphological filtering + area threshold |

---

### Exam-Ready Answer

> To detect moving objects under outdoor illumination changes, the pipeline
> starts with preprocessing: Gaussian smoothing suppresses noise, adaptive
> histogram equalisation normalises brightness, and conversion to HSV
> separates illumination from colour information.
>
> Background modeling uses a Gaussian Mixture Model per pixel, which adapts
> continuously to slow illumination drift by updating the mean and variance
> of each Gaussian component with a small learning rate $\alpha$. Pixels
> that do not fit any background Gaussian are classified as foreground. The
> raw binary mask is cleaned with erosion, dilation, and area filtering to
> remove noise blobs.
>
> Two main sources of false detections remain. Global illumination events
> affect a large fraction of the image simultaneously and are detected by
> thresholding the total foreground area. Shadows are suppressed using HSV
> analysis: a foreground pixel is reclassified as shadow if its brightness
> is reduced relative to the background but its hue is preserved, since
> shadows darken a surface without changing its colour. Real moving objects
> change both brightness and hue and are therefore retained.