# Vision Transformer (ViT) — Introduction to Transformer and Self-Attention

## Introduction

**Vision Transformer (ViT)** is one of the important architectures in the field of **Computer Vision**, which transfers the core idea of the **Transformer** architecture from natural language processing to image processing.

As its name suggests, Vision Transformer aims to use the **Attention mechanism** to understand the relationships between different parts of an image.

However, to properly understand Vision Transformer, we first need to answer some fundamental questions:

* What is a Transformer?
* Why was the Transformer introduced?
* What problem does Self-Attention solve?
* Why does Attention have advantages over recurrent architectures such as RNNs?
* What is Positional Encoding and why is it required?
* What does Multi-Head Attention do?

The foundation of the Transformer architecture was introduced in the famous paper:

> **Attention Is All You Need**

This paper was published in **2017** and introduced the Transformer architecture for processing sequential data, especially text data.

---

# 1. Text Data and the Concept of Sequence

Unlike many other types of data, text data has an inherent structure called a **Sequence**.

Consider the following sentence:

```text
I went to Iran
```

If we consider each word as a Token, we have:

```text
[I] → [went] → [to] → [Iran]
```

The order of these words is extremely important.

For example:

```text
Iran is in Tehran
```

and:

```text
Tehran is in Iran
```

have completely different meanings.

Therefore, in text data, knowing only which words exist in a sentence is not enough; the **position and relationships between words are also important.**

---

# 2. Before Transformer: Recurrent Neural Networks (RNN)

Before the emergence of Transformer, one of the main approaches for processing sequential data was:

**Recurrent Neural Networks (RNNs)**

RNNs process data step by step.

For example:

```text
I → went → to → Iran
```

The model first receives the word `I`, then `went`, then `to`, and finally `Iran`.

Conceptually:

```text
Token 1       Token 2       Token 3       Token 4
   I     →     went    →      to     →      Iran
   ↓             ↓             ↓             ↓
 State 1     → State 2     → State 3    → State 4
```

Each State keeps information from previous steps.

---

# 3. Using RNN to Understand the Entire Sentence

If our goal is to understand the meaning of an entire sentence, we can use the final hidden state as a representation of the whole sentence.

For example:

```text
I went to Iran
        ↓
 Sentence Vector
```

This vector can be used for tasks such as:

* Classification
* Sentiment Analysis
* Text Classification

In other tasks such as **Named Entity Recognition (NER)**, we may need an output representation for each individual Token.

Therefore, an RNN can produce an output at every step:

```text
I      → Output 1
went   → Output 2
to     → Output 3
Iran   → Output 4
```

---

# 4. First Problem of RNN: Long-Term Dependencies

One of the major challenges of recurrent networks is processing very long sequences.

Assume we have a sentence containing a large number of Tokens:

```text
Token 1 → Token 2 → Token 3 → ... → Token 1000
```

Information from early Tokens must be transferred through many intermediate states.

Therefore:

```text
Token 1
   ↓
State 1
   ↓
State 2
   ↓
State 3
   ↓
...
   ↓
State 1000
```

As the sequence becomes longer, preserving information from the initial Tokens becomes more difficult.

As a result, important information from the beginning of the sequence may gradually disappear during processing.

---

# 5. Importance of Attention

Consider the following sentence:

```text
I went to Iran
```

If we ask:

```text
Where did I go?
```

the most important word for answering this question is:

**Iran**

However, if the question is:

```text
What did I do?
```

the more important word is:

**went**

Therefore, the importance of each word is not fixed and depends on the **Context** and the specific Task.

This is where the concept of **Attention** becomes important.

---
# 6. The Main Idea of Attention

The idea behind **Attention** is that instead of forcing the model to transfer information only step by step, it allows the model to **directly examine the relationships between different Tokens.**

For example:

```text
I      went      to      Iran
 ↘       ↓        ↓       ↙
      Relationships between Tokens
```

The model analyzes how each Token is related to other Tokens.

As a result, different weights can be assigned to the relationships between Tokens.

Conceptually:

```text
I  ───────── Iran
│              ↑
│          Attention
↓              │
went ──────────┘
```

The more important a Token relationship is for a specific Task, the higher Attention weight it receives.

---

# 7. Attention Weight

We can define weights for the relationships between Tokens.

For example:

```text
I → Iran       0.2
went → Iran    0.5
Iran → went    0.8
```

