
# CLAHE (Contrast Limited Adaptive Histogram Equalization)

## Introduction

With **Histogram Stretching** and **Histogram Equalization**, we saw that although they perform very well on many images, they aren't always the best choice. The most significant problem with both methods is that they process the image **globally** — every pixel is changed purely based on the histogram of the entire image, with no regard for its position or its relationship with neighboring pixels.

This leads to problems such as:

- Excessive contrast enhancement
- Noise amplification
- Loss of the image's natural appearance
- Creation of overly bright or overly dark regions
- Disrupted colors in color images

To address these problems, we use a method called **CLAHE**, one of the most widely used contrast enhancement algorithms in image processing.

---

# What Is CLAHE?

CLAHE stands for:

**Contrast Limited Adaptive Histogram Equalization**

If we break this phrase down into its parts, each part carries a specific meaning:

- **Adaptive Histogram Equalization (AHE)**
- **Contrast Limited (CL)**

Each of these two parts solves one of the problems found in the previous methods.

---

# What Is Adaptive Histogram Equalization?

To understand this part, we first need to know how it differs from regular histogram equalization.

In regular histogram equalization, the entire image has just a single histogram.

The algorithm only computes how many times each intensity value occurs, without any regard for where those pixels are located.

For example, if intensity value 50 occurs a thousand times in the image, that's the only thing that matters to the algorithm.

Where in the image those thousand pixels are located doesn't matter at all.

---

# Pixel Location Doesn't Matter in a Histogram

Suppose we completely shuffle an image at random — swapping the positions of all its pixels.

Although the image's appearance changes entirely, its histogram remains exactly the same as before.

The reason is simple.

A histogram only counts how many times each intensity value occurs; it has nothing to do with where the pixels are located.

This is exactly the problem that prevents global methods from taking the image's true structure into account.

The meaning of an image comes from how pixels are arranged relative to each other, not just from their individual intensity values.

---

# The Core Idea Behind Adaptive Histogram Equalization

The solution to this problem is quite simple.

Instead of processing the entire image at once, we divide the image into small regions.

For example:

- 8×8
- 16×16
- 32×32

or any size that suits the problem.

Histogram equalization is then performed separately on each region.

As a result, each region is processed according to its own conditions.

This is why the method is called **Adaptive Histogram Equalization**.

---

# How It Works

Suppose the image is divided into a number of small blocks.

```
□ □ □ □
□ □ □ □
□ □ □ □
□ □ □ □
```

The algorithm processes each block in turn.

For each block:

1. The histogram of that region is computed.
2. Histogram equalization is applied only to that block.
3. The algorithm then moves on to the next block.

This way, each part of the image is enhanced according to the characteristics of that specific region.

---

# Is Adaptive Histogram Equalization Completely Problem-Free?

No.

If we simply divide the image into small blocks and equalize each one independently, a new problem arises.

Each block is processed independently of the others.

As a result:

- One block might become very bright.
- The neighboring block might be a bit darker.
- The next block might become bright again.

Eventually, the boundaries between blocks become visible, and the image takes on a checkerboard-like appearance.

This phenomenon is usually called a **Block Artifact**.

---

# The Second Part of CLAHE: Contrast Limited

To prevent excessive contrast enhancement, the second part of the algorithm comes into play.

CL stands for:

**Contrast Limited**

At this stage, the algorithm doesn't allow the histogram of a region to be stretched beyond a certain point.

In simple terms, a limit is defined for how much contrast enhancement is allowed.

If the number of pixels at a given intensity exceeds this limit, that excess is clipped.

This is why the term **Clipping** is commonly used here.

---

# How Does Clipping Work?

Suppose a block has a large number of pixels sharing a particular intensity value.

In regular histogram equalization, this would cause a drastic increase in contrast.

But in CLAHE, a specific limit is defined beforehand.

If the height of a histogram bar exceeds this limit, the excess portion is removed.

This excess is then redistributed uniformly across the other intensity values.

As a result:

- Contrast enhancement is kept under control.
- Noise is amplified less.
- The image's natural appearance is preserved.

<img width="960" height="350" alt="image" src="https://github.com/user-attachments/assets/4b27064c-68a5-4355-a4c7-df20bde6009a" />

---

# Why Does the Clip Limit Matter?

One of the most important parameters in CLAHE is the **Clip Limit**.

This parameter determines how much contrast enhancement the algorithm is allowed to apply.

If the Clip Limit is too small:

- The image stays more natural.
- Contrast enhancement will be weaker.

If the Clip Limit is large:

- Contrast increases more.
- The risk of amplified noise also increases.

So this parameter should be chosen based on the type of image.

---

# A Problem That Still Remains

Even with a Clip Limit, there's still one issue left.

Different blocks are still processed independently of each other.

As a result, the boundary between two blocks may still be visible.

To solve this, CLAHE uses one more step.

---

# Bilinear Interpolation

After all blocks have been processed, the algorithm applies **Bilinear Interpolation** between neighboring blocks.

The goal of this step is to smooth out the boundaries between blocks.

Instead of each block appearing completely isolated, pixel values are also adjusted using their neighboring blocks.

As a result, changes in intensity happen gradually, and the boundaries between blocks are nearly eliminated.

---

# Advantage of Bilinear Interpolation

This step results in:

- Smoother brightness transitions between blocks.
- No checkerboard pattern in the image.
- Elimination of artificial boundaries.
- A more natural-looking final result.

This process may reduce sharpness very slightly at some edges of the image, but in practice, its benefits far outweigh this minor drawback.

---

# Advantages of CLAHE

CLAHE offers many advantages over regular histogram equalization, including:

- Local image processing
- Preservation of the image's true structure
- Controlled contrast enhancement
- Prevention of excessive noise amplification
- Reduced formation of saturated regions
- Better preservation of image detail
- Prevention of drastic changes in uniform regions
- Well suited for medical images, satellite images, industrial images, and low-light images

---

# CLAHE on Color Images

One of the most important advantages of CLAHE is that it also performs very well on color images.

This algorithm is usually applied to the brightness channel (Value or Luminance).

As a result:

- The image's contrast increases.
- The image's original colors are largely preserved.
- The severe color-shifting problem seen with histogram equalization occurs far less often.

This is why CLAHE is considered one of the most common contrast enhancement methods for color images.

---

# Important Parameters in OpenCV

In OpenCV, CLAHE is created using the following function:

```python
clahe = cv2.createCLAHE(
    clipLimit=2.0,
    tileGridSize=(8,8)
)
```

The two important parameters in this function are:

### `clipLimit`

Determines the limit on contrast enhancement.

The higher this value, the more contrast will be created.

---

### `tileGridSize`

Determines the size of the blocks the image is divided into.

For example:

```python
tileGridSize = (8,8)
```

means the image is divided into 8×8 blocks, and equalization is performed independently on each block.

---

# Summary

CLAHE is an advanced version of histogram equalization designed to solve the problems of traditional methods.

The algorithm first divides the image into small blocks and performs histogram equalization independently on each one. It then uses a **Clip Limit** to prevent excessive contrast enhancement, and finally applies **Bilinear Interpolation** to smooth out the boundaries between blocks so the image looks more natural.

For this reason, CLAHE is considered one of the best and most widely used contrast enhancement methods in many real-world image processing applications — especially for low-light images, medical images, industrial images, and color images.
