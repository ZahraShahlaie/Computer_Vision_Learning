# DeepLab for Semantic Segmentation

In the process of learning **Semantic Segmentation**, we first became familiar with the **Autoencoder** architecture. Then, to overcome some of its limitations, we moved on to **U-Net**. In the next step, we will study one of the important and highly influential architectures in segmentation: **DeepLab**.

The main idea behind DeepLab is:

> How can we achieve a **large Receptive Field** without losing too much spatial information from the image, while also keeping the computational cost under control?

---

# 1. Review of Autoencoder

An Autoencoder used for segmentation generally consists of two main parts:

```text
Input Image
     ↓
  Encoder
     ↓
Feature Map
     ↓
  Decoder
     ↓
Segmentation Map
```

### Encoder

The Encoder gradually reduces the spatial size of the Feature Map using operations such as:

* Convolution
* Pooling
* Strided Convolution

As a result:

```text
256 × 256
      ↓
128 × 128
      ↓
64 × 64
      ↓
32 × 32
      ↓
16 × 16
```

Along this path, the semantic information in the image becomes richer; however, at the same time, **spatial information decreases**.

---

# 2. The Main Problem with Autoencoder

In Semantic Segmentation, we need to make a prediction for **every single pixel**.

However, when the Feature Map becomes smaller, precise spatial information is lost.

For example, if we reach a Feature Map with an:

```text
1 / 32
```

resolution ratio, each pixel in that Feature Map approximately represents a large region of the input image.

Therefore, the model may be able to determine:

> "There is an object here."

But it may have difficulty determining the object's boundaries precisely.

---

# 3. How Does U-Net Solve This Problem?

U-Net uses **Skip Connections**.

The idea is to directly transfer information from the Encoder to the Decoder:

```text
Encoder                    Decoder

Feature Map ─────────────→ Feature Map
     ↓                         ↑
     ↓                         ↑
     ↓                         ↑
```

As a result, the Decoder has access to two types of information simultaneously:

### Semantic Information

From the deep Feature Maps of the Encoder.

### Spatial Information

From the early Feature Maps of the Encoder.

Therefore:

> **U-Net combines semantic and spatial information using Skip Connections.**

This is one of the main reasons U-Net became highly successful for Semantic Segmentation.

---

# 4. An Important Limitation of U-Net

Despite the strong performance of U-Net, there is an architectural issue with it.

U-Net has an approximately symmetric structure:

```text
Encoder              Decoder

Convolution          Convolution
Convolution          Convolution
      ↓                    ↑
Convolution          Convolution
Convolution          Convolution
      ↓                    ↑
      Bridge
```

There are many parameters on both sides of the network.

If we want to make the Encoder larger, for example by:

* Adding more layers
* Using Residual Blocks
* Increasing the number of channels

the number of parameters in the Decoder will also usually increase.

As a result:

> Making U-Net larger can significantly increase the number of parameters and computational cost.

DeepLab follows a different idea:

> **Instead of focusing heavily on the Decoder, we try to design the Encoder so that it produces a suitable Feature Map for segmentation.**

---

# 5. The Main Idea of DeepLab

DeepLab tries to achieve two goals simultaneously:

1. **Large Receptive Field**
2. **Appropriate Feature Map Resolution**

One of the most important ideas introduced in DeepLab to achieve these goals is:

# Atrous Convolution

It is also known as:

* **Dilated Convolution**
* **Dilation Convolution**

---

# 6. What Is a Receptive Field?

Before discussing Atrous Convolution, we need to understand the concept of **Receptive Field**.

Receptive Field means:

> The amount of the input image that each cell of a Feature Map can ultimately see or represent.

For example, a Convolution with a:

```text
3 × 3
```

Kernel initially examines only a small region of the image.

However, when multiple Convolution layers are placed one after another, each Feature Map receives information from a larger region of the image.

Therefore:

```text
Depth ↑
Receptive Field ↑
```

As the network becomes deeper, the Receptive Field generally increases.

---

# 7. Ways to Increase the Receptive Field

There are several ways to increase the Receptive Field.

### Method 1: Making the Network Deeper

We can add more Convolution layers:

```text
Conv
 ↓
Conv
 ↓
Conv
 ↓
Conv
```

However, this increases:

* Parameters
* Computation
* Processing time

---

### Method 2: Using a Larger Kernel

For example, instead of:

```text
3 × 3
```

we can use:

```text
7 × 7
```

or an even larger Kernel.

However, this method also increases the number of parameters and computational cost.

---

### Method 3: Pooling

Pooling reduces the Resolution and allows Feature Maps in deeper layers to cover a larger region of the input image.