These values indicate how much connection or attention the model assigns to different parts of the input in a specific situation.

However, in the real implementation of Transformer, Attention weights are not manually assigned like this. They are obtained through computations involving:

* Query
* Key
* Value

The main idea is:

> The model learns which parts of the input are more important for understanding each other part.

---

# 8. A Major Advantage of Attention: Parallel Processing

One of the major limitations of RNNs is that Tokens must be processed sequentially:

```text
Token 1
   ↓
Token 2
   ↓
Token 3
   ↓
Token 4
```

However, Attention can process all Tokens simultaneously:

```text
Token 1 ─┐
Token 2 ─┤
Token 3 ─┼──→ Attention
Token 4 ─┤
Token N ─┘
```

This capability allows Transformer models to take better advantage of **Parallel Processing** on hardware such as GPUs.

Unlike RNNs, Transformer does not need to process Tokens one by one to understand their relationships.

---

# 9. A Remaining Problem: Token Order

In Attention, the model can observe all Tokens simultaneously.

However, an important question remains:

If the order of Tokens is ignored, how does the model understand the difference between:

```text
I went to Iran
```

and:

```text
Iran went to I
```

?

This is where the concepts of:

# Positional Encoding

and

# Positional Embedding

are introduced.

---

# 10. What is Positional Encoding?

To allow the model to understand the position of each Token, positional information is added to the Token representation.

Simply:

```text
Token Embedding
       +
Positional Information
       ↓
Transformer Input
```

For example:

```text
I       → Position 0
went    → Position 1
to      → Position 2
Iran    → Position 3
```

Therefore, the model can have two types of information simultaneously:

1. **What is this Token?**
2. **Where is this Token located in the sequence?**

Different methods exist for Positional Encoding, including:

* Sinusoidal and cosine-based functions
* Learnable positional embeddings

---

# 11. Self-Attention

One of the most important concepts in Transformer is:

**Self-Attention**

In Self-Attention, each Token interacts with all other Tokens in the same Sequence.

For example:

```text
I   went   to   Iran
```

The model can examine:

```text
I      ↔ went
I      ↔ to
I      ↔ Iran

went   ↔ to
went   ↔ Iran

to     ↔ Iran
```

As a result, the model can directly understand relationships within the entire Sequence.

---

# 12. Query, Key, and Value

In the implementation of Self-Attention, three different representations are created for each Token:

* **Query (Q)**
* **Key (K)**
* **Value (V)**

Conceptually:

```text
Input Embeddings
       ↓
 ┌─────┼─────┐
 ↓     ↓     ↓
 Q     K     V
```

Then, the similarity between Query and Key is used to calculate Attention.

The famous Scaled Dot-Product Attention formula is:

```text
Attention(Q, K, V)
=
softmax(QKᵀ / √dₖ)V
```

where:

* `Q` = Query
* `K` = Key
* `V` = Value
* `dₖ` = Key dimension

The model first calculates the relationship between Tokens and then combines Value representations based on the Attention weights.

---

# 13. Why Multi-Head Attention?

A single Attention mechanism may not be enough to represent all types of relationships within the data.

Different relationships may represent:

* Semantic relationships
* Syntactic relationships
* Connections between subject and verb
* Long-range dependencies between distant words

Therefore, Transformer uses multiple Attention mechanisms.

This structure is called:

# Multi-Head Attention

Conceptually:

```text
                Input
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Head 1     Head 2      Head 3
       │          │          │
       ↓          ↓          ↓
   Attention   Attention   Attention
       │          │          │
       └──────────┼──────────┘
                  ↓
             Concatenate
                  ↓
             Linear Layer
```

Each Head can learn a different type of relationship.

Instead of learning only one attention pattern, the model creates multiple perspectives for understanding relationships in the data.

---

# 14. Transformer Architecture

The original Transformer architecture introduced in the paper:

**Attention Is All You Need**

was based on:

```text
Encoder + Decoder
```

A simplified view:

```text
Input
  ↓
Encoder
  ↓
Context Representation
  ↓
Decoder
  ↓
Output
```

The Encoder extracts representations from the input, while the Decoder uses these representations to generate the output.

However, modern Transformer-based models do not necessarily contain both Encoder and Decoder.

Generally, Transformer models can be divided into three main categories:

---

## Encoder-Only

Examples:

```text
BERT
```

These models are mainly used for:

* Understanding data
* Representation learning

---

## Decoder-Only

