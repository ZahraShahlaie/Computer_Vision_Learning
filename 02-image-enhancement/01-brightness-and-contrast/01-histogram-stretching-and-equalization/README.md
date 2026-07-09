
# 📈 Image Contrast Enhancement

## 📌 Introduction

In many images, details are not clearly visible. This is usually caused by low **contrast**. When the difference in brightness between different regions of an image is small, objects become hard to distinguish from the background, and the image looks dull and flat.

One of the most important topics in **Image Enhancement** is increasing contrast. The goal of this step is to improve the visual quality of an image and make details easier to recognize, without changing the actual content of the image.

---

# What Is Contrast?

Contrast refers to the difference in **intensity** between different parts of an image.

The greater the difference in intensity, the sharper the boundaries between objects, and the more visible the details become.

On the other hand, if most pixels in an image have similar intensity values, the image is said to have **low contrast**.

---

# Visualizing Contrast with a Histogram

One of the best tools for examining an image's contrast is the **histogram**.

A histogram shows how many times each intensity level appears in the image.

For grayscale images, intensity values lie in the range:

0 to 255

If most pixels fall within a small range, the image has low contrast.

Example:

```
         ███████████
0 ------------------------------------------ 255
```

Here the image only uses a small portion of the full intensity range.

But in an image with good contrast:

```
████████████████████████████████████████████
0 ------------------------------------------ 255
```

Intensity values are spread over a much wider range.

---

# A Simple Contrast Metric

One simple way to measure contrast is:

$$
Contrast=\frac{I_{max}-I_{min}}{I_{max}+I_{min}}
$$

where:

- $$I_{max}$$: the maximum intensity value in the image
- $$I_{min}$$: the minimum intensity value in the image

This value lies between zero and one:

- Closer to 1 → high contrast
- Closer to 0 → low contrast

This is only a simple definition; other contrast metrics exist in image processing as well.

---

# 🎯 Goal of Contrast Enhancement

The main goal of contrast enhancement is:

> To widen the intensity range of an image and increase the difference between different regions so that details become more visible.

If an image only uses a small portion of the 0–255 range, contrast enhancement methods try to spread these values across a wider range.

---

# Histogram Stretching

Histogram Stretching, also known as Contrast Stretching, is one of the simplest methods for increasing contrast.

The main idea:

> The lowest intensity value in the image is mapped to 0, the highest is mapped to 255, and all other values are linearly distributed between these two.

For this reason, this method is also called:

**Linear Contrast Stretching**

---

# The Concept of Stretching

Suppose the image's intensity values only range between:

90 and 180

Before stretching:

```
      ███████████████
0 ------------------------------------------255
```

After stretching:

```
████████████████████████████████████████████
0 ------------------------------------------255
```

In this case, the image uses a much wider range of intensity values, and contrast increases.

---

# Why Is Stretching Linear?

In this method, all values change by a fixed ratio.

For example:

Before:

```
90
120
180
```

After:

```
0
96
255
```

The relative order of brightness values is preserved, but the distance between them increases.

---

# The Histogram Stretching Formula

$$
I'=\frac{I-I_{min}}{I_{max}-I_{min}}\times255
$$

where:

- $$I$$: the original intensity value
- $$I_{min}$$: the minimum intensity value
- $$I_{max}$$: the maximum intensity value
- $$I'$$: the new intensity value

After applying this transformation:

$$I'_{min}=0$$

and

$$I'_{max}=255$$

---

# Implementation on a Grayscale Image

A grayscale image has only a single intensity channel.

Steps:

1. Find the minimum value
2. Find the maximum value
3. Apply the stretching formula to all pixels
4. Convert the output to `uint8`

Sample code:

```python
i_min = img_gray.min()
i_max = img_gray.max()

img_gray_stretch = (
    (img_gray - i_min) /
    (i_max - i_min)
    * 255
).astype('uint8')
```

---

