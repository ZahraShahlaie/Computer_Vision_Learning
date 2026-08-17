

# YOLOv2 (YOLO9000) — The Second Generation of Real-Time Object Detection

YOLOv1 was able to perform the entire object detection process in a single neural network, unlike two-stage methods such as R-CNN.

However, despite its very high speed, it had several important limitations:

* Lower accuracy (mAP) compared to other methods
* Weak performance in detecting small objects
* Using Fully Connected layers, which increased computational cost
* Inefficient use of initial Bounding Boxes

To solve these problems, a few months later, the second version of YOLO was introduced with the name:

> **YOLO9000: Better, Faster, Stronger**

---

# Why Was It Named YOLO9000?

The number 9000 was chosen because the model was able to recognize more than 9000 different object classes.

In fact, YOLOv2 was not only an Object Detection model trained on the COCO dataset, but by combining data from:

* ImageNet Classification Dataset
* COCO Detection Dataset

it was able to learn a very large number of object classes.

Therefore, the name:

```
YOLO9000
```

refers to the ability of the model to detect more than 9000 classes.

---

# Speed and Accuracy Comparison Between YOLOv1 and YOLOv2

One of the main goals of YOLOv2 was to maintain high speed while improving accuracy.

In YOLOv2, different input image sizes were tested:

| Input Size | mAP     |
| ---------- | ------- |
| 256×256    | Lower   |
| 320×320    | Better  |
| 416×416    | Higher  |
| 480×480    | Higher  |
| 544×544    | Highest |

The result:

As image resolution increases:

* More image details are preserved.
* Small objects are detected better.
* mAP increases.

However, on the other hand:

* The number of computations increases.
* The inference speed decreases.

Therefore, there is a trade-off between:

```
Accuracy  <-------> Speed
```

YOLOv2 was able to significantly improve accuracy compared to YOLOv1 while maintaining acceptable real-time speed.

---

# YOLOv1 Problem with Grid and Detecting Multiple Objects

In YOLOv1, the image is divided into a Grid:

```
7×7
```

This means the image is divided into 49 Cells.

Each Cell:

* Is responsible for predicting an Object whose center lies inside that Cell.
* Predicts two Bounding Boxes.

Each Bounding Box contains:

```
x , y , w , h , Confidence
```

However, there was an important limitation:

Each Cell had only one Class Probability Vector.

This means the two Bounding Boxes inside the same Cell shared the same class prediction.

For example, if the centers of two Objects are inside one Cell:

```
Dog + Cat
```

The model cannot say:

```
Box 1 → Dog

Box 2 → Cat
```

because there is no separate Class Prediction for each Bounding Box.

Therefore, YOLOv1 practically assumes:

> Each Cell is responsible for only one Object.

---

# Difference Between Local and Global Prediction in YOLOv1

In YOLOv1, object responsibility is determined locally.

Meaning:

The center of the Object determines which Cell is responsible for it.

However, a Bounding Box can be very large and cover a large part of the image.

Therefore:

* Responsibility assignment → Local based on Cell
* Bounding Box dimensions → Global with respect to the entire image

---

# Main Architectural Change in YOLOv2

## YOLOv1 Architecture

In YOLOv1, the architecture was:

```
Input Image
      ↓
CNN Backbone
(Feature Extraction)
      ↓
Fully Connected Layer
      ↓
Prediction
(Bounding Box + Class)
```

The main problem:

The existence of Fully Connected layers.

---

# Why Were Fully Connected Layers a Problem?

In Fully Connected layers:

Each neuron is connected to every neuron in the previous layer.

Therefore:

* The number of parameters becomes very large.
* More memory is consumed.
* Computational cost increases.

For example:

```
Feature Map
      |
      |
Fully Connected
      |
Prediction
```

made the network heavier.

---

# Removing Fully Connected Layers in YOLOv2

In YOLOv2, the network became fully convolutional.

The new architecture:

```
Input Image
      ↓
Convolutional Backbone
      ↓
Feature Extraction
      ↓
Convolutional Prediction Layer
      ↓
Bounding Box + Class Prediction
```

