

# 🖼️ Image Processing in OpenCV

## 📌 Overview

This section covers how images are handled in **OpenCV**, including reading images, understanding data types, color spaces, and conversion between formats.

---

## 📥 Installation and Import

To start working with images in OpenCV, we first import the required libraries:

```python
import cv2
import numpy as np
import matplotlib.pyplot as plt
```

---

## 📖 Reading Images with `imread`

Images are read using the `cv2.imread()` function:

```python
img = cv2.imread("image.jpg")
```

### 🔢 Data Type (`dtype`)

When an image is loaded, its data type is:

```
uint8
```

This means:

* Values are **unsigned integers**
* Range is **0 to 255**
* No floating-point (decimal) values exist in standard images

Each image is stored as a **NumPy array (`numpy.ndarray`)**, which means we can use all NumPy operations directly on images.

---

## 🧮 Image as a NumPy Array

Since images are NumPy arrays:

```python
type(img)
# numpy.ndarray
```

We can:

* Slice images
* Modify pixels
* Apply mathematical operations
* Use NumPy functions directly

---

## 🎨 Color Space: BGR vs RGB

### ⚠️ Important Note

OpenCV reads images in **BGR format**, not RGB.

* Standard format in most libraries: **RGB**
* OpenCV format: **BGR**

So the red and blue channels are swapped in OpenCV.

---

## 🔄 Converting BGR to RGB

To display images correctly using Matplotlib, we must convert BGR → RGB:

```python
img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
plt.imshow(img_rgb)
plt.show()
```

---

## 📊 Matplotlib vs OpenCV

* **OpenCV** → uses BGR format
* **Matplotlib** → assumes RGB format

👉 If we do not convert, colors will appear incorrect.

---

## ⚫ Grayscale Images in Matplotlib

Matplotlib does not automatically interpret grayscale images correctly unless specified.

```python
plt.imshow(img_gray, cmap='gray')
```

---

## 📥 Reading Image in Grayscale

We can directly load an image in grayscale using:

```python
img_gray = cv2.imread("image.jpg", 0)
plt.imshow(img_gray, cmap='gray')
```

---

## ❓ RGB to Gray vs BGR to Gray

Yes, there is a difference.

Since we convert from **3 channels → 1 channel**, information is reduced.

### 📌 Standard Conversion Formula (Perceptual Weighting):


`Gray = (0.299R + 0.587G + 0.114B) / 3`

This is based on human visual perception:

* Green has the highest impact
* Blue has the lowest impact

⚠️ Note:
Simple averaging `(R + G + B) / 3` is not standard and does not reflect human vision correctly.

---

## 🚀 Key Insight

* Images are stored as numerical matrices (`uint8`)
* OpenCV uses **BGR format by default**
* Matplotlib expects **RGB format**
* Grayscale conversion is a **lossy transformation from 3 channels to 1**
* Correct grayscale depends on perceptual weighting, not simple averaging