# Limitations of Histogram Stretching

## Sensitivity to Outliers

The main drawback of histogram stretching is its direct dependence on the image's minimum and maximum values.

Suppose:

Most pixels: 120 to 160

A few pixels: 250

Even if the value 250 belongs to only a handful of pixels, the image's maximum will still be considered 250.

As a result, the stretching range gets determined by a non-representative value.

Consequences:

- The main part of the image doesn't get stretched enough.
- The contrast improvement is smaller than expected.

---

# Creating Banding

Another problem with histogram stretching is that it can create large gaps between intensity values.

If the original intensity values are:

```
100, 101, 102
```

after stretching they might become:

```
20, 80, 150
```

As a result, smooth brightness transitions are lost, and a phenomenon called:

**Banding**

occurs.

---

# Applying Stretching to Color Images

For RGB images, the image is first split into three channels:

```
Red
Green
Blue
```

Stretching is then applied to each channel separately.

Steps:

1. Split the channels
2. Compute the min and max of each channel
3. Apply stretching
4. Merge the channels back together

---

# The Problem with Stretching in RGB

Independently changing the RGB channels breaks the relationship between colors.

As a result:

- Sky color may shift.
- Skin tones may look unnatural.
- The overall color balance of the image may be lost.

For this reason, in many applications it's preferable to work in color spaces such as:

- HSV
- LAB

instead of RGB.

---

# Histogram Equalization

Histogram Equalization is another important method for increasing image contrast.

Its underlying idea differs from histogram stretching.

While stretching only aims to widen the intensity range, equalization aims to:

> Make the distribution of intensity values as uniform as possible.

In other words, equalization tries to spread pixels across the entire intensity range so that each portion of the 0–255 range carries a larger share of the image's information.

---

# The Idea Behind Histogram Equalization

Suppose an image's histogram looks like this:

```
    █████████████████
0 ------------------------------------------255
```

meaning most pixels are concentrated in a small range.

Histogram equalization tries to transform this distribution into a wider one:

```
████████████████████████████████████
0 ------------------------------------------255
```

The goal isn't necessarily a perfectly uniform histogram, since that isn't always achievable, but the algorithm attempts to get as close as possible to a uniform distribution.

---

# Histogram Stretching vs. Histogram Equalization

| Feature | Histogram Stretching | Histogram Equalization |
|---|---|---|
| Transformation type | Linear | Non-linear |
| Goal | Widen the intensity range | Make the intensity distribution uniform |
| Dependence on min/max | High | Lower |
| Preserves histogram shape | Yes | No |
| Contrast change | Gradual | Usually stronger |
| Risk of altering image appearance | Lower | Higher |

---

# Why Does Equalization Handle Outliers Better?

In histogram stretching, only the image's minimum and maximum matter.

In equalization, however, the decision is based on the **distribution of all pixels**.

So if there are a few pixels with very high or very low intensity, their impact is much smaller.

For this reason, equalization typically performs better than stretching on images that contain outliers.

---

That said, it should be noted that equalization doesn't necessarily produce a more visually pleasing image; its main purpose is to increase visible information and improve contrast.

---

# Limitations of Histogram Equalization

Despite its good performance, this method also has some drawbacks.

## 1. Drastic Changes in Appearance

Since equalization changes the intensity distribution, the overall brightness of the image may shift.

For example:

- Dark regions may become excessively bright.
- Bright regions may become overly enhanced.

---

## 2. Noise Amplification and Unwanted Detail Enhancement

If the image contains noise, equalization may amplify that noise as well.

This happens because the algorithm tries to stretch out all intensity variations.

---

# Applying Histogram Equalization to RGB Images

As with histogram stretching, equalization can also be applied to the R, G, and B channels.

Steps:

1. Split the R, G, and B channels
2. Apply equalization to each channel
3. Merge the channels back together

Example:

```python
channels = cv2.split(img)

equalized_channels = []

for ch in channels:
    ch_eq = cv2.equalizeHist(ch)
    equalized_channels.append(ch_eq)

img_rgb_equalized = cv2.merge(equalized_channels)
```

---

# The Problem with Equalization in RGB Space

Applying equalization independently to each RGB channel usually causes drastic color shifts.

Reason:

Each channel changes independently, so the relationship between:

- Red
- Green
- Blue

no longer matches the original image.

Possible results:

- Unrealistic colors
- Shifted sky color
- Unnatural skin tones
- Unwanted color casts

For this reason, it's better not to work directly on RGB channels for color images.

---

# Contrast Enhancement in HSV Space

One better approach for processing color images is to use the HSV color space.

HSV consists of three channels:

## H (Hue)

Represents the color information of the image, such as:

- Red
- Green
- Blue

---

## S (Saturation)

Represents how saturated the color is.

---

## V (Value)

Represents the brightness/intensity of the image.

Since our goal in contrast enhancement is to change brightness, not color, we only process the:

```
V
```

channel.

---

# Advantage of Using HSV

In RGB, any change to one channel can affect the final color.

In HSV, however:

- H → color
- S → saturation
- V → brightness

are separated from each other.

So by modifying only V:

- Contrast increases.
- The original colors of the image are preserved.

---

# Histogram Stretching in HSV Space

Steps:

## 1. Convert RGB to HSV

```python
img_hsv = cv2.cvtColor(
    img,
    cv2.COLOR_RGB2HSV
)
```

---

## 2. Split the Channels

```python
h, s, v = cv2.split(img_hsv)
```

---

## 3. Apply Stretching to V

```python
v_min = v.min()
v_max = v.max()

v_stretch = (
    (v - v_min) /
    (v_max - v_min)
    * 255
).astype('uint8')
```

---

## 4. Merge the Image Back Together

```python
img_hsv_stretched = cv2.merge(
    [h, s, v_stretch]
)
```

---

# Histogram Equalization in HSV Space

Here, equalization is only applied to the V channel:

```python
v_equalized = cv2.equalizeHist(v)
```

Then:

```python
img_hsv_equalized = cv2.merge(
    [h, s, v_equalized]
)
```

---

# Comparing the Two Methods in HSV

## Stretching on V

Advantages:

- Better color preservation
- Increased contrast
- Less alteration of image appearance

Disadvantages:

- Still sensitive to outliers.

---

## Equalization on V

Advantages:

- Stronger contrast enhancement
- Good for images with poor intensity distribution
- Well-suited for medical images and X-rays

Disadvantages:

- The image may become over-enhanced.
- Risk of an unnatural appearance.

---

# Applications in Medical Imaging

One important application of histogram equalization is in medical imaging.

Medical images such as X-rays typically have:

- Low contrast
- Hidden details
- A limited intensity range

Using equalization, it's possible to:

- Make bone edges more visible.
- Better observe internal structures.
- Extract more detail.

For this reason, this method performs very well in many medical applications.

---

# Final Summary

Contrast enhancement is one of the most important steps in image enhancement.

This section covered two main methods:

## Histogram Stretching

A linear method that expands the intensity range from a narrow band to the full 0–255 range.

This method is:

- Simple
- Fast
- Easy to understand

but performs worse in the presence of outliers.

---

## Histogram Equalization

A non-linear method that tries to make the intensity distribution more uniform.

This method:

- Reveals more detail.
- Handles some of stretching's problems better.

but may alter the image's appearance and amplify noise.

---

## Conclusion

No single contrast enhancement method is best for every image.

The right choice depends on the type of image and the goal of the processing.

For color images, it's usually better to process only the brightness component in a color space such as HSV, rather than modifying the RGB channels directly.

Going forward, to address the limitations of histogram equalization, more advanced methods such as:

- CLAHE (Contrast Limited Adaptive Histogram Equalization)

will be explored.
