# YOLOv3 Architecture

Nowadays, in real-world projects, people usually do not use the original YOLOv1 or YOLOv3 models. Instead, newer versions such as YOLOv8, YOLOv9, YOLOv10, and others are commonly used.

So why do we study older architectures such as YOLOv1, YOLOv2, and YOLOv3?

The main reason is that understanding the fundamental architectures helps us:

* Analyze newer architectures more effectively.
* Understand what problem each architectural change solves.
* Tune Hyperparameters more intelligently.
* Modify and customize architectures when needed.
* Analyze model behavior more accurately.

For example, if we do not understand what Anchor Boxes do, we cannot decide:

* Whether to increase or decrease the number of Anchors.
* Which Anchor Box sizes are suitable.
* When using fewer Anchor Boxes is sufficient.

Therefore, the main goal is understanding the **philosophy behind architectures**, not simply using a pre-trained model.

---

# Main Changes of YOLOv3 Compared to YOLOv2

YOLOv3 introduced several important changes compared to YOLOv2:

1. Changing the network Backbone
2. Using Multi-Label Classification
3. Using Feature Pyramid Network (FPN)
4. Using Multi-Scale Prediction
5. Increasing the number of Anchor Boxes

---

# 1. Backbone Change in YOLOv3

## YOLOv2 Backbone

YOLOv2 used:

```
Darknet-19
```

This network contained approximately 19 convolutional layers.

The main goals of Darknet-19 were:

* High speed
* Fewer parameters
* Suitable for Real-Time Detection

---

# YOLOv3 Backbone

In YOLOv3, the Backbone was changed to:

```
Darknet-53
```

Meaning the network became deeper:

```
Darknet-19

        ↓

Darknet-53
```

Increasing depth allowed the model to:

* Extract more complex features.
* Increase detection accuracy.
* Build a stronger representation.

However, a major problem appears.

---

# Problem of Deeper Networks: Vanishing Gradient

As networks become deeper, the problem of gradient propagation increases.

During Backpropagation, the Gradient may become extremely small in early layers.

As a result:

* Early layers are not trained effectively.
* Learning becomes more difficult.
* Accuracy may decrease.

To solve this problem, the idea of:

# Skip Connection

was introduced.

---

# What Is Skip Connection?

Skip Connection was widely introduced in ResNet.

The main idea:

Instead of allowing information to pass through only one path:

```
Input

 ↓

Conv

 ↓

Conv

 ↓

Output
```

a direct path is also created:

```
          ┌──────────────┐
          │              │
Input → Conv → Conv → (+) → Output
          │              │
          └──────────────┘
```

Meaning that part of the information is transferred directly to the next layer without modification.

---

## Advantages of Skip Connection

### 1. Improving Gradient Flow

The Gradient can reach early layers more easily.

### 2. Enabling Deeper Networks

The network can become deeper without suffering from Vanishing Gradient problems.

### 3. Increasing Accuracy

The network learns better and more complex features.

---

# Backbone Comparison

When selecting a Backbone, there is always a Trade-off:

```
Accuracy  ↔  Speed
```

Meaning:

* Larger models → Higher accuracy but lower speed
* Smaller models → Higher speed but lower accuracy

---

## Darknet-19

Advantages:

* High speed
* Suitable for Real-Time Detection

Disadvantages:

* Lower accuracy

---

## Darknet-53

Advantages:

* Higher accuracy
* Better Feature Extraction

Disadvantages:

* More computations
* Lower speed

---

# 2. Multi-Label Classification in YOLOv3

Another major change in YOLOv3 was related to Classification.

---

## YOLOv2 Method

YOLOv2 used:

```
Softmax
```

for Classification.

Softmax calculates class probabilities so that:

```
Sum(probabilities)=1
```

For example:

```
Dog      0.05
Cat      0.15
Human    0.80
```

The model selects only the class with the highest probability.

---

# Problem with Softmax

In some datasets, classes can overlap.

For example, an image:

```
A woman on the street
```

can simultaneously belong to:

