

# 📌 Introduction to Image Structure and Types

## 🧠 Overview

Understanding image structure begins with how images are formed in the real world. When light hits an object, it reflects and is captured by the human eye or a camera sensor. This reflected light is then converted into a processable signal. Without light, no image is formed, and the scene appears completely dark.

## 📷 Image Formation in Cameras

In digital cameras, the same principle applies. Reflected light from objects is captured by the sensor and converted into numerical data, which is then stored in memory. If no light is present, pixel values become zero, resulting in a completely black image.

## 🧮 Image Representation as a Matrix

In image processing, an image is represented as a 2D or 3D matrix composed of small elements called **pixels**.

Each pixel represents the intensity of light at a specific location in the image. In other words, every element in the matrix holds a numerical value that describes brightness.

## 🔢 Pixel Intensity Range

Pixel values are typically in the range of **0 to 255**:

* **0** → Black (no light intensity)
* **255** → White (maximum light intensity)
* Values in between → Different shades of gray

## 🖼️ Types of Images

### 1. Binary Images

* Each pixel has only two possible values: `0` or `1`
* Commonly used for masks or simple object representation

### 2. Grayscale Images

* Each pixel has a value between 0 and 255
* Represents only intensity (no color information)

### 3. RGB Images

* Composed of three channels: **Red, Green, Blue**
* Each channel ranges from 0 to 255
* The combination of these channels defines the final color of each pixel

## 📐 Image Structure

* Binary and grayscale images:
  `(H, W)`

* RGB images:
  `(H, W, C)` where `C = 3`

## 🚀 Conclusion

In digital systems, an image is essentially a numerical matrix representing light intensity. This representation allows images to be processed, analyzed, and understood by computer vision algorithms and neural networks.

