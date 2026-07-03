

# 🔢 Bit Depth in Digital Images

## 📌 What is Bit Depth?

Bit depth defines how many different intensity or color values each pixel in an image can represent. In simple terms, it determines how many levels of brightness or color are available per pixel.

Higher bit depth means:

* More detailed images
* Smoother color transitions
* Wider color range

---

## 🎚️ 8-bit Images

In 8-bit images:

[
2^8 = 256
]

So each pixel can take **256 different values**:

* 0 → black
* 255 → white
* Values in between → grayscale intensities

📌 This is the most common standard in image processing.

---

## 🧩 General Case (n-bit Images)

For an n-bit image:

[
2^n \text{ possible levels}
]

Examples:

* 4-bit → (2^4 = 16) levels (0–15)
* 10-bit → (2^{10} = 1024) levels
* 16-bit → (2^{16} = 65536) levels

---

## 🎨 Effect of Bit Depth on Image Quality

Higher bit depth leads to:

* Smoother gradients
* Better detail preservation
* Reduced banding artifacts

However:

* Increased memory usage
* Higher computational cost

---

## 🖥️ RGB Color Images

In standard RGB images:

* Each channel (R, G, B) has 8 bits

So:

[
8 \times 3 = 24 \text{ bits per pixel}
]

📌 This is known as a **24-bit image**.

---

## 🌈 Number of Colors in RGB Images

For a 24-bit image:

[
2^{24} = 16,777,216 \text{ colors}
]

📌 Approximately **16.7 million colors**.

---

## 🖥️ 10-bit Images

In professional imaging systems:

* Each channel = 10 bits

[
2^{10} = 1024 \text{ levels per channel}
]

Total colors:

[
1024 \times 1024 \times 1024 \approx 1 \text{ billion colors}
]

📌 Much smoother gradients compared to 8-bit images.

---

## 💾 Memory Usage of an Image

General formula for raw image memory:

[
Height \times Width \times Channels \times (Bit\ Depth / 8)
]

### 📌 Example:

For a 200 × 200 RGB 8-bit image:

[
200 \times 200 \times 3 \times 1 = 120{,}000 \text{ bytes}
]

≈ **120 KB**

---

## 🗜️ RAW vs Compressed Images

### 📁 Compressed formats (JPEG, PNG):

* Reduced file size
* Some information may be lost (lossy compression in JPEG)

### 🧠 RAW representation in memory:

* Fully uncompressed
* Stored as NumPy arrays in RAM
* Larger than disk file size

---

## 🧠 Image Loading in Python / OpenCV

When an image is loaded in OpenCV:

* It is read from disk (compressed format)
* Converted into an uncompressed NumPy array
* Stored in RAM

📌 Therefore, memory usage in RAM is often higher than file size on disk.

---

## ⚠️ Reducing Bit Depth

If bit depth is reduced (e.g., 8-bit → 4-bit):

* Memory usage decreases
* But:

  * Significant loss of detail
  * Reduced color accuracy
  * Visible artifacts and banding

📌 Therefore, 8-bit or higher is commonly used in most computer vision applications.

---

## 🚀 Summary

Bit depth is a fundamental concept in digital imaging that balances:

* Image quality
* Memory usage
* Computational cost

Higher bit depth provides better visual quality but requires more storage and processing power.
