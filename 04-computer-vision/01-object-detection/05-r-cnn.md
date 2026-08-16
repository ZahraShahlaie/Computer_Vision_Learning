

# Two-Stage Object Detection and Region Proposal

## Introduction

In Object Detection, a model must solve two main tasks simultaneously:

1. **Classification** → Identify the class of the object.
2. **Localization** → Determine the exact location of the object using a Bounding Box.

However, a new challenge appears when an image contains multiple objects.

For example:

```
Airplane
Tree
Car
Bird

```

The model must be able to:

* Find all possible objects.
* Classify each object.
* Predict the location of each object.
* Generate the corresponding Bounding Boxes.

To solve this problem, Object Detection architectures are generally divided into two main categories:

```
Object Detection
       |
       ├── Two-Stage Detectors
       |
       └── Single-Stage Detectors

```

This section focuses on **Two-Stage Object Detection** and the concept of **Region Proposal**.

---

# Two-Stage Object Detection

Two-Stage Detectors are one of the earliest and most important approaches in Object Detection.

The main idea is to divide the detection process into two separate stages:

```
Input Image
      |
      ↓
Stage 1:
Region Proposal
      |
      ↓
Candidate Regions
      |
      ↓
Stage 2:
Classification + Bounding Box Refinement
      |
      ↓
Final Detection Results

```

---

# Stage 1 — Region Proposal

The goal of the first stage is not to classify objects directly.

Instead, the model tries to find image regions that are likely to contain objects.

These candidate regions are called:

> **Region Proposals**

The purpose of this stage is:

* Reduce the search space.
* Generate possible object locations.
* Remove unnecessary background regions.

For example:

```
Image

 |
 ├── Region 1
 ├── Region 2
 ├── Region 3
 ├── Region 4
 └── ...

```

Each region represents a possible location where an object might exist.

---

# Stage 2 — Classification and Bounding Box Regression

After generating Region Proposals, each candidate region is processed by a CNN.

The network answers three important questions:

1. Does this region contain an object?
2. What is the class of the object?
3. How can the Bounding Box be refined?

The overall pipeline is:

```
Image
  |
  ↓
Region Proposal
  |
  ↓
CNN Feature Extractor
  |
  ├── Classification
  |
  └── Bounding Box Refinement

```

The second stage improves the initial proposals and produces more accurate detections.

---

# The Problem of Region Proposal Generation

A key question is:

> How can we generate Region Proposals efficiently?

An image contains many different areas:

* Sky
* Ground
* Background
* Buildings
* Objects
* People

Checking every possible region is computationally expensive.

Therefore, we need efficient methods to find meaningful candidate regions.

---

# Sliding Window

One of the simplest approaches for generating Region Proposals is:

> **Sliding Window**

The idea is to move a fixed-size window across the image and analyze each region.

For example:

```
Window Size = 100 × 100

Stride = 50 pixels

```

Meaning:

* A 100×100 window is created.
* The window moves 50 pixels each step.
* Each position becomes a possible Region Proposal.

Example:

```
Image

┌──────────┐
│ Window   │
└──────────┘

        ↓ move

┌──────────┐
│ Window   │
└──────────┘

```

---

# Problems with Sliding Window

Although Sliding Window is simple, it has several limitations.

---

## 1. Huge Number of Windows

For a large image, thousands or millions of windows may be generated.

Example:

```
1 Image

↓

100000 Region Proposals

```

Each region would require CNN processing, causing enormous computation.

---

## 2. Different Object Sizes

Objects can appear in different sizes.

For example:

* Large car
* Small bird
* Medium-sized person

Therefore, multiple window sizes are required:

```
10×10
50×50
100×100
200×200
...

```

This dramatically increases the number of possible regions.

---

## 3. No Understanding of Image Content

Sliding Window does not understand the image.

It scans every location, including irrelevant areas.

For example:

* Empty sky
* Background
* Ground

are also processed.

Therefore:

```
More Computation
        +
Lower Efficiency
        +
Slower Detection

```

---

# Selective Search

To overcome the limitations of Sliding Window, a method called:

> **Selective Search**

was introduced.

Selective Search became an important component of early Object Detection methods, especially **R-CNN**.

The main idea:

Instead of blindly scanning the entire image, Selective Search intelligently proposes regions that are more likely to contain objects.

---

# How Selective Search Works

Selective Search is based on image segmentation concepts.

Unlike Object Detection, where we usually predict Bounding Boxes, segmentation focuses on identifying accurate pixel-level regions.

The general process:

1. Divide the image into smaller regions.
2. Measure similarity between regions.
3. Merge similar regions.
4. Generate possible Bounding Boxes.

Pipeline:

```
Image

 ↓

Initial Regions

 ↓

Region Similarity

 ↓

Region Merging

 ↓

Region Proposals

```

---

# Why Selective Search is Better than Sliding Window

Sliding Window may generate:

```
100000+ proposals

```

while Selective Search typically generates around:

```
≈ 2000 proposals

```

This significantly reduces computation.

Advantages:

* Fewer candidate regions.
* Faster processing.
* More meaningful proposals.

---

# Limitation of Selective Search

Although Selective Search is better than Sliding Window, it still has a major drawback.

For a single image:

```
1 Image

↓

~2000 Region Proposals

```

Each region must be processed separately by a CNN.

This means:

```
2000 CNN Forward Passes

```

for only one image.

This makes the method slow and unsuitable for real-time applications.

---

# Real-Time Detection Challenge

Object Detection is commonly used in applications such as:

* Security cameras
* Autonomous vehicles
* Surveillance systems

A typical video stream may contain:

```
30 FPS

```

meaning:

```
30 images per second

```

If processing each image takes several seconds, the system cannot operate in real-time.

Therefore, early Two-Stage methods had limitations in speed.

---

# R-CNN and Selective Search

The original R-CNN pipeline works as follows:

```
Input Image

      ↓

Selective Search

      ↓

~2000 Region Proposals

      ↓

Resize Regions

      ↓

CNN Feature Extraction

      ↓

Classifier

      ↓

Bounding Box Regression

```

The model first finds possible object regions and then analyzes each region using CNN.

---

# Main Problems of R-CNN

Although R-CNN achieved high accuracy, it had several problems:

* Large computational cost.
* Slow training process.
* Slow inference speed.
* Not suitable for real-time applications.

The main reason:

Each Region Proposal is independently passed through the CNN.

```
Region 1 → CNN
Region 2 → CNN
Region 3 → CNN
...
Region 2000 → CNN

```

This repeated computation makes R-CNN inefficient.

---

# Summary

Two-Stage Object Detection follows a two-step strategy:

```
Image
  ↓
Region Proposal
  ↓
Feature Extraction
  ↓
Classification
  +
Bounding Box Regression

```

The main ideas:

* **Sliding Window** generates candidate regions but is computationally expensive.
* **Selective Search** improves Region Proposal generation by reducing unnecessary regions.
* **R-CNN** combines Selective Search with CNN-based classification and localization.
* Two-Stage methods achieve high accuracy but suffer from slower inference speed.

The limitations of R-CNN motivated the development of improved methods such as:

```
R-CNN
   ↓
Fast R-CNN
   ↓
Faster R-CNN
   ↓
Single-Stage Detectors (YOLO, SSD)

```

which aim to achieve a better balance between:

```
Accuracy
     +
Speed
     +
Computational Cost

```
