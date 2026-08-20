# Image Segmentation — Core Concepts, Datasets, and Model Architecture

In classification, we only want to know **what's in the image**. In object detection, on top of the class, we also determine **where the object is located**. In segmentation, we go a step further: we want to determine, **for every single pixel in the image, which class it belongs to**.

---

## Image Segmentation

In segmentation, the goal is much more precise — the model has to make decisions at the **pixel level**. For every pixel, it decides:

```
Does this pixel belong to the circle?
Or to the background?
```

For example:

```
Background = 0

Object = 1
```

The model's output is no longer a bounding box — it's a matrix the same size as the image:

```
Prediction Mask


0 0 0 0 0
0 1 1 0 0
0 1 1 0 0
0 0 0 0 0
```

Each number represents the class of that pixel.

---

## The Overall Structure of a Segmentation Model

A segmentation model has two important parts:

```
Input Image
      |
      |
      ▼
Segmentation Model
      |
      |
      ▼
Prediction Mask
```

Input:

```
Image

Height × Width × Channels
```

for example:

```
512 × 512 × 3
```

Output:

```
Mask

512 × 512 × 1
```

Why only one channel? Because we only need a single class value for each pixel. For example:

```
Pixel (10,20)

Prediction = 1
```

meaning this pixel belongs to the object.

---

## The Concept of a Mask in Segmentation

In segmentation, the annotation for an image is called a **mask**. A mask is an image that, instead of holding color values, holds the class of each pixel.

For example, in binary segmentation:

```
0 = Background

1 = Object
```

The original image:

```
RGB Image

512×512×3
```

The mask:

```
512×512×1
```

---

## Why Do We Need a Mask?

When training a neural network, we always have two values.

### Ground Truth

The actual correct answer:

```
True Mask
```

### Prediction

The answer the model produced:

```
Predicted Mask
```

These two are then compared:

```
Loss =
Difference(True, Prediction)
```

The network's weights are then corrected through backpropagation. The overall process:

```
     Image
       |
       ▼
     Model
       |
       ▼ 
Prediction Mask

       |
       |
       ▼

Compare With True Mask

       |
       ▼

      Loss

       |
       ▼

  Backpropagation
```

---

## The Structure of a Segmentation Dataset

A segmentation dataset usually has two parts:

```
Dataset

├── Images
│      ├── image_01.png
│      ├── image_02.png
│
└── Masks
       ├── image_01_mask.png
       ├── image_02_mask.png
```

Every image needs a corresponding mask, for example:

```
brain_01.png

brain_01_mask.png
```

---

## A Common Problem With Masks

In many datasets, mask values aren't standardized. For example, we might expect:

```
Background = 0

Tumor = 1
```

but instead find:

```
Background = 0

Tumor = 255
```

or values ranging:

```
0 to 255
```

To fix this, we typically apply thresholding, for example:

```python
mask[mask > 157] = 1
mask[mask <= 157] = 0
```

Result:

Before:

```
0 , 255
```

After:

```
0 , 1
```

---

## Overlaying a Mask on the Image

To see exactly where segmentation was performed, we overlay the mask on the original image. Steps:

1. Read the image.
2. Read the mask.
3. Convert the mask to a binary form.
4. Assign a color to the segmented region.
5. Combine it with the original image.

For example:

```
Original Image


+
 
 
Mask


=

 
 
Segmented Image
```

For transparency, we typically use an alpha value. Formula:

```
Output =
(1-alpha)*Image
+
alpha*Mask
```

For example:

```
alpha = 0.5
```

makes both the original image and the mask visible at the same time.

---

## The General Architecture of Segmentation Models

Segmentation models are usually built on an **encoder–decoder** architecture. Overall structure:

```
  Input Image

      |
      ▼

    Encoder

      |
      ▼

  Feature Map

      |
      ▼

   Decoder

      |
      ▼

Segmentation Mask
```

---

## What Is the Encoder?

The encoder is responsible for extracting features, similar to a standard CNN:

```
Image

100×100×3

        ↓

50×50×64

        ↓

20×20×256

        ↓

5×5×512
```

At this stage:

- The image's spatial size shrinks.
- The number of feature channels increases.

### Encoder Characteristics

The deeper we go:

**Semantic information increases.** For example, the model recognizes:

```
This region looks like a tumor.
```

But:

**Spatial information decreases** — the model no longer knows exactly:

```
Where is the tumor's precise boundary?
```

---

## What Is the Decoder?

The decoder is responsible for reconstructing the image — it takes the small feature map and upsamples it back to full size:

```
5×5

↓

20×20

↓

50×50

↓

100×100
```

The goal: producing the final mask:

```
100×100×1
```

---

## The Problem With a Simple Encoder-Decoder

The core problem:

The encoder produces:

```
High semantic information

Low spatial information
```

But the decoder needs both. For segmentation, simply knowing that:

```
This region is a tumor
```

isn't enough — we need to know:

```
Exactly which pixels are the tumor.
```

---

## Why Is Segmentation Harder Than Detection?

In detection, the model only outputs a box:

```
+---------+
| Object  |
|         |
+---------+
```

But in segmentation, the object's exact boundary must be determined:

```
   ****
 ********
  ******
```

So the number of decisions required is far greater.

Detection:

```
A handful of boxes
```

Segmentation:

```
Millions of pixel-level classifications
```

---

## Auto-Encoders in Segmentation

This architecture — **encoder + decoder** — is often referred to as an **autoencoder**. Why?

The encoder:

```
Image
 ↓
Encoded Representation
```

compresses the image into a compact representation. The decoder:

```
Encoded Representation
 ↓
Output Mask
```

reconstructs the information back from that compressed representation.

---

## Summary

In image segmentation:

- The goal is to determine the class of every pixel.
- The model's output is a mask.
- The mask is the dataset's original annotation.
- Training requires both the image and its ground-truth mask.
- Common architectures follow an encoder-decoder pattern.
- The encoder extracts semantic features.
- The decoder reconstructs the image's original dimensions.
- The main challenge is preserving both spatial and semantic information at the same time.
- Advanced models like U-Net solve this problem using techniques such as skip connections.

Overall structure:

```
          Image
            |
            ▼
        Encoder
            |
            ▼
      Feature Maps
            |
            ▼
        Decoder
            |
            ▼
     Segmentation Mask
```

This is the foundation underlying nearly all well-known segmentation models.
