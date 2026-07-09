
# Brightness and Contrast in Image Processing

## Introduction

We've talked about **Contrast Enhancement** and methods such as:

- Histogram Stretching
- Histogram Equalization
- CLAHE

In these methods, our main goal was to increase or correct the **contrast** of an image.
But a low-contrast image isn't always the problem.

Sometimes an image already has good contrast, meaning:

- There are bright regions.
- There are dark regions.
- The difference between objects and the background is clearly visible.

But the overall image is too dark or too bright.

In this case, the main issue relates to the image's **brightness**.

<img width="1600" height="1067" alt="lowlightroom" src="https://github.com/user-attachments/assets/65349d5d-61ff-48a8-b5f4-a44a2cbc1cd9" />

---

# The Difference Between Brightness and Intensity

To understand this topic better, we first need to know the difference between these two concepts.

## Intensity

Intensity refers to the brightness value of a specific pixel.

For example, in a grayscale image:

- A value of 0 represents pure black.
- A value of 255 represents pure white.

Every pixel has its own intensity value.

For example:

```
Pixel A = 50
Pixel B = 200
```

meaning these two pixels have different intensity values.

---

## Brightness

Brightness is a more general concept.

Brightness refers to the overall brightness of the image as a whole.

That is, we examine whether the entire image:

- Is dark?
- Is bright?
- Or is in a normal state?

For example:

Suppose we have a photo of a person taken in front of a light source.

It's possible that:

- The image has bright areas.
- The image also has dark areas.
- The image's contrast is good.

But the overall image is darker than what the eye perceives.

In this case, the main problem is:

**Low Brightness**

not low contrast.

---

# The Relationship Between Brightness and Contrast

In real images, these two problems can exist at the same time.

That is, an image might have:

- Low brightness.
- Low contrast as well.

Or:

- Excessive brightness.
- Poor contrast.

Usually, though, one of these problems dominates, and we need to choose the appropriate method based on that specific problem.

---

# Example: A Low-Brightness Image

Suppose we have an image where:

- Its intensity range is nearly complete.
- It has bright pixels.
- It also has dark pixels.

In this image:

The main problem isn't contrast.

Because a difference between bright and dark areas already exists.

But the image still looks a bit dark.

In this case, our goal is:

**Increasing brightness**

---

# Do Contrast Enhancement Methods Solve the Brightness Problem?

At first glance, we might think:

"If the image is dark, let's just run equalization or CLAHE."

But the result isn't always what we want.

---

## Histogram Equalization

If we apply equalization directly to an RGB color image:

- The color channels change independently.
- The color balance gets disrupted.
- The image looks unrealistic.

So applying equalization directly to RGB is usually not a good choice.

If we do want to use equalization, it's better to apply it only to the image's brightness channel.
<img width="948" height="638" alt="image" src="https://github.com/user-attachments/assets/5f4dbf87-a283-47d0-b381-2dfb9d31082b" />

---

# Using CLAHE

CLAHE is one method that can help in this situation.

Because it:

- Examines the image locally.
- Processes each region according to its neighbors.
- Increases contrast in a controlled way.

But CLAHE is still primarily designed for **improving contrast**, not directly correcting brightness.

So if the main problem is brightness, other methods are more suitable.

<img width="944" height="630" alt="image" src="https://github.com/user-attachments/assets/8a2ee061-0add-4927-b104-6ac9b351dd66" />

---

# Linear Correction

One simple method for correcting brightness and contrast is:

## Linear Intensity Transformation

The general formula for this method is:

$$I'(x,y)=\alpha I(x,y)+\beta$$

where:

- $$I(x,y)$$ is the pixel's original intensity value.
- $$I'(x,y)$$ is the pixel's new value.
- $$\alpha$$ is the contrast adjustment factor.
- $$\beta$$ is the brightness adjustment amount.

---

# The Role of Beta in Linear Correction

If we only add a constant value:

$$I'(x,y)=I(x,y)+\beta$$

what happens?

All the image's values shift by a fixed amount.

For example:

Before:

```
0  ----  50
```

After applying:

