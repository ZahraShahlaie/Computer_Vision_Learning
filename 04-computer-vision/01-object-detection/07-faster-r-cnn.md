# Faster R-CNN

**Fast R-CNN** solved one of R-CNN's biggest problems: the expensive **feature extraction** step was performed only once on the whole image, and all region proposals reused that same shared feature map.

But one important problem still remained: region proposals were still generated using the **Selective Search** algorithm.

This step had become the system's bottleneck, preventing the model from reaching the speed needed for real-time applications. That's why, in 2015, the **Faster R-CNN** model was introduced — bringing one of the most important changes in the history of object detection.

---

## The Main Problem With Fast R-CNN

Fast R-CNN still relied on **Selective Search** to find candidate regions that might contain objects. This algorithm had two fundamental problems:

### 1. Slow Speed

Selective Search is a classical image-processing algorithm. It generates about **2,000 region proposals** per image, and processing this many regions significantly increases computational cost.

### 2. Not Trainable

The most important weakness of Selective Search is that it **doesn't learn**. That means:

- Its parameters are fixed.
- It doesn't train on images.
- Its performance doesn't improve as the dataset changes.

In other words, it processes every image using the same predefined rules. As a result, it might:

- Propose many regions that don't actually contain an object.
- Miss some real objects entirely.

This hurts both speed and accuracy.

---

## The Core Idea of Faster R-CNN

The solution was simple but clever: instead of using a classical algorithm like Selective Search, use a **neural network** instead. This network learns on its own where objects typically appear in an image.

This network is called the **Region Proposal Network (RPN)**.

---

## Region Proposal Network (RPN)

RPN stands for **Region Proposal Network**. Its job is exactly one thing:

> Find regions that are likely to contain an object.

Note that the RPN doesn't yet determine *what* that object is — it only says:

> "There's a high probability of an object in this area."

Classifying the object happens in later stages.

---

## Faster R-CNN Architecture

The overall architecture of Faster R-CNN is as follows:

```text
Image
   │
   ▼
CNN (Backbone)
   │
   ▼
Feature Maps
   │
   ├──────────────┐
   │              │
   ▼              ▼
   RPN        ROI Pooling
   │              ▲
   └────Region Proposals────┘
               │
               ▼
      Fully Connected
        │           │
        ▼           ▼
 Classification   Bounding Box Regression
```

---

## Steps of the Faster R-CNN Pipeline

### Step 1

The image is passed through the CNN. The network processes the image only **once**. The output of this step is a set of feature maps.

### Step 2

Instead of generating region proposals on the original image, they're now generated directly on the **feature map**. This is exactly what the RPN does.

### Step 3

The RPN slides over the feature map and finds regions likely to contain an object. The output of this step is a set of region proposals.

### Step 4

The region proposals are fed into **ROI Pooling**, which resizes all regions to the same size so they can be fed into the fully connected layers.

### Step 5

Finally, the network produces two outputs:

- Object classification
- Bounding box regression (coordinate refinement)

---

## Why Does the RPN Use the Feature Map?

As we learned with Fast R-CNN, the feature map is a compressed version of the original image, containing all the important image information. There's no need to redo processing on the original image.

Using the feature map means:

- Higher speed.
- Lower memory usage.
- Simpler processing.

---

## What Happens Inside the RPN?

The RPN is a small neural network whose input is the feature map. First, a convolutional layer is applied to the feature map, preparing it for finding suitable regions.

Since this layer is trainable, its parameters are learned through backpropagation. This means, unlike Selective Search, the RPN can adapt itself to the dataset.

---

## What Is an Anchor Box?

One of the most important ideas in Faster R-CNN is the use of **anchor boxes**.

Anchor boxes are a set of predefined bounding boxes that slide across the feature map. In simpler terms: imagine placing a small window on the feature map. This window slides across every position on the feature map, and at each position it generates several bounding boxes with different sizes and aspect ratios, such as:

