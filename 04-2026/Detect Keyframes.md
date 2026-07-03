---
tags:
    - computer-vision
    - video-analysis
    - keyframes
    - feature-extraction
    - exam-prep
---
# Exercise 4. (8 points)

#### Propose a method to detect keyframes in a video sequence using traditional (non-deep learning) techniques. Explain the features you would extract, how you would measure frame importance or uniqueness, and the criteria for selecting keyframes. Justify each step in your approach. A keyframe in this case could be a frame that is visually different from surrounding frames (e.g. scene changes, new objects) or a frame where meaningful action begins (e.g. explosion, expression change, speaker switch).

## Core Idea

A keyframe is a frame that is **visually distinct** from its neighbours. The strategy is to extract features from each frame, measure how much each frame differs from the previous one, and select frames where that difference is large or locally maximal.

---

## Step 1: Feature Extraction

Three complementary features capture different types of visual change:

#### Color Histogram (HSV space)
Represent each frame as a histogram of hue, saturation and value. Captures overall color distribution without caring about pixel positions.

$$H_t = \text{histogram}(I_t)$$

**Why:** very fast to compute, and highly sensitive to scene cuts, lighting changes, and background switches.

#### Edge Density (Sobel/Canny)
Apply Sobel to each frame and compute the mean gradient magnitude across the image.

$$E_t = \frac{1}{WH}\sum_{x,y} |G(x,y)|$$

**Why:** captures structural changes. A new object entering the scene or a camera cut will change the edge layout significantly.

#### Optical Flow Magnitude
Compute dense optical flow between consecutive frames (e.g. Lucas-Kanade) and take the mean magnitude of flow vectors.

$$F_t = \frac{1}{WH}\sum_{x,y} \sqrt{u(x,y)^2 + v(x,y)^2}$$

**Why:** captures motion intensity. A sudden action like an explosion or a gesture change produces a large spike in flow magnitude even when colors stay similar.

---

## Step 2: Measuring Frame Importance

Combine the three features into a single **frame difference score** $D_t$ comparing frame $t$ to frame $t-1$:

$$D_t = \alpha \cdot d_{\text{hist}}(H_t, H_{t-1}) + \beta \cdot |E_t - E_{t-1}| + \gamma \cdot F_t$$

where $d_{\text{hist}}$ is a histogram distance (e.g. chi-squared or L1), and $\alpha, \beta, \gamma$ are weights tuned to the application.

**Why a weighted combination:** different events are better captured by different features. A speaker switch may have low motion but a large color/edge change. An explosion will spike all three. Combining them gives a more robust importance signal than any single feature alone.

---

## Step 3: Keyframe Selection

#### Criterion 1: Global Threshold
Mark frame $t$ as a keyframe if:

$$D_t > \tau$$

where $\tau$ is a fixed or adaptive threshold (e.g. mean + 2 standard deviations of $D$ over the whole video). This catches hard cuts and sudden changes.

#### Criterion 2: Local Maxima (Non-Maximum Suppression in Time)
Among frames above the threshold, keep only those that are a **local maximum** of $D_t$ within a temporal window of size $w$:

$$D_t = \max(D_{t-w}, \dots, D_t, \dots, D_{t+w})$$

**Why:** prevents selecting multiple redundant frames during a single gradual transition. Only the peak of the change is kept.

#### Criterion 3: Minimum Temporal Distance
Enforce a minimum gap of $k$ frames between any two selected keyframes.

**Why:** even after local maxima suppression, two events can be close together. This avoids over-sampling short busy segments.

---

## Full Pipeline Summary

```text
For each frame t:
    1. Compute H_t, E_t, F_t
    2. Compute D_t vs frame t-1
    
After all frames:
    3. Threshold: keep frames where D_t > tau
    4. Local max suppression in time window w
    5. Enforce minimum gap of k frames
    6. Remaining frames are keyframes
```

---

## Exam-ready answer

To detect keyframes without deep learning, I propose a multi-feature frame difference pipeline. For each frame I extract three features: an HSV color histogram (sensitive to scene cuts and lighting changes), edge density via Sobel (sensitive to structural changes and new objects), and mean optical flow magnitude via Lucas-Kanade (sensitive to motion onset and action). These are combined into a weighted frame difference score $D_t$ comparing each frame to the previous one.

Keyframes are selected in three stages. First, a global threshold removes unimportant frames. Second, local maximum suppression in a temporal window keeps only the peak of each change event, avoiding redundant nearby frames during gradual transitions. Third, a minimum temporal gap is enforced to prevent over-sampling in busy segments.

Each step is justified by the type of event it targets: color histograms catch scene changes, edge density catches structural novelty, optical flow catches meaningful motion. The combination makes the detector robust to the variety of events the question describes: scene cuts, new objects, expressions, and speaker switches.