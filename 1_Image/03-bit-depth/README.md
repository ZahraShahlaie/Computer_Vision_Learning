

#  Bit Depth in Digital Images

## 📌 What is Bit Depth?

Bit Depth defines how many different intensity or color values each pixel in an image can represent.

In simple terms:

> Bit depth = number of brightness or color levels per pixel.

The higher the bit depth, the more detailed and smoother the image will be.

---

## 🎚️ 8-bit Images

In an 8-bit image:

$$
2^8 = 256
$$

Each pixel can represent **256 different values**:

- 0 → black  
- 255 → white  
- Values in between → grayscale levels  

📌 This is the most common format in image processing.

---

## 🧩 General Case (n-bit Images)

If bit depth is **n**, then:

$$
2^n \text{ intensity levels}
$$

Examples:

- 4-bit → $2^4 = 16$ levels (0–15)  
- 10-bit → $2^{10} = 1024$ levels  
- 16-bit → $2^{16} = 65536$ levels  

---

## 🎨 Effect of Bit Depth on Image Quality

As bit depth increases:

- 🎯 Smoother transitions between colors  
- 🎯 Better detail preservation  
- 🎯 Reduced banding effects  

But:

- 💾 Higher memory usage  
- ⚙️ More computational cost  

---

## 🖥️ RGB Color Images

In standard RGB images:

- Each channel (R, G, B) = 8 bits  

So total bit depth per pixel:

$$
8 \times 3 = 24 \text{ bits}
$$

📌 This is called a **24-bit color image**

---

## 🌈 Number of Colors in RGB Images

For a 24-bit image:

$$
2^{24} = 16,777,216
$$

📌 ≈ 16.7 million colors

---

## 🖥️ 10-bit Images

For 10-bit per channel images:

- Each channel = 10 bits  

$$
2^{10} = 1024 \text{ levels}
$$

Total colors:

$$
1024 \times 1024 \times 1024 \approx 1 \text{ billion colors}
$$

---

## 💾 Image Memory Usage (Raw Size)

General formula:

```

Height × Width × Channels × num bits

```

### 📌 Example: 200×200 RGB image (8-bit)

```

200 × 200 × 3 × 1 byte = 120,000 bytes

```

≈ **120 KB (raw image size)**

---

## 📦 RAW vs Compressed Images

Images stored on disk (JPEG, PNG):

- Compressed format  
- Some data is lost or optimized  
- Smaller file size  

When loaded in Python (OpenCV):

- Image is decompressed  
- Stored as NumPy array in RAM  
- Memory usage is higher than disk size  

---

## 🧠 OpenCV Image Representation

When loading an image in OpenCV:

- Stored as a NumPy `ndarray`
- Data type is usually:

```

uint8

```

Meaning:

- Integer values only  
- Range: 0–255  
- No decimal values  

---

## ⚠️ Reducing Bit Depth

If bit depth is reduced (e.g., 8-bit → 4-bit):

- Memory usage decreases  
- But:
  - Image details are lost  
  - Quality decreases  
  - Banding artifacts appear  

📌 Therefore, most computer vision systems use **8-bit or higher**