```
Woman
Human
Person
```

However, Softmax forces the model to select only one class.

---

# YOLOv3 Solution

YOLOv3 replaced Softmax with:

```
Logistic Regression + Sigmoid
```

In this approach, each class is predicted independently.

For example:

```
Human      0.95
Woman      0.92
Person     0.94
Car        0.02
```

Therefore, one Object can have multiple Labels.

---

# 3. Multi-Scale Prediction

One of the problems of YOLOv2:

Poor detection of small Objects.

The reason:

In CNN networks, as we move deeper:

* Feature Maps become smaller.
* Semantic information increases.
* Spatial information decreases.

---

Example:

```
Input Image

512×512


↓


52×52 Feature Map

High Spatial Information


↓


26×26 Feature Map


↓


13×13 Feature Map

High Semantic Information
```

---

## Problem with Small Feature Maps

In small Feature Maps:

* Large Objects are detected well.
* Small Objects may disappear.

Because there is not enough spatial information.

---

# Feature Pyramid Network (FPN)

To solve this problem, YOLOv3 used the idea of Feature Pyramid Network.

The idea of FPN:

Combining:

1. Strong semantic features from deep layers
2. Strong spatial features from early layers

---

# Concept of Semantic and Spatial Information

## Deeper Layers

For example:

```
13×13
```

Characteristics:

* High Semantic Information
* Weak Localization

Meaning the model better understands:

"What is it?"

---

## Earlier Layers

For example:

```
52×52
```

Characteristics:

* High Spatial Information
* Weaker Semantic Information

Meaning the model better understands:

"Where is it located?"

---

# FPN Operation

Assume we have Feature Maps:

```
52×52

26×26

13×13
```

First:

The 13×13 Feature Map is Upsampled:

```
13×13

      ↓

26×26
```

Then it is combined with the 26×26 Feature Map:

```
Concat
```

Result:

```
26×26 Feature Map
```

which contains:

* Semantic information
* Spatial information

---

The same process is repeated for:

```
52×52
```

Finally, three Prediction scales are created:

```
13×13

26×26

52×52
```

---

# Advantage of Multi-Scale Prediction

The model can detect:

## Large Objects

Using:

```
13×13
```

## Medium Objects

Using:

```
26×26
```

## Small Objects

Using:

```
52×52
```

---

# 4. Anchor Boxes in YOLOv3

In YOLOv2:

```
5 Anchor Boxes
```

were used.

However, YOLOv3 used:

```
9 Anchor Boxes
```

These 9 Anchors were distributed between three Feature Maps:

```
13×13 → 3 Anchors

26×26 → 3 Anchors

52×52 → 3 Anchors
```

---

# How Are Anchor Boxes Selected?

Similar to YOLOv2:

```
K-Means Clustering
```

is used.

However, standard distance is not used.

Instead:

```
IoU Distance
```

is used.

---

# Anchor Distribution

Anchors are sorted based on their size.

For example:

```
Largest Anchors

        ↓

      13×13
```

Because smaller Feature Maps are suitable for large Objects.

And:

```
Smallest Anchors

        ↓

       52×52
```

Because larger Feature Maps detect small Objects better.

---

# YOLOv3 Speed

Due to changes such as:

* Larger Backbone
* More Anchors
* Multi-scale Prediction
* FPN

the computational cost increased.

---

# YOLOv3 Limitations

Although YOLOv3 significantly improved small Object detection, in some situations errors increased for:

* Medium Objects
* Large Objects

This shows that every architecture has its own Trade-off.

---

# Final Result

The most important ideas introduced by YOLOv3 were:

* Using a deeper Backbone with Residual Connections
* Multi-Label Classification
* Feature Pyramid Network
* Multi-Scale Detection
* More Anchor Boxes

These concepts were later continued in newer YOLO architectures and many other Object Detection models.

Understanding YOLOv3 helps us analyze newer architectures such as YOLOv5, YOLOv8, and YOLOv10 because many of their ideas are rooted in these fundamental concepts.
