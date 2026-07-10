# Image Denoising

## Introduction

In the real world, almost no image is ever captured completely free of noise. Any image recorded by a camera, sensor, or imaging equipment usually contains some amount of **noise**. The presence of noise degrades image quality and makes it harder to extract information from it.

For this reason, one of the first steps in many image processing and computer vision systems is **image denoising**. The goal of this step is to reduce noise as much as possible while preserving the image's essential details.

---

# What Is Noise?

Put simply, noise refers to unwanted information that gets added to an image during capture or transmission.

The presence of noise pulls an image away from its natural state and reduces its visual quality.

For example, an image affected by noise might show:

- White and black spots appearing.
- A grainy appearance.
- Slightly shifted colors.
- A foggy or dull look.

In effect, noise introduces random, unwanted variations in pixel values that have nothing to do with the actual content of the image.

---

# Why Does Noise Matter?

Noise doesn't just harm the appearance of an image — it also affects the performance of many image processing algorithms.

For example, suppose we want to design a face recognition system.

```
Image → Recognition model → Person's name
```

If the image contains noise, the model receives incorrect information, and the likelihood of errors increases.

This same issue applies to many other applications as well, such as:

- Face recognition
- Object detection
- Image segmentation
- Edge detection
- OCR
- Medical imaging
- Self-driving cars

For this reason, denoising is usually performed before any other processing takes place.

---

# How Is Noise Created?

Various sources can introduce noise into an image.

The most important ones include:

- Camera sensor
- Low hardware quality
- Low ambient light
- Increased sensor sensitivity (ISO)
- Data transmission
- Image compression
- File storage
- High sensor temperature

So even if the scene itself is completely static, the captured image may still contain noise.

---

# The Role of the Camera Sensor

Every camera is made up of millions of tiny sensors that capture light.

Each sensor records the amount of light it receives, and the final image is built from these readings.

If one of these sensors doesn't function correctly, it produces an incorrect value, which results in noise appearing in the image.

That's why sensor quality has a direct impact on image quality.

---

# Noise in Low-Light Conditions

One of the most common causes of noise is shooting in low light.

In such conditions, the sensor tries to capture more light.

To do this, the **ISO** value is usually increased.

The higher the ISO:

- The brighter the image becomes.
- But noise also increases.

That's why photos taken at night usually have more noise than photos taken during the day.

---

# Types of Noise

There are several different types of noise in image processing, each behaving differently.

Two of the most common ones are:

- Salt & Pepper Noise
- Gaussian Noise

---

# Salt & Pepper Noise

This is one of the simplest and most well-known types of noise.

In this case, a number of pixels are randomly turned into either **0** or **255**.

As a result, black and white spots appear scattered across the image.

That's why it's called **salt-and-pepper noise**.

Characteristics of this noise:

- Completely white spots
- Completely black spots
- A relatively small number of corrupted pixels
- Sudden, abrupt changes in the image

This type of noise is usually caused by sensor malfunctions or data transmission errors.

<img width="1389" height="579" alt="image" src="https://github.com/user-attachments/assets/c9548802-ec2c-4db7-b2b7-b0ab2d50b388" />

---

# Generating Salt & Pepper Noise

To simulate this noise, a number of pixels are usually selected at random.

Then:

- Some of them are set to zero.
- Others are set to 255.

As a result, the image ends up with random white and black spots.

---

# Gaussian Noise

One of the most common types of noise found in real-world images is **Gaussian noise**.

Unlike salt-and-pepper noise, which only affects a handful of pixels, Gaussian noise affects nearly the entire image.

In this case, a random value drawn from a normal distribution is added to each pixel's value.

As a result, the image takes on a grainy appearance and its quality decreases.

<img width="1153" height="456" alt="image" src="https://github.com/user-attachments/assets/13e2aee6-72e1-429a-9242-34ee2ab06b01" />

---

# Generating Gaussian Noise

This type of noise is usually generated using a normal distribution.

In NumPy, Gaussian noise can be generated with the following function:

```python
np.random.normal(mean, std, image.shape)
```

<img width="569" height="413" alt="image" src="https://github.com/user-attachments/assets/c476624f-b4fc-44a5-8055-61b51f7e6117" />

The important parameters are:

- **Mean**: the average value of the noise (usually zero)
- **Standard deviation**: the intensity of the noise

The larger the standard deviation, the stronger the noise.

---

# Why Do We Clip After Adding Noise?

After noise is added, some pixel values may fall outside the image's valid range.

For example:

- Negative values may be produced.
- Values greater than 255 may be created.