Examples:

Large language models.

These models are mainly designed for:

* Sequential data generation

---

## Encoder-Decoder

Used for sequence-to-sequence tasks such as:

* Machine Translation

---

# 15. Transformer vs RNN

| Feature                    | RNN            | Transformer                      |
| -------------------------- | -------------- | -------------------------------- |
| Sequential Processing      | Yes            | No, supports parallel processing |
| Parallel Processing        | Limited        | Highly efficient                 |
| Long-range Dependencies    | More difficult | More effective                   |
| Direct Token Relationships | Limited        | Yes, through Attention           |
| GPU Utilization            | More limited   | Highly optimized                 |
| Main Mechanism             | Recurrence     | Attention                        |

One of the most important changes introduced by Transformer was:

> Instead of transferring information only through a sequential chain, Transformer directly models relationships between different parts of the input using Attention.

---

# 16. From Transformer to Vision Transformer

Until now, we have studied Transformer in the context of text data.

But the main question is:

> Can the same idea be applied to images?

The answer is **yes**.

The main idea behind **Vision Transformer (ViT)** is exactly based on this concept.

In images, instead of working with words, we divide the image into smaller regions called:

**Patches**

For example:

```text id="m3n8qf"
Image
  ↓
Patch 1 | Patch 2 | Patch 3
Patch 4 | Patch 5 | Patch 6
Patch 7 | Patch 8 | Patch 9
```

Each Patch can be considered similar to a Token in Natural Language Processing.

Then the process becomes:

```text id="f0j7bx"
Image
  ↓
Image Patches
  ↓
Patch Embeddings
  ↓
Positional Embeddings
  ↓
Transformer Encoder
  ↓
Attention
  ↓
Image Representation
  ↓
Prediction
```

Therefore, an important relationship is established:

| NLP                  | Vision               |
| -------------------- | -------------------- |
| Word Token           | Image Patch          |
| Word Embedding       | Patch Embedding      |
| Position Information | Patch Position       |
| Self-Attention       | Patch Self-Attention |
| Transformer          | Vision Transformer   |

---

# 17. The Main Idea of Vision Transformer

Vision Transformer does not process an image only through convolution operations. Instead, it converts the image into a set of Patches and uses Transformer to learn the relationships between these Patches.

For example:

```text id="0v4w2k"
Image
  │
  ├── Patch 1
  ├── Patch 2
  ├── Patch 3
  ├── ...
  └── Patch N
          ↓
    Patch Embedding
          ↓
    Positional Embedding
          ↓
    Transformer Encoder
          ↓
    Multi-Head Self-Attention
          ↓
    Image Representation
```

As a result, the model can understand relationships between different regions of an image, even when those regions are spatially far apart.

---

# 18. Summary

The evolution of this idea can be summarized as follows:

```text id="k3i5zn"
Sequential Data
      ↓
      RNN
      ↓
Problems with:
- Sequential Processing
- Long-Range Dependencies
- Information Loss
      ↓
   Attention
      ↓
 Self-Attention
      ↓
Multi-Head Attention
      ↓
   Transformer
      ↓
Transformer for Images
      ↓
Vision Transformer (ViT)
```

The most important concepts to remember before entering Vision Transformer are:

---

## 1. Attention

The model learns to focus on different parts of the input according to their importance.

---

## 2. Self-Attention

Each Token can examine its relationship with all other Tokens in the same input.

---

## 3. Multi-Head Attention

Multiple Attention Heads allow the model to learn different types of relationships between parts of the input.

---

## 4. Positional Encoding

Since Attention does not naturally understand order, positional information is added to the input representation.

---

## 5. Parallel Processing

Unlike RNNs, Transformer does not depend on sequential Token processing and can efficiently utilize parallel processing on GPUs.

---

## 6. Vision Transformer

In ViT, an image is divided into small Patches, and each Patch is treated as a Token that is processed by the Transformer.

---

# Main References

* **Attention Is All You Need — Vaswani et al., 2017**
* **BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding — Devlin et al., 2018**
* **An Image is Worth 16x16 Words: Transformers for Image Recognition at Scale — Dosovitskiy et al., 2020**

Using these concepts, we can move toward the architecture of **Vision Transformer (ViT)** and study in detail:

* How an image is converted into Patches
* How Patch Embedding is performed
* How Positional Embedding is added
* How Self-Attention learns relationships between different regions of an image