However, the problem is:

> We lose spatial information.

---

# 8. What Is Atrous Convolution?

<img width="966" height="453" alt="image" src="https://github.com/user-attachments/assets/b3f65b03-06a8-40fc-a8f5-bc19b1025c44" />

Atrous Convolution introduces an interesting idea:

> Instead of having the Kernel examine only neighboring pixels, we introduce gaps between the points that the Kernel examines.

These gaps are controlled by a parameter called:

```text
Dilation Rate
```



<img width="629" height="385" alt="image" src="https://github.com/user-attachments/assets/ef25339b-d8bf-451a-8c6e-4ebde3b61461" />

---

## Dilation Rate = 1

When:

```text
rate = 1
```

Atrous Convolution is effectively the same as standard Convolution.

```text
● ● ●
● ● ●
● ● ●
```

---

## Dilation Rate = 2

In this case, gaps are introduced between the Kernel points:

```text
● . ● . ●
. . . . .
● . ● . ●
. . . . .
● . ● . ●
```

As a result, the Kernel sees a larger region without increasing the actual number of parameters.

---

## Dilation Rate = 3

The gaps become larger and the Receptive Field increases even further.

Therefore:

```text
Dilation Rate ↑
        ↓
Receptive Field ↑
```

<img width="801" height="313" alt="image" src="https://github.com/user-attachments/assets/c948a4f3-69bf-4455-b134-da7561227110" />


---

# 9. A Very Important Point About Atrous Convolution

Contrary to what we might initially think, Atrous Convolution does not add a large number of new parameters to the Kernel.

For example, we can still have a:

```text
3 × 3
```

Kernel.

Only the way the Kernel points are positioned over the Feature Map changes.

Therefore:

> **With approximately the same number of parameters, we can achieve a larger Receptive Field.**

This is exactly what DeepLab needs.

---

# 10. Why Does DeepLab Need Atrous Convolution?

Suppose our Backbone produces a Feature Map with:

```text
Output Stride = 32
```

If the input image is:

```text
256 × 256
```

the final Feature Map will be approximately:

```text
8 × 8
```

The problem is:

> The Feature Map has become very small, and a significant amount of spatial information has been lost.

This is not ideal for segmentation.

DeepLab says:

> Instead of reducing the Resolution this much, we can remove some of the Downsampling and replace it with Atrous Convolution.

As a result, we can obtain:

```text
Output Stride = 16
```

or even:

```text
Output Stride = 8
```

The Feature Map therefore remains larger.

---

# 11. Output Stride

Output Stride describes the ratio between the spatial size of the input image and the spatial size of the output Feature Map.

For example:

```text
Input:
256 × 256

Output Stride = 16

Feature Map:
16 × 16
```

Or:

```text
Output Stride = 8

Feature Map:
32 × 32
```

The smaller the Output Stride:

* The higher the Resolution
* The more spatial information is preserved
* The higher the computational cost

Conversely, the larger the Output Stride:

* The smaller the Feature Map
* The less spatial information is preserved
* The lower the computational cost

Therefore, we have a **Trade-off**.

---

# 12. Trade-off in DeepLab

When designing a network, we cannot always maximize everything simultaneously.

We want:

```text
High Accuracy
+
More Spatial Information
+
Large Receptive Field
+
High Speed
+
Low Computational Cost
```

However, these goals are not always compatible.

Therefore, we need to find a balance.

DeepLab uses Atrous Convolution to try to achieve:

> **A large Receptive Field while maintaining relatively high Resolution.**

---

# 13. DeepLab V3 and ASPP

One of the most important components of **DeepLab V3** is a module called:

# ASPP

### Atrous Spatial Pyramid Pooling


<img width="1212" height="395" alt="image" src="https://github.com/user-attachments/assets/c281e7f1-a133-4b9a-be5e-e5cd7edcf55e" />


The idea behind ASPP is that instead of using only one type of Receptive Field, we use multiple types of Receptive Fields simultaneously.

Suppose our Feature Map is:

```text
16 × 16
```

ASPP creates several different paths over this Feature Map.

---

## First Path

A:

```text
1 × 1 Convolution
```

---

## Atrous Convolution Paths

Three Atrous Convolutions with different Rates:

```text
Rate = 6
Rate = 12
Rate = 18
```

are used.

In other words:

```text
             ┌─ 1×1 Conv ──────┐
             │                 │
Input ───────┼─ Atrous r=6 ────┤
             │                 │
             ├─ Atrous r=12 ───┤ → Concatenate
             │                 │
             ├─ Atrous r=18 ───┤
             │                 │
             └─ Image Pooling ──┘
```

