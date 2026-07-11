
# Image Sharpening

## Introduction

In some images, even when brightness and contrast are just fine, we still feel that the image looks a bit **blurred**. In such images, edges don't have enough clarity, and different parts of the image aren't well separated from each other.

For example:

- The branches of a tree may not be clearly distinguishable.
- The texture of grass may look uniform and flat.
- Details of mountains or rocks may be hard to make out.

In this situation, the problem is usually **weak image edges**, and clarity can be improved using **sharpening**.

---

# What Is Sharpening?

**Image sharpening** is a process that increases the intensity difference around edges.

As a result:

- The boundaries between objects become clearer.
- Fine details become more visible.
- The image looks crisper.

The important thing to understand is that sharpening doesn't add any new information to the image; it only increases the **local contrast around edges** so they become more visible.

---

# Sharpening Filters

One of the *simplest* sharpening methods is using convolution kernels.

Two of the most common kernels are:

### First Filter

```text
 0  -1   0
-1   5  -1
 0  -1   0
```

---

### Second Filter

```text
-1  -1  -1
-1   9  -1
-1  -1  -1
```

Both filters boost the value of the central pixel while reducing the influence of its neighbors, which increases the image's clarity.



<img width="1189" height="253" alt="image" src="https://github.com/user-attachments/assets/eb77e36c-3ce2-4acd-9b34-c64584a5629f" />


---

# How Do These Filters Work?

At the center of the kernel there's a large positive value, while the neighbors carry negative coefficients.

During convolution:

- The central pixel's value gets multiplied several times over.
- A portion of the neighboring values is subtracted from it.

As a result, the difference between the central pixel and its surroundings increases, and edges become clearer.

This is why, after applying a sharpening filter, details such as tree branches, grass texture, or rocks on a mountain become easier to see.

---

# Why Does the Image Look Clearer?

Edges are where intensity changes abruptly.

When sharpening amplifies this difference:

- Object boundaries become clearer.
- Fine textures become more visible.
- The image looks crisper.

In effect, sharpening increases **contrast around edges**, not the overall contrast of the entire image.

---

# The Relationship Between Sharpening and Blur

Sharpening can, to some extent, be thought of as the opposite of blur.

### With Blur

The goal is to reduce the difference between neighboring pixels.

As a result:

- Noise is reduced.
- Edges become softer.
- The image becomes somewhat blurry.

---

### With Sharpening

The goal is to increase the difference between neighboring pixels.

As a result:

- Edges are amplified.
- Details become more visible.
- The image looks clearer.

So blur and sharpening are essentially the reverse of one another.

---

# Why Does Noise Become More Visible After Sharpening?

One of the most important things to keep in mind when sharpening an image is that **noise gets amplified**.

Like edges, noise also produces a sudden change in intensity.

Since sharpening amplifies all abrupt changes in intensity, it doesn't distinguish between a real edge and noise.

As a result:

- Edges become clearer.
- Noise also becomes more noticeable.

That's why, if the original image already contains noise, it usually becomes more pronounced after sharpening.

---

# The Mathematical Reason Behind This

In averaging filters, a pixel's value is replaced with the average of its neighbors.

So if a noisy pixel exists, its value gets spread out among its neighbors, reducing its effect.

But sharpening works in exactly the opposite way.

The central pixel receives a large coefficient, while its neighbors participate in the calculation with negative coefficients.

As a result, if the central pixel is noisy, that same noise gets amplified too.

That's why it's usually recommended to first reduce the image's noise and only then apply sharpening.

---

# Why Must the Kernel's Coefficients Sum to 1?

One important property of sharpening kernels is that their coefficients must sum to **1**.

For example:

### First Kernel

```text
0  -1   0
-1   5  -1
0  -1   0
```

Sum of coefficients:

```text
5 - 4 = 1
```

---

### Second Kernel

```text
-1 -1 -1
-1  9 -1
-1 -1 -1
```

Sum of coefficients:

```text
9 - 8 = 1
```

---

If the sum of the coefficients equals 1:

- The image's overall brightness is preserved.
- The image won't become excessively dark or bright.

If this sum is greater than 1, the image becomes brighter, and if it's less than 1, the image becomes darker.

---

# The Difference Between the Two Sharpening Kernels

Both kernels amplify edges, but the intensity of their effect differs.

The kernel with a central value of **5**:

- Applies milder sharpening.
- Amplifies less noise.
- Produces a more natural-looking result.

But the kernel with a central value of **9**:

- Amplifies edges more strongly.
- Reveals more detail.
- In turn, amplifies noise more as well.

So the larger the central pixel's coefficient, the stronger the sharpening effect.

---

# Advantages

- Increased image clarity
- Amplified edges
- Better display of detail
- Improved visual quality

---

# Disadvantages

- Amplification of existing noise
- Unnatural appearance if overused
- Possible halo effect around edges
- Reduced image quality with excessive sharpening

---

# Summary

Sharpening is one of the most important steps in improving image quality; by increasing the intensity difference around edges, it makes objects clearer and reveals details more effectively. This process doesn't add any new information to the image — it only makes existing edges more prominent.

At the same time, since noise is also a kind of sudden change in intensity, the sharpening process can amplify existing noise as well. That's why, in many image processing applications, **denoising** is performed first, and **sharpening** is then used to increase clarity and better reveal detail.
