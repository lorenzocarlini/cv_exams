---
tags:
  - computer-vision
  - video-segmentation
  - clustering
  - embeddings
  - exam-prep
---
# Exercise 5. (8 points)

#### Reasoning question: Consider two alternative approaches for segmenting a video into visually similar segments:

• Approach A: Use pixel intensity histograms as features and Euclidean
distance as a similarity measure, followed by k-means clustering.
• Approach B: Use deep CNN-based frame embeddings as features and cosine similarity as a distance measure, followed by a clustering method.

Compare the two approaches in terms of robustness, computational efficiency and scalability. Discuss in which scenarios Approach A might outperform Approach B, and vice versa. How the choice of features could affect the quality of segmentation?

## Quick Overview

|              | Approach A                 | Approach B                           |
| ------------ | -------------------------- | ------------------------------------ |
| Features     | Pixel intensity histograms | CNN frame embeddings                 |
| Similarity   | Euclidean distance         | Cosine similarity                    |
| Clustering   | K-means                    | Flexible (e.g. DBSCAN, hierarchical) |
| Compute cost | Low                        | High                                 |
| Robustness   | Low                        | High                                 |

---

## Approach A: Histograms + Euclidean + K-means

Represents each frame as a distribution of pixel intensities. Two frames are similar if their histograms are close in Euclidean distance.

**Strengths:**
- Very fast to compute, no GPU needed
- Simple to implement and interpret
- Scales easily to long videos

**Weaknesses:**
- Histograms discard all spatial information: a frame with the same colors in a different arrangement looks identical
- Euclidean distance on histograms is sensitive to small global shifts (e.g. a slight brightness change inflates the distance)
- K-means requires knowing $k$ in advance and assumes spherical, equally sized clusters, which rarely matches real video structure
- Fails completely at semantic similarity: two frames of "a person talking" with different lighting will not cluster together

---

## Approach B: CNN embeddings + cosine + flexible clustering

Passes each frame through a pretrained CNN (e.g. ResNet) to get a high-dimensional feature vector that captures semantic content, not just raw color.

**Strengths:**
- Captures semantic and structural content: lighting changes, small shifts and texture variations are absorbed by the network
- Cosine similarity measures the angle between vectors, making it invariant to the overall magnitude of the embedding, which improves robustness
- Works well with flexible clustering methods (e.g. DBSCAN) that do not require $k$ in advance and can find arbitrarily shaped clusters

**Weaknesses:**
- Computationally expensive: requires a forward pass through a CNN per frame
- Requires a pretrained model, which may not generalise well to highly specialised domains (e.g. medical imaging, satellite footage)
- Embeddings are not interpretable

---

## Robustness

Approach B is significantly more robust. CNN embeddings are trained on millions of images and learn invariances to illumination, scale, and viewpoint. Approach A breaks under any of these changes since a simple brightness shift alters every histogram bin.

---

## Computational Efficiency

Approach A is much faster. Histogram computation is $O(WH)$ per frame and k-means converges quickly. Approach B requires a CNN forward pass per frame, which is orders of magnitude slower without a GPU, and the resulting high-dimensional embeddings also make clustering more expensive.

---

## Scalability

For very long videos (thousands of frames), Approach A scales better due to low per-frame cost and small feature size. Approach B can be made scalable with batched GPU inference, but still has higher memory and compute requirements. High-dimensional embeddings also make clustering harder as dimensionality increases (curse of dimensionality), often requiring dimensionality reduction (e.g. PCA) as a preprocessing step.

---

## When Approach A outperforms Approach B

- **Controlled, uniform conditions:** if lighting and camera are fixed (e.g. a static surveillance camera), histograms are sufficient and fast
- **Low-resource environments:** no GPU available, real-time processing required
- **Simple scene changes:** hard cuts between very different scenes (day/night, indoor/outdoor) are easily caught by histogram shifts
- **Domain mismatch:** if no pretrained CNN exists for the target domain, embeddings may be meaningless

---

## When Approach B outperforms Approach A

- **Semantic segmentation:** grouping frames by content (scenes with the same character, same location) rather than just color
- **Variable lighting or camera motion:** CNN embeddings are robust to these; histograms are not
- **Subtle transitions:** a gradual scene change or a facial expression shift is invisible to histograms but captured in embedding space
- **General purpose use:** Approach B generalises across very different video types without retuning

---

## How feature choice affects segmentation quality

The choice of features directly determines what "similar" means to the algorithm:

- **Histograms** define similarity as "same color distribution," which is a proxy for visual similarity only under ideal conditions
- **CNN embeddings** define similarity as "same semantic content," which aligns much better with human perception of scene similarity

A poor feature space means that the clustering algorithm, no matter how sophisticated, operates on noisy or misleading inputs. Good features make even simple clustering methods effective; bad features make even the best clustering method fail. This is sometimes summarised as: **the feature representation is more important than the clustering algorithm.**

---

## Exam-ready answer

> Approach A uses pixel histograms and Euclidean distance as a fast, interpretable baseline. It is computationally cheap and scales well, but is fragile to lighting changes, camera motion, and any variation that alters the color distribution without changing the semantic content. K-means further limits it by requiring $k$ in advance and assuming spherical clusters.

> Approach B uses CNN embeddings, which capture high-level semantic content and are robust to low-level variations. Cosine similarity is more appropriate for high-dimensional embedding spaces than Euclidean distance, as it is invariant to vector magnitude. The main drawbacks are computational cost and dependence on a pretrained model.

> Approach A is preferable in real-time, low-resource, or controlled settings where scene changes are hard and obvious. Approach B is preferable whenever semantic understanding, robustness to lighting, or detection of subtle transitions is required.

> The choice of features is the most critical decision in the pipeline: features define what "similar" means, and no clustering algorithm can recover meaningful structure from a poorly chosen feature space.