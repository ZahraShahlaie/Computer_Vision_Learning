


# Vision Transformer (ViT) — Understanding the Architecture and Core Idea

## Introduction

In the previous session, we studied the concept of **Transformer**, the **Attention mechanism**, and especially **Self-Attention**. To understand the architecture of **Vision Transformer (ViT)**, we first need to understand how this model adopts ideas from language models such as **BERT** and transfers them into the field of Computer Vision.

The original Vision Transformer paper:

> **An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale**

showed that images can be processed and classified using only the Transformer architecture without relying on Convolutional Neural Networks (**CNNs**).

The main idea behind ViT is:

> Similar to how words are treated as Tokens in NLP, small regions of an image called Patches are treated as Tokens in Vision Transformer.

---

# 1. Relationship Between Vision Transformer and BERT

One of the important ideas behind Vision Transformer is that its architecture is inspired by **BERT**.

BERT is an:
```

Encoder-Only Transformer

```

architecture.

Unlike the original Transformer model, which consists of:
```

Encoder + Decoder

```

BERT only uses the Encoder part.

The reason is that BERT is designed for **understanding text**, not generating text.

Examples of tasks that require understanding:

- Sentiment Analysis
- Text Classification
- Information Extraction

The general structure of BERT:
```

Input Tokens  
|  
Token Embedding  
|  
Transformer Encoder  
|  
Sentence Representation  
|  
Classification

```

Vision Transformer follows a similar idea for images.

---

# 2. Review of BERT and Self-Attention

In BERT, text is first divided into different Tokens.

For example:
```

I went to Iran

```

is converted into:
```

Token 1 → I  
Token 2 → went  
Token 3 → to  
Token 4 → Iran

```

These Tokens are then passed into the Transformer Encoder.

But how does the model learn relationships between words?

Through:

## Self-Attention

In Self-Attention, every word can interact with other words in the same sentence.

For example:
```

I → Iran  
Iran → went  
to → Iran

```

The model learns which words have stronger relationships with each other.

Each relationship receives a learned weight.

Example:
```

I ---- Iran  
0.8

```

This means these two words have a strong semantic relationship.

---

# 3. How is BERT Trained?

One of the main training approaches of BERT is predicting masked words.

Assume we have the sentence:
```

I went to [MASK]

```

The model must predict the missing word.

During training, BERT:

- Receives a sequence of Tokens.
- Randomly hides some Tokens.
- Attempts to predict the missing Tokens.

This method is called:
```

Masked Language Modeling

```

Through this process, the model learns relationships between words.

---

# 4. CLS Token in BERT

BERT adds a special Token at the beginning of every sentence:
```

[CLS]

```

CLS stands for:
```

Classification

```

The input structure becomes:
```

[CLS] I went to Iran

```

The purpose of CLS is to represent the entire sentence.

Why?

Because the output of each Token mainly represents information about that specific word.

However, tasks such as:

- Sentiment Analysis
- Text Classification

require a global representation of the entire sentence.

Therefore:
```

Sentence  
|  
Transformer  
|  
CLS Vector  
|  
Classification

```

The output CLS vector becomes a summary representation of the whole sentence.

---

# 5. Main Idea of Vision Transformer

Now we apply the same idea to images.

In NLP:
```

Word  
↓  
Token  
↓  
Embedding  
↓  
Transformer

```

In Vision Transformer:
```

Image  
↓  
Patch  
↓  
Patch Embedding  
↓  
Transformer

```

Instead of words, images are divided into small regions called Patches that behave like Tokens.

---

# 6. First Step in ViT: Splitting Image into Patches

The first step in Vision Transformer is:
```

Image → Patch

```

Suppose we have an image:
```

+----+----+----+  
| P1 | P2 | P3 |  
+----+----+----+  
| P4 | P5 | P6 |  
+----+----+----+  
| P7 | P8 | P9 |  
+----+----+----+

```

The image is divided into smaller regions.

Each region is called a Patch.

In the original ViT paper, patches of:
```

16 × 16

```

pixels are commonly used.

This means every Patch contains a small 16×16 region of the image.

---

