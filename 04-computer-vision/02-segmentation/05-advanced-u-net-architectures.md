# A Review of Extended U-Net Architectures

## Introduction

When an architecture performs well on a given problem, researchers usually try to preserve its strengths while addressing its weaknesses through various modifications.

U-Net has followed exactly this path. Since its introduction, a huge number of architectures have been proposed based on it. Each has tried to improve a specific aspect of U-Net, for example:

- Improving gradient flow
- Better information transfer between the encoder and decoder
- Reducing the number of parameters
- Reducing computational cost
- Increasing processing speed
- Better feature extraction
- Improving segmentation accuracy

In this piece, we'll look at three important architectures:

- Residual U-Net / ResU-Net
- U-Net++
- U-Net 3+

The goal isn't just to memorize these architectures — it's to understand what idea was added to U-Net, why it was proposed, and whether it actually improved the model.

---

## 1. U-Net: The Base Architecture

Before looking at the extended architectures, let's review U-Net's core structure.

U-Net consists of two main paths:

```
Encoder
   ↓
Downsampling
   ↓
Bottleneck
   ↓
Upsampling
   ↓
Decoder
```

In the encoder, as the image's spatial dimensions shrink, higher-level features are extracted. In the decoder, the spatial dimensions are expanded back so we can make a prediction for every pixel.

One of U-Net's most important features is the skip connections between the encoder and decoder. This connection is typically done via **concatenation**:

```
Encoder Feature
      │
      │
      └──────────────┐
                     ↓
               Concatenate
                     ↑
                     │
Decoder Feature ─────┘
```

So spatial information that may have been lost during downsampling is passed directly to the decoder.

---

## 2. Residual U-Net
<img width="438" height="791" alt="image" src="https://github.com/user-attachments/assets/17a95221-56ed-480f-b2ae-381fb65d6710" />

One important idea for extending U-Net is using **residual blocks**. This is why, when we see names like:

- Residual U-Net
- ResU-Net

we should immediately think of residual blocks.

### 2.1 What Is a Residual Block?

In a residual block, the input is passed directly to the output. Meanwhile, the main path performs a few convolution operations. For example:

```
              ┌──────────────────────┐
              │                      │
Input ────────┼──→ Conv → BN → ReLU ─→ Conv ──┐
              │                                │
              └────────────────────────────────┘
                                               ↓
                                            Addition
                                               ↓
                                             Output
```

In other words:

```
Output = F(x) + x
```

where:

- `x` is the input.
- `F(x)` is the output of the network's main path.
- These two are finally combined through addition.

---

## 3. Why Add Residual Blocks to U-Net?

One of the challenges in deep networks is getting gradients to flow properly through the network. The deeper the network gets, the harder training can become.

A residual connection creates a direct path for information and gradients to travel through, which lets them flow more easily through the network. As a result, one of the main goals of Residual U-Net is to:

> Improve the flow of information and gradients through the network.

---

## 4. Combining Residual Connections and Skip Connections in U-Net

An important point: the residual connection doesn't replace U-Net's original skip connection — the two can be used together.

In other words, a Residual U-Net might have, at the same time:

```
Residual Connection
        ↓
    Addition
```

and:

```
Encoder ─────────→ Decoder
          Concatenation
```

So the architecture contains two distinct concepts:

- Inside the residual block → **addition**
- Between the encoder and decoder → **concatenation**

This combination lets us benefit from residual learning while also keeping U-Net's core advantage: transferring encoder features to the decoder.

---

## 5. Another Key Change: Upsampling Instead of Transposed Convolution

Another interesting change in some optimized U-Net variants is replacing **transposed convolution** with plain **upsampling**.

In standard U-Net, transposed convolution can be used to increase a feature map's dimensions. Transposed convolution has trainable parameters, so:

```
Transposed Convolution
        ↓
Trainable Weights
        ↓
More Parameters
        ↓
Heavier Model
```

But simple upsampling can increase a feature map's dimensions without adding any trainable weights.

---

## 6. How Does Simple Upsampling Work?

One simple upsampling method is **nearest-neighbor**. Suppose we have an initial feature map:

```
A  B
C  D
```

If we want to double its size, each value is repeated across a larger region:

```
A  A  B  B
A  A  B  B
C  C  D  D
C  C  D  D
```

So a:

```
2 × 2
```

becomes a:

```
4 × 4
```

without introducing any new trainable parameters.

---

## 7. Why Isn't Simple Upsampling Enough on Its Own?

Simple upsampling only increases the feature map's size — it doesn't learn any new features on its own. That's why upsampling is usually followed by a convolution:

```
Feature Map
     ↓
Upsampling
     ↓
Larger Feature Map
     ↓
Convolution
     ↓
Feature Refinement
```

Here, upsampling handles increasing the resolution, while the convolution handles learning and extracting new features. Transposed convolution, in contrast, has its own trainable parameters built in.

---

## 8. The Purpose of Replacing Transposed Convolution

The main goal of this change is to make the network lighter.

```
Transposed Convolution
        ↓
More Parameters
        ↓
Heavier Model
```

versus:

```
Upsampling
     ↓
No Trainable Weights
     ↓
Convolution
     ↓
Fewer Parameters
```

So when designing new architectures, accuracy isn't the only thing that matters. Today, metrics like:

- Number of parameters
- Computational cost
- Memory usage
- Inference speed
- FPS
- Model size

matter a great deal too.

---

## 9. The Outcome of Residual U-Net

An important habit when evaluating a new architecture: don't just say "this architecture has a new idea." Instead, check whether the idea actually produced better results.

In the reported results for the version under review, model performance went from roughly:

```
90.51
```

to roughly:

```
91.87
```

an improvement of about:

```
+1.36
```

But the more important point is that this improvement came alongside a notable reduction in the number of parameters. In the reported comparison, the parameter count went from roughly:

```
363M
```

down to roughly:

```
~768M
```

