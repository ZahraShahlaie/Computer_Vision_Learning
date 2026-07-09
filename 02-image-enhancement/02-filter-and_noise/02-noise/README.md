
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

---

# Generating Gaussian Noise

This type of noise is usually generated using a normal distribution.

In NumPy, Gaussian noise can be generated with the following function:

```python
np.random.normal(mean, std, image.shape)
```

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

# Summary

Noise is one of the most significant factors that degrade the quality of digital images. It can arise from hardware limitations, poor imaging conditions, increased ISO, data transmission, or image storage. Beyond reducing visual quality, noise also lowers the accuracy of many computer vision and image processing algorithms. For this reason, denoising is considered one of the most fundamental preprocessing steps in image processing, and choosing the right denoising method plays a key role in improving image quality and the performance of AI models.