Each path extracts information at a **different scale**.

---

# 14. Why Do We Use Multiple Dilation Rates?

Because objects in an image can have different sizes.

A small Rate:

> Better captures details and nearby regions.

A large Rate:

> Sees a wider region of the image.

Therefore:

```text
Rate = 6
      ↓
Smaller Field of View

Rate = 12
      ↓
Medium Field of View

Rate = 18
      ↓
Larger Field of View
```

As a result, ASPP can extract multi-scale information.

---

# 15. Image Pooling in ASPP

Another path in ASPP is **Image Pooling**.

The idea is to obtain a global view of the image.

For example, using Global Average Pooling, we can summarize the entire Feature Map into a single value.

For example:

```text
16 × 16
   ↓
Global Average Pooling
   ↓
1 × 1
```

However, this output is not suitable for direct Concatenation with the other Feature Maps.

Therefore, we use **Upsampling** to bring it back to the size of the original Feature Map:

```text
16 × 16
   ↓
Global Average Pooling
   ↓
1 × 1
   ↓
Upsampling
   ↓
16 × 16
```

This output is then also combined with the other ASPP paths.

---

# 16. Overall Structure of DeepLab V3

<img width="1222" height="467" alt="image" src="https://github.com/user-attachments/assets/5257e5a6-f24b-4a91-99f5-7751455eb3b4" />
<img width="1192" height="321" alt="image" src="https://github.com/user-attachments/assets/c1f63e35-da72-464c-8106-153778c827db" />

Therefore, we can visualize the DeepLab V3 architecture in a simplified form:

```text
Input Image
     ↓
 Backbone
     ↓
Feature Map
     ↓
   ASPP
     ↓
Multi-Scale Features
     ↓
Prediction
     ↓
Segmentation Map
```

Atrous Convolution is also used within the Backbone to create an appropriate Receptive Field without excessively reducing the Resolution.

---

# 17. Why Doesn't DeepLab V3 Directly Use a Large U-Net-Style Decoder?

One of the major differences between DeepLab and U-Net is that DeepLab tries to address the problem of spatial information **to a large extent during Feature Extraction**.

Instead of heavily relying on a large Decoder like U-Net:

```text
Encoder
   ↓
ASPP
   ↓
Prediction
```

is used.

Therefore, the main focus is on:

* Backbone
* Atrous Convolution
* ASPP
* Multi-Scale Context

---

# 18. DeepLab V3+

After DeepLab V3, the architecture:

# DeepLab V3+

was introduced.

<img width="828" height="543" alt="image" src="https://github.com/user-attachments/assets/3cbf9f43-b767-4f0e-9cf8-5ed43339d075" />

The main idea of V3+ is:

> Preserve the advantages of DeepLab V3 while adding a lightweight Decoder to better recover spatial information.

Therefore, we can think of it as a combination of:

```text
DeepLab V3
+
Lightweight Decoder
```

<img width="1024" height="664" alt="image" src="https://github.com/user-attachments/assets/7689ac34-227b-4b18-9315-33b68ec02c6e" />

---

# 19. Difference Between DeepLab V3 and V3+

In simple terms:

| Feature                              | DeepLab V3   | DeepLab V3+   |
| ------------------------------------ | ------------ | ------------- |
| Atrous Convolution                   | ✓            | ✓             |
| ASPP                                 | ✓            | ✓             |
| Multi-Scale Context                  | ✓            | ✓             |
| Decoder                              | Very limited | ✓ Lightweight |
| Use of Low-Level Spatial Information | More limited | Better        |
| Encoder-Decoder Architecture         | ❌            | ✓             |

---

# 20. Lightweight Decoder in DeepLab V3+

Unlike U-Net, V3+ does not have a heavy and fully symmetric Decoder.

A Feature Map from the Encoder is taken and upsampled.

Low-Level Feature Maps are also used to recover spatial information.

A simplified representation is:

```text
                 Low-Level Feature
                       │
                       ↓
Input → Backbone → ASPP → Upsampling
                       │
                       ↓
                  Decoder
                       │
                       ↓
              Segmentation Map
```

As a result, V3+ attempts to:

> **Combine the strong Context of DeepLab V3 with Low-Level spatial information.**

---

# 21. Backbone Changes in DeepLab V3+

One of the changes in DeepLab V3+ is the use of a modified Backbone based on **Xception**.

However, an important point is:

> The Backbone is not a fixed and unchangeable part of the architecture.

For example, we can use:

* ResNet
* Xception
* MobileNet
* EfficientNet
* Or newer Backbones

