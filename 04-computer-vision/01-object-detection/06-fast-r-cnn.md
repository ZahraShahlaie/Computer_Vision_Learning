# Fast R-CNN: A Faster, More Optimized Version of R-CNN

Although R-CNN achieves high accuracy in object detection, its performance is extremely poor in terms of speed.

The biggest problem with R-CNN is its enormous computational cost — processing a single image takes about 45 seconds.

This is completely unacceptable for real-world applications, since object detection is usually expected to run in real time.

## Why Does Speed Matter in Object Detection?

Suppose a camera is recording video at 30 frames per second (30 FPS). This means 30 images are produced every second. To analyze the video in real time, we need to be able to process all 30 images within that same one second.

If processing a single image takes 45 seconds:

```
Image 1 → 45 seconds
Image 2 → 45 seconds
...
```

It's clear that such a model could never keep up with real-time video processing. This is why processing speed is one of the most important design criteria for object detection algorithms.

## What Is the Main Problem With R-CNN?

Let's review the steps of R-CNN.

First, the image is passed to Selective Search. Selective Search generates about 2,000 region proposals. Then, for each region proposal, the following steps are performed:

```
Region Proposal
        │
        ▼
       CNN
        │
        ▼
Feature Extraction
        │
        ▼
Classification + Bounding Box Regression
```

This is exactly where the problem lies. With 2,000 region proposals, we need to:

- Pass the image through the CNN 2,000 times.
- Perform feature extraction 2,000 times.

Even though most of these regions overlap heavily with each other. As a result, the network repeats essentially the same work over and over. This is the biggest bottleneck in the R-CNN model.

## The Core Idea of Fast R-CNN

The designers of Fast R-CNN targeted exactly this problem. The idea was very simple but clever.

Instead of:

```
Each Region
        │
        ▼
       CNN
```

the entire image is passed through the CNN only **once**:

```
Image
   │
   ▼
CNN (only once)
   │
   ▼
Feature Map
```

This feature map is then reused for all region proposals. As a result, the very expensive feature-extraction step is performed only a single time.

## Fast R-CNN Architecture

The overall architecture of Fast R-CNN is as follows:

```
    ______ Image
   │         │
   │         ▼
   │        CNN
   │         │
   │         ▼
   │     Feature Map
   │         │_________
   │                   │
   │                   │
   ▼                   ▼
Selective Search   ROI Pooling  ----> Fully Connected
        │               ▲
        └───────────────┘             │           │
                                      ▼           ▼
                              Classification   Bounding Box Regression
```

## Steps of the Fast R-CNN Pipeline

### Step 1

The entire image is passed into the CNN.

```
  Image
    │
    ▼
   CNN
```

### Step 2

The CNN processes the image only once. The output of this step is a set of feature maps.

```
  Image
    │
    ▼
   CNN
    │
    ▼
Feature Maps
```

### Step 3

Alongside the CNN, the Selective Search algorithm also runs, responsible for generating region proposals. Fast R-CNN still uses the same Selective Search algorithm — nothing has changed on that front.

### Step 4

The feature maps and region proposals are fed into a layer called **ROI Pooling**. This is where all of Fast R-CNN's main innovation lies.

## What Is an ROI?

**ROI** stands for **Region of Interest** — meaning a region of the image where we suspect an object might be located.

## What Does ROI Pooling Do?

Suppose the original image is:

```
224 × 224
```

After passing through the CNN, the feature map size shrinks to:

```
14 × 14
```

Now, if we have a region proposal in the original image, we need to figure out exactly where that region corresponds to on the feature map. This is exactly what ROI Pooling does.

## Locating the ROI on the Feature Map

Suppose the original image size is 224 × 224, and the feature map size is 14 × 14. The scale factor is then:

```
Scale Factor = Feature Map Size / Image Size
```

For example:

```
14 / 224 = 1 / 16
```

If the ROI coordinates in the original image are:

```
(160, 80)
```

then the corresponding coordinates on the feature map are:

```
160 / 16 = 10
80 / 16 = 5
```

So the ROI on the feature map will be located approximately at:

```
(10, 5)
```

## What Happens After Locating the ROI?

ROI Pooling now crops the corresponding section from the feature map. But there's a problem.

## The Problem of Varying ROI Sizes

Not all objects are the same size. For example:

- A car might have a large region, such as 80 × 120.
- A bird might have a much smaller region, such as 20 × 25.

As a result, extracted ROIs come in different sizes. But a fully connected network only accepts inputs of a fixed size.

## The ROI Pooling Solution

ROI Pooling converts all regions into a **fixed size**. For example, it converts every ROI into 7 × 7. It doesn't matter whether the original region was 15 × 20 or 80 × 120 — the output will always end up being 7 × 7. This is what allows all ROIs to be fed into the fully connected layers.

## The Major Advantage of ROI Pooling

In R-CNN:

```
Each Region
      │
      ▼
     CNN
```

In Fast R-CNN:

```
Image
   │
   ▼
CNN
   │
Feature Maps
   │
ROI Pooling
```

So the expensive feature-extraction step is performed only once, which is what drives the massive speed improvement.

## The Flaw in ROI Pooling

ROI Pooling has an important issue: converting coordinates from the original image to the feature map requires a scaling operation. For example:

```
160 / 16 = 10
```

But if the value is instead:

```
163 / 16 = 10.18
```

the network is forced to round the number:

```
10.18
→
10
```

or

```
11
```

This rounding introduces a small error, known as **misalignment**. This issue was later resolved in the Mask R-CNN model with the introduction of **ROI Align**.

## How Is Fast R-CNN Trained?

After the region proposals are generated, we need to determine which class each region belongs to. To do this, each region proposal is compared against the ground-truth bounding box, using **IoU (Intersection over Union)** as the comparison metric.

If:

```
IoU > 0.5
```

the region is treated as a **positive** sample and is assigned the object's class label.

If:

```
IoU < 0.5
```

the region is treated as **background** and is not used for learning the object.

## Advantages of Fast R-CNN

- Feature extraction is performed only once.
- Much faster than R-CNN.
- Significantly reduced computational cost.
- A single network handles both classification and bounding box regression.
- Simpler, more efficient training than R-CNN.

## The Limitation of Fast R-CNN

Although Fast R-CNN is much faster than R-CNN, one important problem still remains: region proposals are still generated using Selective Search.

Selective Search is a classical image-processing algorithm and is relatively slow. As a result, even though the CNN now runs only once, generating region proposals is still the system's bottleneck.

This limitation led to the next version, **Faster R-CNN**, in which Selective Search was removed entirely and replaced with a **Region Proposal Network (RPN)**.

## R-CNN vs. Fast R-CNN

| Feature | R-CNN | Fast R-CNN |
|---|---|---|
| Number of CNN runs | Once per region proposal | Only once |
| Feature extraction | Multiple times | Once |
| Speed | Very slow | Much faster |
| Computational cost | Very high | Lower |
| Region proposal method | Selective Search | Selective Search |
| ROI Pooling | Not present | Present |
| Main bottleneck | CNN & feature extraction | Selective Search |

## Summary

Fast R-CNN was one of the most important advances in object detection. Its central idea is that the entire image is passed through the CNN only once, and the resulting feature maps are shared across all region proposals. Using **ROI Pooling**, the corresponding section of the feature map for each region is then extracted and resized to a fixed size, so it can be fed into the fully connected layers.

This change dramatically improved training and inference speed compared to R-CNN, without sacrificing accuracy. However, the reliance on Selective Search remained a major bottleneck — one that was resolved in the next generation, **Faster R-CNN**, with the introduction of the **Region Proposal Network (RPN)**.