*(Note: architectural details and training settings must be interpreted exactly as reported in the original paper — parameter counts shouldn't be compared as raw numbers separate from those settings.)*

The key takeaway from this example: a change to an architecture is only genuinely valuable when it considers computational cost and model complexity, not just performance.

---

## 10. U-Net++

The next well-known architecture is **U-Net++**.


<img width="904" height="516" alt="image" src="https://github.com/user-attachments/assets/595be5b4-0aa6-4eed-a790-1548692fd8fc" />


Its "nested U-Net" name refers to the way new connections are created in this architecture.

---

## 11. The Core Idea of U-Net++

In standard U-Net, skip connections are fairly direct:

```
Encoder ─────────────────→ Decoder
```

But U-Net++ asks:

> Why limit the connection between the encoder and decoder to a single direct path?

As a result, it creates a number of additional intermediate paths, adding more connections between different parts of the network.

---

## 12. Nested Skip Connections

U-Net++'s defining feature is exactly these **nested skip connections**. In addition to U-Net's original skip connections, additional paths are created between layers.

These extra paths let the encoder's and decoder's feature maps connect to each other across different distances and different levels.

---

## 13. Why Does U-Net++ Add These Connections?

One of the main goals here is to reduce the **semantic gap** between encoder and decoder features.

In standard U-Net, the encoder's feature and the corresponding decoder's feature can differ significantly in their level of processing. U-Net++ tries to reduce this gap by creating intermediate paths. As a result:

```
Encoder Feature
       ↓
Intermediate Processing
       ↓
More Refined Feature
       ↓
Decoder
```

Instead of being passed directly, a feature can be processed and refined along intermediate paths first.

---

## 14. The Cost of U-Net++

Of course, adding more connections isn't free. When we have:

- More convolutions
- More feature maps being produced
- More connecting paths

the architecture naturally becomes heavier. So an important question arises:

> Does the added complexity actually translate to better performance?

---

## 15. U-Net++ and the "Wide U-Net++" Concept

In some experiments, the number of channels and kernels is also changed. This matters, because a network's parameter count doesn't depend only on the shape of its connections. For example:

```
More Channels
      ↓
More Parameters
      ↓
Heavier Model
```

So if a model performs better simply because it has more parameters, we can't conclude that the new architecture is genuinely more efficient. That's why, when comparing architectures, we need to compare models under as similar conditions as possible.

---

## 16. U-Net 3+

The next important architecture is **U-Net 3+**.

Its full title is:

> Full-Scale Connected U-Net

<img width="1199" height="397" alt="image" src="https://github.com/user-attachments/assets/ef527ff3-5894-4007-aa3c-d228e9cb7bad" />

which itself conveys the architecture's core idea: **full-scale connections**.

---

## 17. The Core Idea of U-Net 3+

In standard U-Net, skip connections mainly link the encoder and decoder at corresponding levels. In U-Net++, intermediate, nested connections were added.

U-Net 3+ takes this idea one step further: in this architecture, features from multiple different scales can all be passed to the decoder at once. Conceptually:

```
Encoder Level 1 ────────┐
Encoder Level 2 ────────┤
Encoder Level 3 ────────┼──→ Decoder
Encoder Level 4 ────────┤
                         │
Decoder Features ────────┘
```

So the decoder isn't dependent on just one feature map from a single level.

---

## 18. Full-Scale Feature Fusion

U-Net 3+'s key idea is to combine information across multiple scales. In segmentation, this matters a great deal, because:

- Low-level features carry more precise spatial information.
- High-level features carry richer semantic information.

So if we combine information from these different scales, the decoder can use both:

```
Spatial Information
+
Semantic Information
```

at the same time.

---

## 19. Comparing the Three Architectures

Putting the core idea of each architecture side by side:

| Architecture | Core Idea |
| --- | --- |
| U-Net | Skip connection + concatenation |
| Residual U-Net | Residual block + addition |
| U-Net++ | Nested skip connections |
| U-Net 3+ | Full-scale feature connections |

---

## 20. A Note on Comparing Architectures

Different papers may use different evaluation metrics, for example:

- IoU
- Dice
- Precision
- Recall
- F1
- PR curve

So we shouldn't directly compare numbers from different papers without accounting for:

- Dataset
- Train/test split
- Preprocessing
- Loss function
- Image resolution
- Number of channels
- Number of parameters
- Training strategy

---

## 21. The Precision-Recall Curve

One evaluation method used in some experiments is the **precision-recall curve**. In general, the better the PR curve, and the larger the area under it, the better the model's performance. So if:

```
AUC(PR) ↑
```

it usually indicates better model performance. Still, this kind of comparison is only valid when the experimental conditions are similar.

---

## 22. Is a Newer Architecture Always Better?

No — and this is one of the most important lessons from reviewing different architectures.

An architecture might have:

```
Accuracy ↑
Parameters ↑↑
Computation ↑↑
Speed ↓
```

In this case, the model became more accurate, but not necessarily more efficient. The ideal outcome, in contrast, looks like:

```
Accuracy ↑
Parameters ↓
Computation ↓
Inference Speed ↑
```

This is what we can call true **model optimization**.

---

## 23. What Does "Optimized" Actually Mean?

When we say an architecture "has become more optimized," we don't just mean higher accuracy. Optimization can include things like:

```
Parameters ↓
FLOPs ↓
Memory ↓
Latency ↓
Model Size ↓
Inference Time ↓
```

while:

```
IoU ↑
Dice ↑
F1 ↑
```

stay the same or even improve.

So in real-world projects, we're not always chasing the highest possible accuracy — we're looking for the best trade-off between accuracy and cost.

---

## 24. Why Does Studying These Architectures Matter?

Reviewing these architectures isn't just about recognizing a few well-known models — it's a way of learning how to design neural network architectures in general.

When you come across a paper titled something like:

> U-Net + Something

you should ask yourself:

1. What problem existed in U-Net?
2. Exactly which part did the new idea change?
3. What effect does this change have on the network?
4. Did the number of parameters increase or decrease?
5. Did accuracy improve?
6. Did the computational cost change?
7. Was this change actually worthwhile?

This kind of thinking moves you from being a "model user" toward becoming an "architecture analyst."

---

## 25. Idea: Adding an Intermediate Layer

We could add a new layer or block between two parts of the network, for example:

```
Encoder
   ↓
New Block
   ↓
Decoder
```

Then we'd check whether this new block actually improved feature representation.

---

## 26. Idea: Using a Different Convolution Type

We could also check whether using a different type of convolution might help, for example:

```
Standard Convolution
        ↓
Alternative Convolution
```

The point isn't simply to add a new layer — we should have a hypothesis first, such as:

> "I think this change will let the network extract better features at a lower cost."

Then we'd test that hypothesis through experimentation.

---

## Summary

U-Net is an extremely important architecture in semantic segmentation, which is exactly why so many architectures have been built on top of it. In this piece, we covered three important directions.

### Residual U-Net

Core idea:

```
Residual Block
+
Addition
```

Goal:

- Improve gradient flow
- Enable more direct information transfer
- Increase the network's learning capacity

### U-Net++

Core idea:

```
Nested Skip Connections
```

Goal:

- Create more connections between the encoder and decoder
- Reduce the semantic gap between features
- Make better use of intermediate features

### U-Net 3+

Core idea:

```
Full-Scale Feature Connections
```

Goal:

- Use multi-scale features simultaneously
- Combine spatial and semantic information
- Create broader connectivity between the encoder and decoder

### Final Takeaway

New architectures rarely appear out of nowhere — they're usually built by identifying a weakness in a previous architecture and proposing an idea to address it. U-Net has followed exactly this path.

So studying architectures like Residual U-Net, U-Net++, and U-Net 3+ isn't just about learning a handful of models — it's practice in learning how to analyze, modify, test, and optimize an architecture.