- Square
- Horizontal rectangle
- Vertical rectangle
- Small
- Medium
- Large

This allows objects of different sizes to be detected more effectively.

---

## Why Do We Need Multiple Anchor Boxes?

Not all objects have the same size or shape. For example:

- A car is large.
- A bottle is small.
- A person is elongated.
- A TV is roughly square.

If we only had one type of bounding box, many objects wouldn't be covered well. That's why multiple anchor boxes with different sizes and aspect ratios are placed at each position.

---

## What Does Each Anchor Box Output?

Two things are computed for each anchor box.

### 1. Objectness Score

The first question is:

> Does this anchor box contain an object or not?

This is a binary classification problem. The output is typically produced by a sigmoid function, giving a value between 0 and 1:

- Close to 0 → low probability of an object.
- Close to 1 → high probability of an object.

This value is called the **objectness score**.

### 2. Bounding Box Regression

The anchor box might not be positioned exactly on the object — it could be slightly larger or smaller. At this stage, the network adjusts the bounding box coordinates so it fits tightly around the object. This process is called **bounding box regression**.

---

## The Problem of Duplicate Bounding Boxes

Since several anchor boxes can land on the same object, multiple nearly identical bounding boxes may be produced. For example, a single car might be detected by three different anchor boxes, resulting in three almost identical bounding boxes. Clearly, only one of them should remain.

---

## The Solution: Non-Maximum Suppression (NMS)

To remove duplicate bounding boxes, the **Non-Maximum Suppression (NMS)** algorithm is used. Its steps are as follows:

### Step 1

All bounding boxes are sorted by **objectness score**.

### Step 2

The bounding box with the highest score is selected.

### Step 3

The IoU between this bounding box and all others is computed. If the IoU exceeds a threshold (e.g., 0.7), both bounding boxes are considered to represent roughly the same object. In this case, the box with the lower score is removed.

### Step 4

This process continues until no duplicate bounding boxes remain — leaving only the best bounding boxes.

---

## Why Is Faster R-CNN Faster?

There are two main reasons:

### Reason 1

Selective Search has been removed entirely.

### Reason 2

The RPN itself is very small. While the overall network may have tens of millions of parameters, the RPN makes up only a small fraction of it, so its computational cost is very low.

---

## Fast R-CNN vs. Faster R-CNN

| Feature | Fast R-CNN | Faster R-CNN |
|---|---|---|
| Region proposal generation | Selective Search | Region Proposal Network (RPN) |
| Trainable | ❌ No | ✅ Yes |
| Speed | Moderate | Much faster |
| Number of proposals | ~2,000 | ~300 (sometimes even fewer) |
| Computational cost | High | Lower |
| Accuracy | Good | Better |
| Suitable for real-time | Limited | Much more suitable |

---

## Advantages of Faster R-CNN

- Removes Selective Search
- Uses a Region Proposal Network
- Makes the proposal-generation stage trainable
- Dramatically reduces computational cost
- Reduces the number of region proposals
- Increases processing speed
- Improves detection accuracy
- Shares the feature map between the RPN and the detector
- Uses anchor boxes to detect objects of different sizes
- Removes duplicate bounding boxes using NMS

---

## Summary

**Faster R-CNN** is one of the most important advances in object detection. In this model, the slow, non-trainable **Selective Search** algorithm was replaced with a neural network called the **Region Proposal Network (RPN)**. This network operates directly on the feature maps extracted from the image, proposing likely object regions more accurately and more quickly, and since it's trainable, it improves its performance as it learns from the training data.

Using **anchor boxes**, objects of different sizes and aspect ratios are covered effectively, and finally, **Non-Maximum Suppression (NMS)** removes duplicate bounding boxes, leaving only the best proposals for the final detection stage. The result of these changes is a dramatic improvement in both speed and accuracy compared to Fast R-CNN, making Faster R-CNN one of the most influential two-stage object detection models.