Since 8-bit images only accept values between **0 and 255**, we need to constrain these values.

That's why the `np.clip` function is used.

```python
image = np.clip(image, 0, 255)
```

Finally, the image is converted back to the `uint8` data type.

---

# Effect of Noise on the Histogram

The presence of noise alters the distribution of intensity values in an image.

As a result:

- The image's histogram becomes wider.
- New peaks are formed.
- The intensity distribution deviates from its natural shape.

So examining a histogram can also help detect the presence of noise to some extent.

---

# Effect of Noise on Edges

One of the most significant effects of noise is the degradation of image edges.

Edges are the regions where intensity changes abruptly.

Most computer vision algorithms extract important image information from precisely these edges.

When noise is present:

- Edges become blurred.
- Object boundaries become unclear.
- Important image features are lost.

As a result, the performance of many image processing algorithms declines.

---

# Should Noise Always Be Removed?

In nearly every image processing project, denoising is considered one of the preprocessing steps.

That said, in some deep learning models, noisy images are intentionally fed to the model during training so that it becomes more robust to noise.

But even in this case, at actual inference time, the system usually still tries to denoise the image as much as possible before feeding it to the model.

---

# Average Filter

One of the simplest methods for removing noise is the **average filter**, implemented in OpenCV with the `cv2.blur()` function.

The idea behind this filter is very simple. For each pixel, its value is replaced with the **average of its surrounding pixels**.

Suppose we use a 3×3 kernel. In this case, each pixel's value is computed as the average of the 9 pixels in its neighborhood.

In other words, all neighbors have equal weight and contribute equally to the final result.

---

## How It Works

If one of the surrounding pixels has an abnormal (noisy) value, averaging reduces its impact.

For example, suppose the values in a region are:

```
10   20   15
18  100   22
12   19   17
```

The number **100** is an abnormal value and likely represents noise.

The average filter computes the average of these 9 values and replaces the central value with it. As a result, the value 100 gets pulled closer to its neighbors, and the intensity of the noise is reduced.

---

## Advantages

The main advantage of the average filter is its simplicity and speed.

This filter:

- Is very simple to implement.
- Runs very fast.
- Is suitable for removing weak noise.
- Has low computational cost.

---

## The Main Problem with the Average Filter

Despite its simplicity, this filter has one significant drawback.

Not all changes in an image come from noise.

In many parts of an image, a sudden change in intensity is due to the presence of an **edge**.

Edges represent the most important information in an image and mark the boundaries between objects.

For example, the boundary between a flower and its background, or between a butterfly and a flower, are both edges.

When the average filter is applied to an image, it averages across all pixels without any distinction. As a result:

- Noise is reduced.
- But edges also get smoothed and blurred.

That's why the image ends up looking blurred, losing some of its detail.

The larger the kernel size (such as 5×5 or 7×7), the greater this blurring effect, and the more detail gets removed from the image.

---

# Gaussian Filter

To reduce this problem, the **Gaussian filter** is used.

In OpenCV, this filter is implemented with the following function:

```python
cv2.GaussianBlur()
```

Unlike the average filter, which treats all neighbors equally, the Gaussian filter assigns a **different weight** to each pixel.

---

## The Core Idea Behind the Gaussian Filter

In this filter, the closer a pixel is to the center of the kernel, the more important it is.

Pixels farther away receive less weight.

So during averaging, information near the central pixel has a greater influence on the final result.

These weights are computed based on a **normal (Gaussian) distribution**.

As a result:

- The center of the kernel carries the highest weight.
- As distance from the center increases, weights gradually decrease.

This causes the image to blur less than with the average filter, while edges are better preserved.

---

## The Role of the Sigma Parameter

In Gaussian blur, in addition to kernel size, there's another important parameter known as **sigma (σ)**. Sigma determines the spread of the Gaussian distribution.

If sigma is small:

- The weight of the central pixel is much greater than the others.
- Farther pixels have little influence.
- Image details are better preserved.

If sigma becomes larger:

- The weight of farther pixels also increases.
- Averaging is performed over a larger region.
- The image becomes softer and blurrier.

So choosing the right sigma value plays an important role in output quality.

---

## Effect of Kernel Size

Like the average filter, the kernel here must also have odd dimensions, such as:

- 3×3
- 5×5
- 7×7
- 9×9

The larger the kernel size, the more noise gets removed, but the greater the chance of losing detail as well.

So a balance must be struck between removing noise and preserving detail.

---

# Comparing the Average Filter and the Gaussian Filter

Both filters work based on averaging, but there's an important difference between them.