Meaning:

There is no Dense Layer anymore.

Advantages:

* Fewer parameters
* Lower memory usage
* Faster inference speed

---

# Removing Fully Connected Layers and Preserving Global Information

In YOLOv1:

```
Feature Map
↓
Fully Connected
↓
Prediction
```

was used.

The problem:

Fully Connected layers had a huge number of parameters.

Because every neuron was connected to all neurons in the previous layer.

Result:

* More computation
* More memory usage
* Lower speed

In YOLOv2, the network became fully convolutional:

```
Feature Map
↓
1×1 Convolution
↓
Prediction
```

However, removing Fully Connected layers does not mean that the network becomes only local.

Because in deep CNNs:

As the number of layers increases:

* The Receptive Field becomes larger.
* Information from different regions of the image is combined.

Therefore:

Early layers see:

```
Local Features
```

Deep layers receive:

```
Global Context
```

---

# New Backbone: Darknet-19

YOLOv2 introduced a new Backbone called:

```
Darknet-19
```

The number 19 represents the number of layers in the network.

General structure:

```
Image
 ↓
Darknet-19
 ↓
Feature Maps
 ↓
Detection Head
```

---

## Comparison Between Darknet-19 and GoogleNet

Compared to GoogleNet, Darknet-19:

* Has lower computational cost.
* Provides higher speed.
* Achieves better accuracy.

Result:

Around:

```
30% reduction in computational cost
```

was achieved.

Therefore, the two main factors increasing YOLOv2 speed were:

### 1. Removing Fully Connected Layers

### 2. Using the lightweight Darknet-19 Backbone

---

# Using Anchor Boxes in YOLOv2

One of the most important changes in YOLOv2 was the use of:

```
Anchor Boxes
```

---

## YOLOv1 Problem

In YOLOv1, the network directly predicted Bounding Boxes.

Meaning:

The network had to learn:

* Size
* Aspect ratio
* Location

of Bounding Boxes by itself.

This was difficult.

---

# Anchor Box Idea

In YOLOv2:

Before training, several initial Bounding Box shapes are defined.

For example:

```
Small Box
Medium Box
Large Box
Tall Box
Wide Box
```

These boxes become Anchors.

The network only learns how much these Anchors should be adjusted.

---

# How Were Anchor Boxes Selected?

In common approaches, Anchor Boxes are manually selected.

However, YOLOv2 used:

```
K-Means Clustering
```

The goal:

Finding the best initial Bounding Box sizes.

---

## Why Was Standard K-Means Not Suitable?

Because K-Means uses Euclidean distance:

```
distance = √((w1-w2)^2+(h1-h2)^2)
```

But the appropriate metric for Bounding Boxes is:

```
IoU
```

---

# New Distance Metric in YOLOv2

YOLOv2 defined:

```
Distance = 1 - IoU
```

Meaning:

The more similar two Bounding Boxes are:

Higher IoU

and lower distance.

Therefore, K-Means works based on the real similarity between Bounding Boxes.

---

# Result of Using Anchor Boxes

With this method:

* The network converged faster.
* More accurate Bounding Boxes were generated.
* mAP increased.

---

# Anchor Boxes and Increasing Predictions in YOLOv2

In YOLOv1:

Each Cell had only two Bounding Boxes and predictions were limited.

However, YOLOv2 used:

```
Anchor Boxes
```

For example:

```
13×13 Feature Map
Each Cell:
5 Anchor Boxes
```

Each Anchor has an independent prediction.

Each Anchor contains:

```
x,y,w,h
Confidence
Class Probability
```

Unlike YOLOv1 where two Boxes shared the same class prediction, in YOLOv2 each Anchor can make an independent decision.

---

# Increasing the Number of Predictions

In YOLOv1:

Each Cell had a limited number of predictions.

In YOLOv2:

Each Cell contains multiple Anchor Boxes.

For example:

```
Feature Map = 13×13
Anchor Boxes = 5
```

Therefore:

Each Cell predicts:

```
5 Bounding Boxes
```

---

# Using Higher Resolution Feature Maps

One of the problems of YOLOv1:

Poor detection of small Objects.

The reason:

In deep CNN layers:

Feature Maps become smaller.

For example:

```
416×416 Image
↓
13×13 Feature Map
```

A lot of spatial information is lost.

---

# Grid Change in YOLOv2

One problem in YOLOv1 was the small final Feature Map.

In YOLOv1:

```
Feature Map = 7×7
```

Therefore, for small objects:

* Spatial information was limited.
* Some small objects disappeared.
* mAP decreased.

In YOLOv2:

To improve this problem:

```
Feature Map = 13×13
```

was used.

Meaning the number of Cells increased:

YOLOv1:

```
7×7 = 49 Cells
```

YOLOv2:

```
13×13 = 169 Cells
```

Therefore, the network had better spatial localization for small objects.

---

# Why Was 13×13 Not Enough?

Even a 13×13 Feature Map may not be sufficient for very small objects.

Therefore, YOLOv2 introduced:

```
Passthrough Layer
```

The idea was to transfer higher-resolution Feature Maps to deeper layers.

For example:

Before:

```
26×26×512
```

After Passthrough:

```
13×13×2048
```

Meaning:

Spatial resolution decreases, but information is not removed; it is transferred from spatial dimensions into channel dimensions.

---

# Why Not Use 26×26 Directly?

If a Feature Map of:

```
26×26
```

was directly used:

The number of predictions would increase significantly.

Because in YOLOv2:

Each Cell has:

```
5 Anchor Boxes
```

Therefore:

13×13:

```
13×13×5 = 845 Boxes
```

But:

26×26:

```
26×26×5 = 3380 Boxes
```

This would greatly increase computational cost.

Passthrough helped:

* Preserve details.
* Avoid too many predictions.
* Maintain real-time speed.

---

# Using Passthrough Layer

YOLOv2 introduced a new technique:

```
Passthrough Layer
```

The idea:

Instead of discarding early Feature Maps, transfer their information to deeper layers.

---

## Simple Example

Assume we have a Feature Map:

```
4×4×1
```

With Passthrough it becomes:

```
2×2×4
```

Meaning:

Spatial resolution decreases, but:

Depth increases.

Information is not removed.

---

Advantages:

The network has both:

* Low-level information (details)
* High-level information (object semantics)

at the same time.

Result:

Better detection of small objects.

---

# Why Is the Output 125 Channels?

In YOLOv2:

Number of Anchor Boxes:

```
5
```

For each Anchor, we need:

### Bounding Box:

```
x,y,w,h
```

Four values

### Confidence:

One value

### Class:

20 classes

Therefore:

```
5 × (4 + 1 + 20)
```

```
5 × 25
=125
```

Therefore, the output is:

```
13 × 13 × 125
```

---

# Why Was YOLOv2 Better Than YOLOv1?

Main changes:

| Change                          | Result                        |
| ------------------------------- | ----------------------------- |
| Removing Fully Connected Layers | Higher speed                  |
| Darknet-19                      | Lower computational cost      |
| Anchor Boxes                    | Higher accuracy               |
| K-Means based on IoU            | Better Box selection          |
| Passthrough Layer               | Better small object detection |
| Fully Convolutional Design      | More flexibility              |

---

# Final Summary

YOLOv2 was a major improvement over YOLOv1.

The most important ideas were:

1. Converting the network into a Fully Convolutional architecture
2. Using the lightweight Darknet-19 Backbone
3. Introducing Anchor Boxes
4. Using K-Means based on IoU
5. Using Passthrough Layer to preserve detailed information
6. Increasing the number of predictions

Result:

* Real-time speed was maintained.
* mAP increased.
* Small object detection improved.
* The architecture became the foundation for many later YOLO versions.

---

**YOLOv2 showed that building a fast Object Detector is not only about increasing network depth; instead, architecture design, Bounding Box generation strategy, and intelligent use of Feature Maps are equally important.**
