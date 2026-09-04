# YOLO Segmentation Tutorial — Image Segmentation


## Table of Contents

1. [What Is Segmentation? How It Differs from Detection](#1-what-is-segmentation-how-it-differs-from-detection)
2. [Understanding the Concept of a Mask](#2-understanding-the-concept-of-a-mask)
3. [Running Segmentation with Pretrained YOLO Models](#3-running-segmentation-with-pretrained-yolo-models)
4. [Highlighting a Mask on the Original Image](#4-highlighting-a-mask-on-the-original-image)
5. [Annotating Segmentation Data in Roboflow and Its File Format](#5-annotating-segmentation-data-in-roboflow-and-its-file-format)
6. [Training a Segmentation Model on Custom Data](#6-training-a-segmentation-model-on-custom-data)
7. [Using Ultralytics' Built-in Datasets](#7-using-ultralytics-built-in-datasets)

---

## 1. What Is Segmentation? How It Differs from Detection

A quick recap first: in **Classification**, we only say which class is present in the image (e.g. "cat"). In **Object Detection**, in addition to the class, we predict a **bounding box** (center, width, height) for each object. In **Object Tracking**, we do the same thing across a video's frames, assigning each object a persistent ID.

**Segmentation** is a different problem entirely: here, we're no longer looking for a rectangular box. Instead, the model has to classify **every single pixel** of the image. Pixels that share a common property (e.g. all belong to the same cat) are grouped into one "segment."

> In plain terms: Detection says "there's a cat here, roughly within this rectangle." Segmentation says "exactly these pixels, pixel by pixel, belong to the cat."

### A quick reminder on image structure

A color image with dimensions like 1080×1080 is actually a matrix with 3 color channels (Red, Green, Blue). In Segmentation, instead of predicting box coordinates, the model produces a matrix the same size as the input image, where each pixel is labeled with its class number — this matrix is called a **Mask** (explained fully in the next section).

### Two types of Segmentation

- **Semantic Segmentation:** only the **class** of each pixel matters, not which specific instance of that class it belongs to. If there are two different cats in the image, both get marked with the same color/class — no distinction is made between them.
- **Instance Segmentation:** in addition to the class, each **instance** is also identified separately. If there are two cats in the image, each is recognized as an independent object (with its own mask), even though they share the same class.

> **YOLO does Instance Segmentation.**

### YOLO's strengths and weaknesses for segmentation

YOLO was originally designed for **Object Detection**, not Segmentation; segmentation capability was added later (starting around version 7, with a notable improvement in version 8).

> **A useful analogy:** using YOLO for Segmentation is a bit like peeling a fruit with a large knife — it works fine for large fruit, but for fine details (like edges, or small objects) it may not have enough precision and can miss part of the object's real boundary.

If very high precision at the pixel/edge level is critical for your use case (especially for Semantic Segmentation), networks like **U-Net** are usually a more precise choice (especially on small datasets). But if a more general, faster-to-implement solution is your priority, YOLO is a solid choice.

---

## 2. Understanding the Concept of a Mask

A **Mask** is an image the same size as the original, where every pixel is either **zero** or **one** (a binary space):

- Value `1` (or white): this pixel is part of the object of interest.
- Value `0` (or black): this pixel is not part of the object (background, or a different object).

For every object we want the model to detect, we need a separate mask. For example, in a microscopy dataset of cells, each cell has its own mask file that marks exactly that one cell's boundary with `1`s, with everything else at `0`.

### Reading and displaying a mask in code

```python
import cv2
import matplotlib.pyplot as plt

image = cv2.imread("dataset/image.png")
image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)

mask = cv2.imread("dataset/test_mask/mask_1.png")
# Note: since a mask is purely black/white (0 or 255), converting
# BGR to RGB makes no visible difference and can be skipped.

plt.imshow(image)
plt.show()

plt.imshow(mask)
plt.show()
```

- In a regular color image, every pixel has 3 values (the R, G, B channel values).
- In a mask, since it's purely black or white, every pixel is either `(0, 0, 0)` or `(255, 255, 255)` — which is why channel ordering (BGR vs. RGB) doesn't affect how it displays.

### Displaying a mask highlighted on the original image

Often, instead of showing the black/white mask separately, we'd rather **highlight it directly on the original image with a chosen color** — exactly like what you see in well-known segmentation datasets (e.g. people in green, cars in blue, buildings in red). For this, we first define a color:

```python
color = (0, 255, 0)  # pure green, in (R, G, B) format
```

The exact procedure for blending this color with the original image (using `cv2.addWeighted`) is fully covered in Section 4.

---

## 3. Running Segmentation with Pretrained YOLO Models

### Loading a segmentation model

Ultralytics segmentation models are identified by the **`-seg`** suffix in the filename:

```python
from ultralytics import YOLO

model = YOLO("yolov8n-seg.pt")   # Nano version, specifically for segmentation
```

> This model is also trained on the COCO dataset and covers the same 80 common classes; the difference from the regular version (`yolov8n.pt`) is that its output includes a mask in addition to the bounding box.

### Running prediction

```python
results = model.predict(source="image_city.jpg")
result = results[0]
```

If you display this with `result.show()` or `results[0].plot()`, you'll see the model simultaneously draws the **bounding box**, **class**, **confidence score**, and the **colored mask region** on the image.

### Showing only the segmentation (no bounding boxes)

```python
plt.imshow(result.plot(boxes=False))
plt.show()
```

With `boxes=False`, you'll see only the colored mask regions, without the rectangular boxes around each object.

### Accessing boxes and masks

```python
result.boxes         # bounding box info (as before)
len(result.boxes)     # number of detected objects (e.g. 16)

result.masks          # mask info
len(result.masks)     # should match the number of objects (one mask per object)
```

Since every object has both a box and a mask, `len(result.boxes)` and `len(result.masks)` are always equal.

### Extracting a mask's raw data

```python
mask_data = result.masks[0].data   # a PyTorch tensor
mask_data.shape                     # something like: torch.Size([1, 384, 640])
```

- The first dimension (`1`) is the channel count — since a mask has only one channel (not 3 color channels like a regular image).
- The next two dimensions are the height and width of the image size the model operated on (e.g. 640, YOLO's standard input size).

For easier handling, convert this tensor to a NumPy array and rearrange its dimensions with `transpose` into the conventional image format (height, width, channel):

```python
import numpy as np

mask = mask_data.numpy()
mask = np.transpose(mask, (1, 2, 0))   # from (channel, height, width) to (height, width, channel)

mask.shape   # now something like (384, 640, 1)
```

---

## 4. Highlighting a Mask on the Original Image

To highlight a mask precisely on the original image, a few technical steps are needed.

### Step 1: matching the mask's size to the original image

The mask's output size (e.g. 384×640) usually doesn't match the actual input image's size. We resize the mask with `cv2.resize` to match the original image's dimensions:

```python
height, width, _ = image.shape

mask_resized = cv2.resize(mask, (width, height))
```

### An important gotcha: interpolation, and its fix

When resizing a binary matrix (only 0s and 1s), the underlying algorithm usually uses **linear interpolation** rather than producing exact 0s and 1s — meaning it averages neighboring pixels. As a result, some pixels end up with intermediate values (e.g. 0.3 or 0.6), not exactly zero or one.

**The fix:** define a threshold and convert any value above it to `1`, and everything else to `0`:

```python
mask_resized = (mask_resized > 0.5).astype(int)
```

### Step 2: converting the single-channel mask to a 3-channel image

Since the original image has 3 color channels but the mask has only one, we need to convert the mask to 3 channels before combining them:

```python
mask_scaled = cv2.convertScaleAbs(mask_resized, alpha=255)  # convert 0/1 values to 0/255
mask_3ch = cv2.cvtColor(mask_scaled, cv2.COLOR_GRAY2RGB)
```

> Note: trying `cv2.cvtColor(mask, cv2.COLOR_GRAY2RGB)` directly on raw `0/1` values usually errors out or produces incorrect results, since `cv2` expects standard 0–255 values. `cv2.convertScaleAbs` handles this scale conversion and also fixes any potential negative values.

### Step 3: coloring the mask

```python
color = (255, 0, 0)  # red
colored_mask = mask_3ch * np.array(color, dtype="uint8")
```

Multiplying the binary mask (now with values of `0` or `255`) by your chosen color turns every pixel that was part of the object into that color, while the rest stays black.

### Step 4: blending the colored mask with the original image

To blend the colored mask onto the original image with transparency, use `cv2.addWeighted`:

```python
final_image = cv2.addWeighted(image, 1, colored_mask, 0.5, 0)

plt.imshow(final_image)
plt.show()
```

- The second parameter (`1`) is the weight of the original image, and the fourth parameter (`0.5`) is the weight of the colored mask — adjust these numbers to increase or decrease transparency.

### Highlighting all masks in a loop

To highlight every detected object on a single image:

```python
mask_image = image.copy()

for i, mask_obj in enumerate(result.masks):
    mask = mask_obj.data.numpy()
    mask = np.transpose(mask, (1, 2, 0))
    mask_resized = cv2.resize(mask, (width, height))
    mask_resized = (mask_resized > 0.5).astype("uint8")

    mask_scaled = cv2.convertScaleAbs(mask_resized, alpha=255)
    mask_3ch = cv2.cvtColor(mask_scaled, cv2.COLOR_GRAY2RGB)
    colored_mask = mask_3ch * np.array(color, dtype="uint8")

    mask_image = cv2.addWeighted(mask_image, 1, colored_mask, 0.5, 0)

plt.imshow(mask_image)
plt.show()
```

Key point: on each loop iteration, the colored mask is added onto the **accumulated previous image** (`mask_image`), not the original raw image — so the final result includes the highlights of all objects together.

### Highlighting only a specific class (e.g. only people)

Within the same loop, before drawing each mask, you can check that object's class and only draw masks belonging to the class you want:

```python
model.names   # class dictionary, e.g. {0: 'person', 2: 'car', ...}

for i, mask_obj in enumerate(result.masks):
    class_id = int(result.boxes.cls[i])
    if class_id == 0:   # only the 'person' class
        # ... the same mask-drawing steps described above
        pass
```

This lets you highlight just one class of interest instead of every detected object.

---

## 5. Annotating Segmentation Data in Roboflow and Its File Format

To train a segmentation model on your own custom data, you need annotated data just like Object Detection — except here, instead of a bounding box, the annotation is a precise **polygon boundary** around each object.

### Steps for annotating in Roboflow

1. **Create New Project** → this time, choose **Instance Segmentation** as the project type.
2. Upload your images.
3. Go into the **Annotate** section and open an image.
4. To draw the boundary around an object, you have two options:
   - **Manual mode:** click point by point around the object's edge. The more points you place, and the more precise they are, the more accurate the final boundary will be.
   - **Smart mode:** the tool uses a built-in model to guess the object's boundary automatically — faster, but not necessarily as precise as manual mode; you can review and correct the result as needed.
5. Assign the correct class to each object (if there are multiple objects of the same class in the image, annotate each one separately — since this is Instance Segmentation).
6. Once all images are annotated, go to the **Generate** section, set the output image size for YOLO, and generate the dataset.
7. In the **Export Dataset** section, choose the **YOLOv8 (Segmentation)** format.

### The annotation file's structure (YOLO segmentation format)

Unlike Object Detection, where every annotation line has a fixed 5 numbers (`class x_center y_center width height`), in Segmentation each annotation line has a **variable number** of values:

```
0  0.45 0.32  0.50 0.28  0.55 0.40  0.48 0.55  0.40 0.48  ...
```

- **First number:** the class ID (as before).
- **Remaining numbers:** consecutive `x, y` pairs, each representing a **point on the polygon boundary** around the object — all normalized (between 0 and 1), exactly like what we saw in Detection annotation.
- The number of points can vary per object (depending on how many points were placed during annotation), but is always an **even** number (since each point has an `x` and a `y`).

### Reconstructing a mask from the annotation file (hands-on exercise)

To understand this format better, you can read the points from the text file and draw them directly onto a blank mask:

```python
import numpy as np
import cv2
import matplotlib.pyplot as plt

with open("dataset/train/labels/sample.txt", "r") as f:
    lines = f.readlines()

object_1 = lines[0].split()
class_id = object_1[0]
coords = object_1[1:]                 # all x, y values, as strings
coords = [float(c) for c in coords]    # convert to floats

img_size = 640   # the size the image was processed at

points = []
for i in range(0, len(coords), 2):
    x = int(coords[i] * img_size) - 1     # convert back to pixel scale
    y = int(coords[i + 1] * img_size) - 1
    points.append([x, y])

points = np.array(points)

# draw the polygon onto a blank mask
mask = np.zeros((img_size, img_size), dtype="uint8")
image_final = cv2.polylines(mask, [points], isClosed=True, color=255, thickness=2)

plt.imshow(image_final, cmap="gray")
plt.show()
```

Key points about this exercise:

- To iterate over the `x, y` pairs, a loop with a step of `2` is used (`range(0, len(coords), 2)`) — each iteration grabs one coordinate pair.
- The normalized value must first be converted to `float` (since it's read from a text file as a string), then multiplied by the image size (e.g. 640) to get real pixel coordinates.
- You may need to subtract 1 from the result (`- 1`) to avoid an "index out of range" error when a value lands exactly at the image size.
- `cv2.polylines` draws a closed polygon (`isClosed=True`) from a list of points — exactly the same boundary that was drawn point by point in Roboflow.

If you repeat this for every object in the file (not just the first), the reconstructed mask should exactly match what you annotated in Roboflow — a good sanity check for your annotation quality.

---

## 6. Training a Segmentation Model on Custom Data

The overall process is very similar to training an Object Detection model (covered in the main README); the key difference is the **model and dataset type**.

### Finding a ready-made dataset

If you don't want to annotate your own data from scratch, several sources offer ready-made segmentation datasets:

- **Roboflow Universe** — a large collection of publicly annotated datasets, including a dedicated segmentation section.
- **Kaggle**
- Other public dataset catalogs (e.g. datasets for detecting cracks in asphalt or concrete, medical imaging like MRIs, etc.).

> Important: any dataset you pick must have a **task type that matches your model type**. If the dataset is a segmentation dataset, the model must be a `-seg` variant; if the dataset is for detection, the model must be a regular detection model. Mismatching the two causes an error during training.

### Downloading the dataset in Colab

If your dataset is available as a ZIP link (e.g. from Roboflow Universe):

```bash
!wget "https://example.com/path/to/dataset.zip" -O dataset.zip
!unzip dataset.zip -d crack-seg
```

> Before running training, make sure the runtime is set to **GPU** (in Colab: Runtime → Change runtime type → GPU) so training runs noticeably faster.

### Running training

```python
!pip install ultralytics
from ultralytics import YOLO

model = YOLO("yolov8n-seg.pt")   # make sure it's the segmentation variant

model.train(
    data="/content/crack-seg/data.yaml",   # full, absolute path to data.yaml
    epochs=50,
    imgsz=640,
)
```

- Just like Object Detection, `data.yaml` must contain the `train`/`val` paths and class names.
- The same "use an absolute path" rule emphasized earlier applies here too, to avoid "file not found" errors.

### Reviewing training outputs

After training finishes, under a path like `runs/segment/train` you'll find:

- **`results.png`**: several loss plots (not just one overall loss) — including **segment loss** (mask accuracy error) and **class loss** (classification error), alongside other plots similar to Object Detection.
- **`weights/best.pt`**: the best weights obtained during training; this is the file you'll download and load for final use.

> Note: with a limited number of epochs and a small model (Nano), it's normal for the model to fall short of perfect results — for example, missing some objects entirely. More epochs, and if needed a larger model (e.g. `yolov8s-seg.pt` or higher), usually improves accuracy.

### Testing the trained model on a new image

```python
from ultralytics import YOLO
import cv2, matplotlib.pyplot as plt

model = YOLO("best.pt")
model.names   # the classes defined in your dataset

image_test = cv2.imread("dataset/test/images/sample.jpg")
image_test = cv2.cvtColor(image_test, cv2.COLOR_BGR2RGB)

plt.imshow(image_test)
plt.show()

results = model.predict(image_test)
plt.imshow(results[0].plot())
plt.show()
```

By comparing the raw image with the `plot()` output, you can see how many real objects the model correctly identified, and where it made mistakes or missed something — exactly the kind of analysis we did earlier for detection models.

---

## 7. Using Ultralytics' Built-in Datasets

An interesting detail: some well-known datasets featured in **Ultralytics' documentation** (e.g. `coco8-seg`, `cat-seg`, `brain-tumor`, etc.) don't need to be manually downloaded and unzipped. You can simply pass **that dataset's YAML filename** directly to the `train` function — Ultralytics handles the download, extraction, and setup itself.

```python
from ultralytics import YOLO

model = YOLO("yolov8n-seg.pt")

model.train(
    data="cat-seg.yaml",   # just the filename, no manual download needed
    epochs=50,
)
```

- You can try this approach for any dataset listed in Ultralytics' **Datasets** documentation section (whether segmentation or detection).
- When run, the library searches for, downloads, and extracts the dataset into a path like `content/datasets/cat-seg`, then proceeds directly into training.

### Critical note: matching model type to dataset type

If the dataset you reference doesn't match the model type, training will fail with an error:

```python
model = YOLO("yolov8n-seg.pt")     # a segmentation model

model.train(
    data="brain-tumor.yaml",        # but this dataset is a detection dataset!
    epochs=50,
)
# Result: the dataset downloads successfully, but fitting fails with an error,
# because the dataset's annotation format (bounding boxes) doesn't match
# the model's expected output format (masks).
```

> General rule: always confirm, before starting training, that the **dataset's task** (Detection / Segmentation / etc.) exactly matches the **type of model you loaded**. This shortcut only works for datasets listed in Ultralytics' official dataset catalog; for any other dataset (from Roboflow, Kaggle, or elsewhere), you'll need to follow the standard manual download-and-unzip process described in Section 6.

---

## Learning Path Summary for This README

```
What is Segmentation? Semantic vs. Instance, YOLO's strengths/weaknesses
        │
        ▼
Understanding the concept of a mask, viewing it with cv2/matplotlib
        │
        ▼
Running segmentation with pretrained models (the -seg suffix)
        │
        ▼
Highlighting a mask on the original image (resize, threshold, color, addWeighted)
        │
        ▼
Annotating custom data in Roboflow (polygons) and understanding its file format
        │
        ▼
Training a segmentation model on custom data and testing it
        │
        ▼
Using Ultralytics' built-in datasets by simply naming the YAML file
```

## Suggested Resources

- [Official Ultralytics Docs — Segmentation](https://docs.ultralytics.com/tasks/segment/)
- [Ultralytics Docs — Datasets section](https://docs.ultralytics.com/datasets/)
- [Roboflow Universe](https://universe.roboflow.com) for finding ready-made annotated datasets

---

> This README was compiled from lecture transcripts for personal review and documentation purposes. Always consult the official Ultralytics documentation for accurate, up-to-date technical details.