**Average Filter**

- All pixels have equal weight.
- Reduces noise.
- Heavily blurs edges.
- Makes the image blurrier.

**Gaussian Filter**

- Pixel weights differ.
- The central pixel carries more importance.
- Details and edges are better preserved.
- Usually produces a more natural, higher-quality result.

---

# Performance on Gaussian Noise

The Gaussian filter is designed for removing **Gaussian noise**.

With this type of noise, nearly all pixels in the image are affected, and a layer of noise is visible across the entire image.

Applying Gaussian blur significantly reduces this noise and makes the image look more natural.

That said, no filter can completely remove Gaussian noise; the goal is to reduce the noise's intensity while preserving as much of the image's information as possible.

---

# Performance on Salt & Pepper Noise

If we apply these same filters to **salt & pepper noise**, the result won't be very satisfactory.

The reason is that this type of noise consists of pixels with extremely large or extremely small values.

Averaging only slightly reduces the value of these pixels but doesn't fully remove them.

As a result:

- The noise still remains in the image.
- At the same time, the image becomes blurrier.

So the average filter and the Gaussian filter aren't good choices for removing salt & pepper noise.

---

# Median Filter

## Core Idea

Unlike the average filter, which uses the **average** of neighboring values, the median filter uses the **median**.

All the values within the kernel window are first sorted, and then the middle value is selected as the pixel's new value.

That's why this method is called the **median filter**.

---

## How It Works

Suppose we have a 3×3 kernel with the following values:

```
1   2   5
5   255 10
4   3   6
```

The number **255** is a salt & pepper noise value.

### If we take the average:

The presence of the value 255 causes the average to increase drastically, so the new value is still affected by the noise.

### If we take the median:

First, the values are sorted:

```
1, 2, 3, 4, 5, 5, 6, 10, 255
```

Then the middle value is selected:

```
Median = 5
```

As a result, the noisy value **255** has no significant effect on the output and gets removed.

---

## Why Must Kernel Size Be Odd?

Just like other filters, the kernel size in the median filter must also be **odd**.

For example:

- 3×3
- 5×5
- 7×7

The reason is the existence of a single **middle element**.

If the number of elements were even, there would be no single middle value, and we'd have to average the two middle values — whereas the goal of the median filter is to select the middle value itself, not its average.

---

## Effect of Kernel Size

The larger the kernel:

- The more noise gets removed.
- The smoother the image becomes.
- The greater the chance of losing fine detail.

So kernel size should be chosen based on the amount of noise present.

---

# Median Filter on Gaussian Noise

When the median filter is applied to Gaussian noise, the result isn't very satisfactory.

The reason is that Gaussian noise affects **nearly every pixel** in the image, rather than creating just a few outlier values.

With Gaussian noise, each pixel's value usually only changes slightly.

For example:

```
10 → 13
15 → 17
20 → 18
```

In this situation, nearly all the values are close to one another, and there's no strong outlier that the median could remove.

That's why the median filter has limited effect on Gaussian noise.

---

# Median Filter on Salt & Pepper Noise

This filter is specifically designed for exactly this type of noise.

With salt & pepper noise, a number of pixels are suddenly turned into extremely small or extremely large values.

For example:

```
0
255
0
255
```

Since these values end up at the very beginning or end of the sorted list, they're usually not selected as the middle value, and the noise is removed effectively.

That's why the median filter is one of the best options for removing salt & pepper noise.

---

## Advantages

- Very effective at removing salt & pepper noise
- Preserves edges better than the average filter
- Robust against outlier values
- Simple to implement

---

## Disadvantages

- Performs poorly on Gaussian noise.
- With large kernels, fine details may be lost.
- Can smooth out some small textures in the image.

---

# Bilateral Filter

## Core Idea

One of the most significant problems with average and Gaussian blur is that they only consider the distance between pixels and ignore differences in intensity.

As a result, during averaging, the image's edges also get blurred.

The bilateral filter solves this problem by using **two criteria at the same time**:

1. Spatial distance between pixels
2. Intensity or color difference

That's why it's called a **bilateral filter**.

---

# First Criterion: Spatial Distance

This part uses the same idea as the Gaussian filter.

Closer pixels have higher weight, and as distance increases, weight decreases.

So nearby neighbors have a greater influence on the averaging.

---

# Second Criterion: Intensity

In addition to distance, the bilateral filter also examines the difference in intensity.

If two pixels have similar intensity values, they're likely to belong to the same object, so they receive greater weight.

But if the intensity difference is large, they're likely on opposite sides of an edge, in which case they receive very little weight.