$$\beta=60$$

it becomes:

```
60 ---- 110
```

meaning:

- The image has become brighter.
- But the gap between values hasn't changed.

So:

Brightness has changed, but contrast has stayed the same.

---

# The Role of Alpha in Linear Correction

Now suppose:

$$I'(x,y)=\alpha I(x,y)$$

When:

$$\alpha > 1$$

values spread further apart from each other.

For example:

Before:

```
0 ---- 50
```

With:

$$\alpha=2$$

it becomes:

```
0 ---- 100
```

meaning:

- The gap between intensity values has widened.
- Contrast has increased.

So:

**The value of Alpha affects contrast.**

---

# The Problem with Linear Correction

Despite its simplicity, this method has some limitations.

## 1. Loss of Information

An image's intensity range is limited.

For 8-bit images:

$$0 \leq I \leq 255$$

If a new value exceeds 255, it gets clipped.

For example:

```
270 → 255
```

As a result, several different values collapse into the same value.

This causes a loss of detail.

<img width="1208" height="412" alt="image" src="https://github.com/user-attachments/assets/b52bd617-7101-4d8b-aedc-9a42053f6e94" />

For instance, when alpha equals 2, a value like 200 also gets multiplied by 2, gets clipped, and its detail is lost.

---

## 2. Excessive Brightness Increase

If the value of Alpha becomes too large:

- Bright regions quickly become saturated.
- Detail in bright areas is lost.

For example, in an image of a window:

Before:

- A tree behind the window is visible.

After a strong increase:

- The window turns completely white.
- The information behind it is erased.

---

With Beta, values shift and this can reduce our contrast.

We end up going outside the valid range and losing information.

So the linear approach has drawbacks, which means we need a method that can perform better.

---

# Gamma Correction

To address the problems of linear correction, we use another method:

## Gamma Correction

This method applies a non-linear transformation to the image.

The general formula:

$$I' = 255(\frac{I}{255})^\gamma$$

---

# The Idea Behind Gamma Correction

First, we convert the image's values from the range:

[0-255]

to the range:

[0-1]

Then we apply the Gamma value.

---

# Gamma Behavior

## When Gamma = 1

When:

$$\gamma=1$$

then:

$$I'=I$$

meaning no change occurs.

---

## Gamma Less Than 1

If:

$$\gamma < 1$$

values become brighter.

For example:

```
0.2 → 0.45
```

So for dark images:

**A Gamma value less than 1 increases brightness.**

---

## Gamma Greater Than 1

If:

$$\gamma > 1$$

values become smaller.

Therefore:

- The image becomes darker.
- This is suitable for images that are too bright.

---

# Underexposed and Overexposed

There are two important terms in image processing:

## Underexposed

An image that has too little light.

That is:

- It's dark.
- It has low brightness.

To correct it, we usually use:

$$\gamma < 1$$

---

## Overexposed

An image that has received too much light.

That is:

- It's very bright.
- Detail in bright areas has been lost.

To correct it, we usually use:

$$\gamma > 1$$

---

# Comparing Linear and Gamma Correction

## Linear Correction

Advantages:

- Simple
- Fast
- Controllable

Disadvantages:

- May cause pixels to saturate.
- Detail in bright or dark areas may be lost.
- Changes are entirely linear.

---

## Gamma Correction

Advantages:

- Non-linear.
- Better preserves the 0–255 range.
- Less detail is lost.
- Better suited for correcting brightness.

Disadvantages:

- Choosing the right Gamma value matters a lot.
- Some images may need experimental tuning.

---

# Summary

When enhancing images, we first need to identify what the actual problem is:

If the problem is **contrast**:

- Histogram Stretching
- Histogram Equalization
- CLAHE

are suitable choices.

If the problem is **brightness**:

- Linear Correction
- Gamma Correction

are more appropriate choices.

It's also worth noting that brightness and contrast are related to each other, and changing one can affect the other.

Among brightness-correction methods, Gamma Correction generally performs better, since it allows brightness to be increased or decreased without pushing values drastically outside the valid range, thereby preserving the image's information more effectively.