# 7. Converting Patch into Vector (Patch Embedding)

Raw image patches cannot directly enter the Transformer.

Similar to NLP, where words are converted into vectors before entering the model, image patches must also be transformed.

In NLP:
```

Word  
↓  
Embedding Layer  
↓  
Vector

```

In Vision Transformer:
```

Image Patch  
↓  
MLP  
↓  
Patch Vector

```

This transformation is usually performed using:
```

MLP (Multi Layer Perceptron)

```

or a Fully Connected Layer.

The goal is:

To convert every Patch into a vector representation.

Example:
```

Patch 1  
|  
↓  
[0.25, 0.71, 0.12,...]

```

This vector represents the visual information of that image region.

---

# 8. Feeding Patches into Transformer

After Patch Embedding, the image becomes a sequence of vectors.

Example:
```

Patch 1 → Vector  
Patch 2 → Vector  
Patch 3 → Vector  
...  
Patch N → Vector

```

These vectors are treated like text Tokens and passed into the Transformer Encoder.

At this stage, Self-Attention learns relationships between image patches.

For example:
```

Patch 1 ↔ Patch 10  
Patch 5 ↔ Patch 20

```

Even if these patches are spatially far apart.

---

# 9. Difference Between ViT and CNN

One of the most important advantages of Vision Transformer compared to CNN is its ability to understand global relationships.

In CNN:

Convolution operations are performed on local regions of the image.

The model initially sees only local information.

Example:
```

Small Image Region  
↓  
Convolution

```

To understand relationships between distant regions, CNN requires:

- More layers
- Deeper architectures

However, in ViT:

All patches are available to the Attention mechanism at the same time.

Therefore, the model can directly analyze:
```

Patch 1  
↔  
Patch 100

```

As a result:

- Better Global Representation
- Understanding relationships between distant image regions
- Less dependency on local structures

---

# 10. Patch Ordering Problem and Positional Embedding

Self-Attention does not naturally understand order or position.

For Transformer, these sequences are identical:
```

Patch 1  
Patch 2  
Patch 3

```

and:
```

Patch 3  
Patch 1  
Patch 2

```

Therefore, positional information must be added.

The solution is:

# Positional Embedding

Each Patch receives both:

- Feature information
- Position information

Conceptually:
```

Patch Embedding  
+  
Position Embedding  
|  
↓  
Transformer Input

```

This allows the model to understand:

- Where each Patch is located.
- How different regions are spatially related.

---

# 11. CLS Token in Vision Transformer

Similar to BERT, ViT also introduces a special Token.

The input structure:
```

[CLS]  
Patch 1  
Patch 2  
Patch 3  
...  
Patch N

```

This Token does not represent a specific image region.

Its purpose is:

To represent the entire image.

At the end:
```

CLS Output  
|  
↓  
Softmax  
|  
Class Prediction

```

Example:
```

Image → Transformer → CLS Vector → Dog

```

---

# 12. Overall Vision Transformer Architecture

The complete ViT pipeline:
```

Input Image

```
  ↓
```

Image Patching

```
  ↓
```

Patch Embedding

```
  ↓
```

Add Positional Embedding

```
  ↓
```

Add CLS Token

```
  ↓
```

Transformer Encoder

```
  ↓
```

CLS Representation

```
  ↓
```

MLP Head

```
  ↓
```

Classification

```

---

# 13. Summary

The main idea of Vision Transformer can be summarized as follows:

1. Divide the image into small patches.
2. Treat each patch as a Token.
3. Convert patches into vector representations.
4. Add positional information.
5. Feed patches into the Transformer Encoder.
6. Use Self-Attention to learn relationships between image regions.
7. Use CLS Token to create a global image representation.
8. Use the final representation for classification.

In short:
```

Image  
↓  
Patches  
↓  
Patch Embedding  
↓  
Positional Embedding  
↓  
Transformer Encoder  
↓  
Attention  
↓  
Image Representation  
↓  
Classification

```

Vision Transformer demonstrated that CNNs are not the only solution for image understanding. By using Attention mechanisms and Transformer architectures, powerful visual representations can also be learned for Computer Vision tasks.