This property is what allows **edges to be preserved** during denoising.

---

## Preserving Edges

Suppose part of an image contains two regions:

```
10  12  11
13  12  14

90  92  91
89  93  90
```

The intensity difference between these two regions is very large.

Average or Gaussian filters would blend these two regions together, blurring the boundary between them.

But the bilateral filter recognizes that the intensity difference is large, so pixels on either side of the edge have very little influence on each other, and the image's boundary is preserved.

---

# Bilateral Filter Parameters

In OpenCV, this filter is used as follows:

```python
cv2.bilateralFilter(
    src,
    d,
    sigmaColor,
    sigmaSpace
)
```

The important parameters are:

### d

The diameter of the neighborhood region, or filter size.

---

### sigmaColor

Determines the acceptable amount of intensity difference.

- Small value ⇒ only pixels with very similar intensity participate in the averaging.
- Large value ⇒ larger intensity differences are accepted, and the image becomes smoother.

---

### sigmaSpace

Determines how much spatial distance affects the result.

- Small value ⇒ only very close neighbors have an effect.
- Large value ⇒ farther neighbors also contribute to the averaging.

---

# Choosing Parameters

There's no fixed value for the bilateral filter.

The best values are usually found through trial and error.

Several different values need to be tried to strike the right balance between:

- Noise reduction
- Edge preservation
- Detail preservation

---

# Bilateral Filter on Gaussian Noise

This filter is one of the best options for removing Gaussian noise.

The reason is:

- Noise is reduced.
- Edges are largely preserved.
- Image details are retained better than with Gaussian blur.

For this reason, the bilateral filter is used in many image processing applications for removing Gaussian noise.

---

# Bilateral Filter on Salt & Pepper Noise

The bilateral filter doesn't perform well on salt & pepper noise.

With this type of noise, the intensity difference is very large, and noisy pixels usually have values of 0 or 255.

As a result, the filter can't correct these outliers effectively, and much of the noise remains.

So the bilateral filter isn't a good choice for salt & pepper noise.

---

# Computational Cost

Compared to average, Gaussian, and median filters, the bilateral filter performs significantly more computation.

That's because for each pixel it must simultaneously compute:

- Spatial distance
- Intensity difference

Also, the larger the kernel size, the more computations are needed, and the longer the algorithm takes to run.

So although the bilateral filter produces very high-quality output, it also comes with a higher computational cost.

---

# Final Filter Comparison

| Filter | Best For | Edge Preservation | Speed |
|---|---|---|---|
| Average Filter | Weak noise | Poor | Very fast |
| Gaussian Filter | Gaussian noise | Moderate | Fast |
| Median Filter | Salt & Pepper noise | Good | Moderate |
| Bilateral Filter | Gaussian noise | Very good | Slow |

Salt & Pepper noise

<img width="1189" height="715" alt="image" src="https://github.com/user-attachments/assets/8d46b9a8-7610-4b1b-be9e-dce4479a2e9c" />

Gaussian noise
<img width="1189" height="715" alt="image" src="https://github.com/user-attachments/assets/f6e30436-0906-418c-9bb6-325ccfa2c7c5" />

---

# Summary

Noise is one of the most significant factors that degrade the quality of digital images. It can arise from hardware limitations, poor imaging conditions, increased ISO, data transmission, or image storage. Beyond reducing visual quality, noise also lowers the accuracy of many computer vision and image processing algorithms. For this reason, denoising is considered one of the most fundamental preprocessing steps in image processing, and choosing the right denoising method plays a key role in improving image quality and the performance of AI models.

The average filter is the simplest denoising method, but since it gives equal weight to all pixels, it blurs edges and destroys image detail.

The Gaussian filter, by using weights based on a Gaussian distribution, performs better — it reduces noise while preserving the image's structure and edges more effectively. For this reason, it's one of the most widely used denoising filters in image processing.

That said, both of these methods rely on averaging and aren't a good choice for removing **salt & pepper noise**. For this type of noise, the **median filter** is usually used, which we examined in the section above.

Each filter is designed for a specific type of noise, and no single method is the best choice for every situation:

- The **average filter** is the simplest denoising method but destroys image detail.
- The **Gaussian filter** performs better than the average filter on Gaussian noise but still blurs some edges.
- The **median filter** is the best option for removing salt & pepper noise, since it's robust against outlier values and removes them effectively.
- The **bilateral filter**, by considering both spatial distance and intensity difference at the same time, reduces Gaussian noise while largely preserving the image's edges — though it comes with a higher computational cost than the other filters.