Therefore, when we see:

```text
DeepLab V3 + ResNet
```

this does not mean that DeepLab can only be used with ResNet.

---

# 22. Why Was DeepLab Successful?

If we want to summarize the main idea of DeepLab in a few sentences, it targets three important problems:

### 1. Preserving Spatial Information

By reducing the Resolution less aggressively.

### 2. Increasing the Receptive Field

Using:

```text
Atrous Convolution
```

### 3. Extracting Multi-Scale Information

Using:

```text
ASPP
```

And in V3+:

### 4. Better Recovery of Spatial Details

Using a lightweight Decoder.

---

# 23. An Important Point About Rates 6, 12, and 18

The values:

```text
6
12
18
```

are not fixed or sacred rules.

These values were selected through experiments in the original work.

Therefore, we can also experiment with:

```text
Rate = 4, 8, 12
```

or:

```text
Rate = 6, 12, 18, 24
```

or even change the number of ASPP paths.

When designing an architecture, we should always consider the balance between:

* Accuracy
* mIoU
* Parameters
* FLOPs
* Memory
* Latency

---

# 24. Importance of Output Stride

Another important Hyperparameter in DeepLab is **Output Stride**.

For example, we can investigate different settings:

```text
Output Stride = 8
Output Stride = 16
Output Stride = 32
```

Generally, as Output Stride increases, the Feature Map becomes smaller and less spatial information is preserved.

On the other hand, a smaller Output Stride increases the computational cost.

Therefore, its selection depends on the task and available computational resources.

---

# 25. Conceptual Comparison Between U-Net and DeepLab

### U-Net

Main focus:

> Combining spatial and semantic information using Skip Connections.

```text
Encoder
   ↓
Bridge
   ↓
Decoder
   ↑
Skip Connections
```

### DeepLab V3

Main focus:

> Extracting large Context and Multi-Scale information without excessively reducing the Resolution.

```text
Backbone
   ↓
Atrous Convolution
   ↓
ASPP
   ↓
Prediction
```

### DeepLab V3+

Main focus:

> Combining the powerful Context of DeepLab V3 with a lightweight Decoder.

```text
Backbone
   ↓
ASPP
   ↓
Lightweight Decoder
   ↑
Low-Level Features
```

---

# 26. The Main Idea of DeepLab at a Glance

The entire idea can be summarized as follows:

```text
                    DeepLab
                       │
        ┌──────────────┴──────────────┐
        ↓                             ↓
Preserving Resolution          Increasing Receptive Field
        │                             │
        ↓                             ↓
Less Downsampling              Atrous / Dilated Conv
        │                             │
        └──────────────┬──────────────┘
                       ↓
                    ASPP
                       ↓
              Multi-Scale Context
                       ↓
                Segmentation
```

And in **DeepLab V3+**:

```text
DeepLab V3
    +
Lightweight Decoder
    +
Low-Level Features
    ↓
Better Spatial Detail
```

---

# 27. An Important Point for Architecture Design

One of the most important lessons we can learn from studying U-Net and DeepLab is:

> Deep Learning architectures do not always have to be used exactly as they were presented in the original paper.

We can modify different components and experiment with them.

For example:

* Change the Backbone.
* Change the number of Atrous Convolutions.
* Change the Dilation Rates.
* Change the Output Stride.
* Make the Decoder lighter.
* Add Attention mechanisms.
* Use newer Backbones.

However, we should always evaluate the results using appropriate metrics.

---

# 28. Summary

The evolution of the main ideas can be viewed as follows:

```text
Autoencoder
     │
     │ Problem:
     │ Loss of Spatial Information
     ↓
U-Net
     │
     │ Solution:
     │ Skip Connections
     ↓
DeepLab
     │
     │ Solution:
     │ Atrous Convolution
     ↓
DeepLab V3
     │
     │ Solution:
     │ ASPP + Multi-Scale Context
     ↓
DeepLab V3+
     │
     │ Solution:
     │ Lightweight Decoder
     │ + Low-Level Features
     ↓
Better Semantic Segmentation
```

### Key Concepts in This Section

```text
Autoencoder
      ↓
U-Net
      ↓
Skip Connection
      ↓
Receptive Field
      ↓
Atrous / Dilated Convolution
      ↓
Dilation Rate
      ↓
Output Stride
      ↓
ASPP
      ↓
DeepLab V3
      ↓
DeepLab V3+
```

**The key idea behind DeepLab** is that for Segmentation, having only deep Features is not enough; we need both **wide Context** and **appropriate spatial information**. Atrous Convolution and ASPP are specifically designed to achieve this balance.
