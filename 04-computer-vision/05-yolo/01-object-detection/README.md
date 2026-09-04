# YOLO Tutorial — From Zero to Object Detection


## Table of Contents

1. [Core Concepts: Object, Object Detection, and Object Tracking](#1-core-concepts-object-object-detection-and-object-tracking)
2. [Installing Ultralytics and Setting Up the Environment](#2-installing-ultralytics-and-setting-up-the-environment)
3. [YOLO Input and Output](#3-yolo-input-and-output)
4. [Annotation and Coordinate Normalization](#4-annotation-and-coordinate-normalization)
5. [Drawing a Bounding Box with OpenCV](#5-drawing-a-bounding-box-with-opencv)
6. [Running Object Detection with Pretrained Models](#6-running-object-detection-with-pretrained-models)
7. [Evaluation Metrics: IoU, Precision, Recall, AP, and mAP](#7-evaluation-metrics-iou-precision-recall-ap-and-map)
8. [Preparing and Annotating a Custom Dataset with Roboflow](#8-preparing-and-annotating-a-custom-dataset-with-roboflow)
9. [Training YOLO on a Custom Dataset in Colab](#9-training-yolo-on-a-custom-dataset-in-colab)
10. [Using the Custom-Trained Model](#10-using-the-custom-trained-model)
11. [Validating the Custom Model](#11-validating-the-custom-model)
12. [Object Tracking Basics on Video](#12-object-tracking-basics-on-video)
13. [Reading from a Webcam with OpenCV](#13-reading-from-a-webcam-with-opencv)
14. [Real-Time Object Tracking on a Webcam](#14-real-time-object-tracking-on-a-webcam)
15. [Building a Custom Dataset with a Webcam and Training a Full Project](#15-building-a-custom-dataset-with-a-webcam-and-training-a-full-project)
16. [Introduction to the Object Counting Project](#16-introduction-to-the-object-counting-project)
17. [Solution 1: Using Ultralytics' Built-in ObjectCounter](#17-solution-1-using-ultralytics-built-in-objectcounter)
18. [Solution 2: Building a Custom Counter with cv2 and YOLO Track](#18-solution-2-building-a-custom-counter-with-cv2-and-yolo-track)

---

## 1. Core Concepts: Object, Object Detection, and Object Tracking

Before touching any code, it helps to have the right conceptual vocabulary for this part of Computer Vision.

- **Object:** anything present in an image — living (a person, an animal) or non-living (a car, a traffic light, a sign, a pothole, etc.). What counts as an "object" is entirely defined by the problem at hand; for one project it might only be "pothole" and "car".
- **Object Detection:** the task of solving two problems at the same time:
  1. **Localization** — where in the image the object is (its bounding box).
  2. **Classification** — what class the object belongs to (e.g. car, person, truck).
- **Object Tracking:** when, instead of a single frame, you have a video and want to follow an object's location across consecutive frames over time (e.g. a car moving across 10 seconds of footage).

> YOLO has historically been known first and foremost for Object Detection, though it supports other tasks as well.

### Confidence Score
A number the model assigns to each prediction, indicating how confident it is that an object exists at that location. By setting a threshold on this score (e.g. above 60%), you can decide which predictions to accept.

### Bounding Box
The box drawn around an object to mark its location in the image.

### Why doesn't the model detect everything?
Pretrained YOLO models (e.g. those trained on the COCO dataset) typically ship with around 80 predefined classes. If the object you care about (say, "pothole") isn't one of those classes, the model simply won't detect it — unless you define your own classes and fine-tune / train the model on your own data.

---

## 2. Installing Ultralytics and Setting Up the Environment

[Ultralytics](https://github.com/ultralytics/ultralytics) is a **high-level API** built on top of PyTorch. Instead of manually loading a model with raw PyTorch, running inference, and parsing the output, Ultralytics wraps all of that into a few simple lines of code.

### Installation

```bash
pip install ultralytics
```

- Ultralytics is built on PyTorch, but you don't need to install PyTorch separately — the package's `requirements` file is read automatically, and any missing dependency (including PyTorch) is installed for you.
- Recommended Python version: **Python ≥ 3.9 (3.11 is a good default)**.
- In Google Colab, just prefix the command with `!`:

```bash
!pip install ultralytics
```

### Installation notes

- **CPU vs. GPU:** by default, the CPU-only build is typically installed. To use a GPU, install the build matching your OS (Linux / macOS / Windows) with the correct CUDA drivers — assuming your hardware actually supports it.
- **Google Colab** is a practical, cost-effective way to get GPU access, especially if your local machine doesn't have a suitable GPU.
- **Always use a virtual environment.** When working with PyTorch alongside other frameworks or projects that need different dependency versions, installing everything into your system's global environment is a common source of dependency conflicts and hard-to-debug errors.

```bash
python -m venv yolo-env
source yolo-env/bin/activate      # Linux / macOS
yolo-env\Scripts\activate         # Windows
pip install ultralytics
```

### Loading your first model

```python
from ultralytics import YOLO

model = YOLO("yolov8n.pt")
```

- Note the capitalization: the class is `YOLO`, in uppercase.
- The `.pt` extension is PyTorch's native weight-file format.
- On first run, the weight file is downloaded automatically. If your connection is unstable, the download may fail — simply re-run the cell, or try a different model size (e.g. `yolov8s.pt`).

---

## 3. YOLO Input and Output

- **Input:** an image.
- **Output:** contrary to a common misconception, the model does **not** generate a new image. What it actually returns is a **numeric vector** containing:
  1. The predicted **class**
  2. The **bounding box center** coordinates (`x_center`, `y_center`)
  3. The **width** and **height** of the box

In other words, for every detected object the model tells you: *"this class, centered here, with this width and height."*

---

## 4. Annotation and Coordinate Normalization

**Annotation** means recording the ground-truth data for every bounding box in an image: class + center + width + height. This is usually stored in a plain text file (`.txt`) with the same name as the image; each line corresponds to one object.

Example of a single YOLO-format annotation line:

```
0  0.55  0.42  0.30  0.45
```

Read in order as: `class_id  x_center  y_center  width  height`

### Why are the values decimals? (Normalization)

Coordinates are **normalized** relative to the image dimensions so they're independent of the image's actual size:

```
x_normalized = x_pixel / image_width
y_normalized = y_pixel / image_height
```

This keeps every value between 0 and 1, regardless of whether the image is 600×400 pixels or any other resolution.

> To convert back to real pixel coordinates (e.g. to draw the box with OpenCV), reverse the operation: multiply the normalized value by the image's actual width or height.

---

## 5. Drawing a Bounding Box with OpenCV

Before touching YOLO itself, it's a good exercise to manually draw an existing annotation onto its image using `cv2`, to make the data format concrete.

```python
import cv2
import matplotlib.pyplot as plt

# 1) Read the image (note: OpenCV reads in BGR by default, not RGB)
image = cv2.imread("accident.jpg")
image = cv2.cvtColor(image, cv2.COLOR_BGR2RGB)

# 2) Read the annotation file
with open("accident.txt", "r") as f:
    lines = f.readlines()

# 3) Real pixel dimensions, needed to un-normalize the coordinates
HP, WP, _ = image.shape  # pixel height, pixel width

for line in lines:
    data = line.split()
    class_id = int(data[0])
    x_center, y_center, w, h = map(float, data[1:])

    # Convert back to pixel scale
    x_center *= WP
    y_center *= HP
    w *= WP
    h *= HP

    # Compute top-left and bottom-right corners
    x_min = int(x_center - w / 2)
    y_min = int(y_center - h / 2)
    x_max = int(x_center + w / 2)
    y_max = int(y_center + h / 2)

    cv2.rectangle(image, (x_min, y_min), (x_max, y_max), (255, 0, 0), 2)

plt.imshow(image)
plt.axis("off")
plt.show()
```

Key points:
- `cv2.imread` loads images as **BGR**; convert to RGB before displaying with `matplotlib` (`cv2.cvtColor(..., cv2.COLOR_BGR2RGB)`).
- `cv2.rectangle` expects two points — **top-left** and **bottom-right** corners — not a center + width/height, so a conversion step is required.
- Box coordinates must be **integers**, since pixels are discrete units and fractional pixel positions have no meaning.

---

## 6. Running Object Detection with Pretrained Models

### YOLOv8 model variants

On Ultralytics' Detection documentation page, pretrained models (trained on the COCO dataset) are listed with the following naming scheme:

| Model | Full name | Approx. Params | Speed | Accuracy (mAP) |
|---|---|---|---|---|
| YOLOv8**n** | Nano | ~3M | Fastest | Lower (~37) |
| YOLOv8**s** | Small | ~11M | Fast | Moderate |
| YOLOv8**m** | Medium | — | Medium | Moderate-high |
| YOLOv8**l** | Large | — | Slower | High |
| YOLOv8**x** | X-Large | ~68M | Slowest | Highest (~53) |

> These figures are approximate and given purely for relative comparison — always check Ultralytics' official docs for exact current numbers.

### The speed vs. accuracy trade-off

Moving from Nano toward X-Large:
- Parameter count (and model size) grows substantially.
- Inference time increases, and stronger hardware is required.
- In exchange, accuracy (measured by mAP) improves.

**Rule of thumb:** limited hardware or a real-time requirement → smaller models (n or s). A powerful GPU and accuracy as the priority → larger models (l or x).

### Inspecting model info

```python
model.info()
```

Shows the number of layers and parameters (the numbers on the docs' comparison table are usually rounded).

### Running prediction

```python
results = model.predict(source=["car.png", "tehran.jpg"])
```

- `source` can be a path to one or more images. You don't need to pass a NumPy array yourself — Ultralytics handles all preprocessing (resizing, converting to array, etc.) internally.
- `results` is a **list**; one entry per input image.

```python
len(results)          # number of input images
first_image = results[0]  # predictions for the first image
```

### Saving results

```python
results = model.predict(source="car.png", save=True)
```

With `save=True`, the annotated image (with drawn bounding boxes) is saved under `runs/detect/predict`. You can also save with a custom name:

```python
results[0].save(filename="car_detect.png")
```

### Accessing class names

```python
results[0].names   # dictionary of dataset class labels (e.g. COCO)
```

### Comparing model sizes (observations from the lecture)

Running the same image through Nano/Small vs. the X model showed that:
- Larger models tend to produce **higher confidence scores** for correct predictions.
- The number and type of detected objects can differ between model sizes (e.g. one model calling an object a "truck" while another calls it a "car", or one model missing a traffic light that the other catches).
- Inference time increases noticeably as model size grows.

### YOLOv9 vs. YOLOv8

At the time of this lecture, YOLOv9 had just been released and its documentation was still incomplete (only Detection and Segmentation were available). Even so, an early comparison showed YOLOv9 reaching a **higher mAP with fewer parameters** than its YOLOv8 counterpart — i.e. faster and more accurate at once. Since this space moves quickly, always check the latest official documentation for up-to-date figures.

---

## 7. Evaluation Metrics: IoU, Precision, Recall, AP, and mAP

Throughout this course, YOLO is treated as a **black box** — internal architecture (number of convolution layers, etc.) is out of scope. What matters is understanding how the model computes **loss** and how it's **evaluated**.

### Loss in Object Detection

Object Detection requires solving two problems simultaneously:
1. **Localization** (bounding box)
2. **Classification** (object class)

The loss reflects both:
- For **classification**: typically **Cross Entropy**.
- For the **box**: a metric called **IoU**.

### IoU (Intersection over Union)

$$
IoU = \frac{\text{Area of overlap between the ground-truth box and the predicted box}}{\text{Area of union of the two boxes}}
$$

- If the ground-truth and predicted boxes don't overlap at all → `IoU = 0`.
- The closer the prediction is to the ground truth → `IoU` approaches `1`.
- **Important:** IoU only measures how good the **box location** is — it has nothing to do with the predicted class. Even if the class is wrong, IoU is still computed purely from spatial overlap.

### Confidence Score

A number the model attaches to each predicted box during inference, reflecting how confident it is that an object exists there. Changing the confidence threshold changes how many predictions get accepted.

### True Positive / False Positive / False Negative

- **True Positive (TP):** a correct prediction — good IoU and correct class.
- **False Positive (FP):** an incorrect prediction — the model detected something that wasn't actually there (or mislabeled/mislocated it).
- **False Negative (FN):** an object that genuinely existed but the model failed to detect (missed).

### Precision and Recall

$$
Precision = \frac{TP}{TP + FP}
$$

> Of everything the model predicted, what fraction was actually correct?

$$
Recall = \frac{TP}{TP + FN}
$$

> Of everything that should have been detected, what fraction did the model actually find?

**Example:** if 10 objects should have been detected, and the model correctly found 8 (TP=8), missed 2 (FN=2), and produced zero wrong predictions (FP=0):
- `Precision = 8 / (8+0) = 1.0`
- `Recall = 8 / (8+2) = 0.8`

Important point: **high recall and low precision can happen at the same time.** If a model predicts objects overly liberally (guessing everywhere), it will likely catch nearly every real object (high recall), but it will also generate a lot of wrong predictions (low precision).

### The Precision-Recall / confidence-threshold trade-off

Changing the confidence threshold creates a **trade-off** between precision and recall:
- Very low threshold → recall goes up (almost nothing is missed) but precision drops (many false alarms).
- Very high threshold → precision goes up (only very confident predictions are accepted) but recall drops (many real objects get missed).

The **Precision-Recall Curve (PR curve)** plots this relationship across different confidence thresholds.

### AP (Average Precision)

**AP** is the **area under the PR curve**. The closer this area is to `1`, the more ideal the model's behavior for that class (both high precision and high recall across most thresholds).

### mAP (mean Average Precision)

For **multi-class** problems, a separate PR curve and AP value is computed per class. mAP is then the average of those AP values across all classes:

$$
mAP = \frac{1}{N_{classes}} \sum_{i=1}^{N_{classes}} AP_i
$$

- Higher `mAP` = a stronger, more reliable model.
- For a **single-class** problem, `mAP` is identical to `AP` (since averaging a single number is meaningless).

### A note on IoU threshold

Besides the confidence threshold, you can also set an **IoU threshold** (e.g. only counting predictions with IoU above 0.5 or 0.8 as valid). This threshold directly affects recall too, and is explored in more depth in several papers on the topic (e.g. a 2018 reference paper mentioned in this lecture). A related metric, **AR (Average Recall)** and its multi-class counterpart **mAR**, also exists, though standard YOLO evaluation usually focuses on mAP.

---

## 8. Preparing and Annotating a Custom Dataset with Roboflow

Pretrained YOLO models only recognize around 80 classes (the COCO classes). To solve a specific, custom problem (e.g. counting cars on a road, or detecting an object that isn't among those 80 classes), you need to **annotate your own data and fine-tune / train the model on it**.

### Annotation tools

A few well-known tools for annotating images:
- **Roboflow** — very widely used, with a simple, friendly interface.
- **CVAT** — one of the most popular open-source annotation tools.
- Many other tools exist and are easy to find with a quick search.

> Training requires a GPU, so the following steps move over to **Google Colab**.

### Working with Roboflow

1. **Sign up / sign in** to Roboflow.
2. **Create New Project** → choose **Object Detection** as the project type, and give the project and class group a name (e.g. `car detection` and class `car`).
   > Note: on the free tier, Roboflow projects are **public** — others can access them too.
3. **Upload images** — drag and drop your raw dataset images (e.g. downloaded from Kaggle).
4. **Draw bounding boxes** — click into each image, select the rectangle tool, draw a box around each object, and assign it the correct class. Tips:
   - Don't draw boxes too tight (cropping part of the object) or too loose (lots of empty space inside the box) — try to match the object's actual boundary as closely as possible.
   - If you have multiple classes, pick the right class for each object from the list.
5. **Save and Continue** → split the data into **Train / Validation / Test**. You can set the percentages manually (for small datasets, make sure at least one image goes to validation so it isn't left empty).
6. **Processing** — options like resizing, changing orientation, and cropping.
7. **Augmentation** — very useful when data is limited. It generates varied versions of each image (horizontal/vertical flip, brightness changes, blur, zoom in/out), effectively multiplying your dataset size.
8. **Export Dataset** — set the output format to **YOLOv8** (or YOLOv9 — the annotation format is identical). Two ways to get the data:
   - **Download zip to computer** — a direct ZIP download.
   - **Show Download Code** — generates a Python snippet that downloads the dataset directly inside Colab (recommended and faster when working in Colab).

```python
# Example of the code Roboflow generates (replace with your actual project values)
!pip install roboflow

from roboflow import Roboflow
rf = Roboflow(api_key="YOUR_API_KEY")
project = rf.workspace("YOUR_WORKSPACE").project("YOUR_PROJECT")
dataset = project.version(1).download("yolov8")
```

> **Order of execution in Colab matters:** run the Roboflow snippet first to download the dataset, *then* install `ultralytics` and import `YOLO`.

After downloading, the dataset structure includes `train` and `valid` folders, each containing `images` and `labels` subfolders.

---

## 9. Training YOLO on a Custom Dataset in Colab

### The data.yaml file

Alongside the dataset downloaded from Roboflow, you'll find a **`.yaml`** file (YAML format, commonly used for configuration). It's a set of **key: value** pairs holding the paths to the train and validation data, plus the class names:

```yaml
train: ../train/images
val: ../valid/images
nc: 1
names: ['car']
```

Instead of manually specifying train/validation paths separately, you simply point the training function at this `data.yaml` file — Ultralytics extracts everything it needs from it.

### Running training

```python
from ultralytics import YOLO

model = YOLO("yolov8n.pt")  # start from pretrained weights (Nano, for faster training)

model.train(
    data="/content/car-detection-1/data.yaml",  # full path to data.yaml
    epochs=50,
    batch=32,
    imgsz=640,
    optimizer="Adam",
    lr0=0.001,
)
```

### Common issues and notes

- **Relative paths can break things.** Always use the **full, absolute path** to `data.yaml` to avoid "file not found" errors caused by the working directory changing between runs.
- Colab may sometimes show a **warning** on first run; in most cases it's safe to click **Cancel** and let execution continue — no harm is done.
- Common training parameters:
  - `data`: path to `data.yaml`
  - `epochs`: number of training epochs (e.g. 50)
  - `batch`: batch size (reduce this if you run out of GPU memory/VRAM)
  - `imgsz`: input image size (default 640)
  - `optimizer`: e.g. `Adam`
  - `lr0`: initial learning rate — typically a small value like `0.001`

### What you'll see during training

For each epoch, you'll see:
- **GPU memory** usage
- Live **batch loss** values

### Training outputs

After training finishes, results are saved under a path like `runs/detect/train` (or with an incrementing number, e.g. `train4`, on subsequent runs):

- **`weights/best.pt`** — the best weights achieved across the *entire* training run (not necessarily the last epoch). This is usually the file you download and use going forward.
- **`weights/last.pt`** — the weights from the final epoch.
- **`results.png`** — a plot of loss over epochs for both train and validation. If loss is still trending downward and hasn't plateaued, that's a good sign more epochs could improve the result further.

```python
# Reload the trained model later for use
model = YOLO("best.pt")
```

---

## 10. Using the Custom-Trained Model

### Loading the model and inspecting classes

```python
from ultralytics import YOLO

model = YOLO("best.pt")   # our custom-trained model
model.names
```

Unlike the default YOLOv8 model with 80 classes, our custom model only has the class(es) we defined in Roboflow (e.g. just `{0: 'car'}`).

### Running prediction

```python
image_path = "dataset/valid/images/sample.jpg"
results = model.predict(source=image_path)
```

### Displaying the original image vs. the annotated image

```python
import matplotlib.pyplot as plt
import cv2

result = results[0]

# Original image (result.orig_img is in BGR)
plt.imshow(cv2.cvtColor(result.orig_img, cv2.COLOR_BGR2RGB))
plt.axis("off")
plt.show()

# Image with bounding boxes and confidence scores drawn
plt.imshow(cv2.cvtColor(result.plot(), cv2.COLOR_BGR2RGB))
plt.axis("off")
plt.show()
```

- `result.orig_img` returns the raw image matrix (in BGR, since OpenCV reads that way).
- `result.plot()` returns the image **with bounding boxes and confidence scores drawn on it**.
- Both need `cv2.cvtColor(..., cv2.COLOR_BGR2RGB)` conversion for colors to display correctly in `matplotlib`.

### Filtering with a confidence threshold

If the model makes a weak, low-confidence prediction (e.g. 33%) on something that isn't actually there, you can suppress it by setting a confidence threshold:

```python
results = model.predict(source=image_path, conf=0.5)  # keep only predictions above 50% confidence
```

### Accessing per-box details

```python
result.boxes            # all boxes detected in this image
result.boxes.conf       # list of confidence scores
len(result.boxes)       # number of detected objects

result.boxes.xyxy       # box coordinates as [x_min, y_min, x_max, y_max] (real pixels)
result.boxes.xyxyn      # same, but normalized (0 to 1)
result.boxes.xywh       # box coordinates as [x_center, y_center, width, height] (real pixels)
result.boxes.xywhn      # same, but normalized
```

This is the same information we previously computed manually with `cv2.rectangle`; here, Ultralytics gives us both the pixel-scale and normalized versions directly.

---

## 11. Validating the Custom Model

Once training is done, the model needs to be **validated** to see how much you can trust it and where its strengths/weaknesses lie.

### Running validation

```python
model = YOLO("best.pt")

validation_results = model.val(data="/content/car-detection-1/data.yaml")
```

> Again: **use the full, absolute path** to `data.yaml`, otherwise you'll get a "path not found" error, since a relative path depends on the current working directory.

Validation outputs and plots are saved under a path like `runs/detect/val` (or `val2`, `val3`, etc.).

### Key metrics in the validation output

```python
validation_results.box.map50     # mAP at IoU threshold = 0.5
validation_results.box.map75     # mAP at IoU threshold = 0.75
validation_results.box.map       # mAP averaged over IoU thresholds from 0.5 to 0.95
```

- **mAP50:** the most common and "easiest" metric — IoU threshold fixed at 0.5.
- **mAP75:** a stricter metric — IoU threshold at 0.75. A high score here reflects very precise bounding boxes.
- **mAP50-95:** the average mAP across a range of IoU thresholds (0.5 to 0.95 in fixed steps) — a more comprehensive, stricter overall metric than mAP50 alone.

> Which metric matters most depends on your problem: if exact box precision isn't critical, mAP50 is enough. If boxes need to be very precise (e.g. for sensitive measurements), pay closer attention to mAP75 or mAP50-95.

### Curves and other analysis tools

```python
validation_results.curves        # a list of several plots (e.g. PR curve, ROC/confidence curve, etc.)
```

- **PR Curve (Precision-Recall Curve):** the closer this curve hugs the top-right corner (both precision and recall high), the better the model.
- **Confidence curve (ROC-like):** shows how recall changes across different confidence thresholds.
- **Confusion Matrix:** available in normalized form too (e.g. "95% true positive").

### Testing different confidence and IoU values

You can explicitly set confidence and IoU thresholds during validation to see their effect on the results:

```python
validation_results = model.val(
    data="/content/car-detection-1/data.yaml",
    conf=0.5,   # confidence threshold
    iou=0.6,    # IoU threshold used for NMS/evaluation
)
```

- Raising the confidence threshold generally **increases precision** but can **decrease recall**.
- Changing the IoU threshold also affects this trade-off; use the precision-recall plots to find the best balance for your specific problem.
- As expected, evaluation naturally gets more complex — and results often dip a bit — as the number of classes and data diversity grow, compared to a simple single-class problem.

---

## 12. Object Tracking Basics on Video

Everything so far has worked on **a single, static image** (Object Detection). But when we have a **video** instead, every moment is a new **frame** — essentially another static image. If we want to **follow** an object across consecutive frames (rather than just detecting it independently in each frame), we're now talking about **Object Tracking**.

> Key difference between Tracking and plain Detection: in Tracking, the model needs to recognize that the object seen in the current frame is **the same object** it saw in the previous frame — not a brand-new, unrelated detection.

### Using the `track` method

Instead of `model.predict(...)`, which is used for a still image, video uses the **`track`** method:

```python
from ultralytics import YOLO

model = YOLO("yolov8s.pt")

source = "sample_video.mp4"

results = model.track(source=source, show=True)
```

- `show=True` displays the output (video + bounding boxes) live on screen as it's processed.
- To **save** the output instead of (or alongside) live display:

```python
results = model.track(source=source, save=True, show=False)
```

With `save=True`, the output video (with tracked bounding boxes) is saved under a path like `runs/detect/track` (or with an incrementing number, e.g. `track2`, on subsequent runs).

> Practical note: smaller models (like `yolov8n.pt`) process video faster; larger models (like `yolov8s.pt`) are more accurate but slower — the same speed/accuracy trade-off we saw earlier still applies here.

---

## 13. Reading from a Webcam with OpenCV

Before moving on to real-time tracking with a webcam, we need to know how to access a camera with OpenCV and read its frames.

### Opening a video source

```python
import cv2

cap = cv2.VideoCapture("sample_video.mp4")   # can also be a video file path
```

- `cap.isOpened()` returns a boolean indicating whether the capture connection opened successfully.
- `cap.release()` frees the resource — important, since every open capture holds onto system memory. After `release()`, `cap.isOpened()` no longer returns `True`.
- `cap.read()` returns a **two-item tuple**: the first value (`ret`) is a boolean indicating whether the read succeeded, and the second value is the **frame** (image) itself.

```python
ret, frame = cap.read()

import matplotlib.pyplot as plt
plt.imshow(frame)   # note: colors are in BGR
plt.show()
```

### Finding the correct camera index

When multiple webcams are connected to a system, you need to figure out which **index** (`0`, `1`, `2`, ...) corresponds to which camera. A simple approach is to loop through the first several indices:

```python
def get_camera_detail(index):
    cap = cv2.VideoCapture(index)
    if cap.isOpened():
        height = cap.get(cv2.CAP_PROP_FRAME_HEIGHT)
        width = cap.get(cv2.CAP_PROP_FRAME_WIDTH)
        cap.release()
        return index, height, width
    return index, None, None


for i in range(10):
    index, h, w = get_camera_detail(i)
    print(f"camera:\n  index: {index}\n  height: {h}\n  width: {w}")
```

Any index that's actually connected to a real camera returns real `height`/`width` values; indices with nothing attached return `None`. This lets you confirm, for example, whether index `0` is your laptop's built-in webcam or an external, higher-quality camera.

### Live camera display loop

```python
cap = cv2.VideoCapture(0)  # once you know the correct index

while cap.isOpened():
    ret, frame = cap.read()

    cv2.imshow("camera", frame)

    if cv2.waitKey(1) & 0xFF == ord("q"):
        break

cap.release()
cv2.destroyAllWindows()
```

Key points:
- `cv2.waitKey(1)` waits 1 millisecond each loop iteration, checking whether a key has been pressed. This short delay lets the loop continue while still allowing keypresses to be detected.
- The condition `== ord("q")` checks whether the pressed key was **Q**; if so, we `break` out of the loop.
- After exiting the loop, always call `cap.release()` (free the camera) and `cv2.destroyAllWindows()` (close any open windows) to release system resources.

---

## 14. Real-Time Object Tracking on a Webcam

Now that we know how to read frames from a webcam, we can run YOLO inside that same loop.

### Why isn't `predict` enough?

If we call `model.predict(frame)` inside the loop, the model makes predictions on **each frame completely independently** — there's no connection between objects detected in the current frame and the previous one. That's just **frame-by-frame detection**, not real tracking.

```python
from ultralytics import YOLO
import cv2

model = YOLO("yolov8n.pt")
cap = cv2.VideoCapture(0)

while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        result = model.predict(frame)[0]
        cv2.imshow("camera", result.plot())
        if cv2.waitKey(1) & 0xFF == ord("q"):
            break

cap.release()
cv2.destroyAllWindows()
```

### Enabling real tracking with `persist=True`

To tell the model that the frames we're feeding it one by one **are related** (and that object identity should be preserved across frames), we need to use `track` instead of `predict`, with the **`persist=True`** parameter:

```python
result = model.track(frame, persist=True)[0]
```

- `persist=True` means: "don't treat these frames as unrelated; keep the tracking state alive across consecutive calls."
- Without this parameter, even when calling `track`, the model effectively processes each frame in isolation and object IDs won't stay consistent.

### Restricting to specific classes

Both `predict` and `track` accept a **`classes`** parameter that constrains the model to only look for certain classes (instead of all 80 in the default model):

```python
model.names   # view the full class dictionary and their indices, e.g. {0: 'person', ...}

result = model.track(frame, persist=True, classes=[0, 39])[0]  # only classes 0 and 39
```

### Full real-time tracking loop

```python
from ultralytics import YOLO
import cv2

model = YOLO("yolov8n.pt")
cap = cv2.VideoCapture(0)

while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        result = model.track(frame, persist=True, classes=[0])[0]
        annotated_frame = result.plot()
        cv2.imshow("camera", annotated_frame)

        if cv2.waitKey(1) & 0xFF == ord("q"):
            break

cap.release()
cv2.destroyAllWindows()
```

---

## 15. Building a Custom Dataset with a Webcam and Training a Full Project

This section walks through the complete lifecycle of a custom Object Detection/Tracking project — from **data collection** to **training and final use** — using nothing but a webcam. The same pattern applies to virtually any problem you can think of (detecting a product on a production line, monitoring a piece of machinery's state, hand-gesture-based interactive apps, counting livestock on a farm, etc.).

### Step 1: Collecting images with the webcam

Instead of running prediction on every frame, at this stage we simply display the live feed and, when a specific key is pressed (e.g. **S**), save the current frame to disk:

```python
import cv2
import os

os.makedirs("dataset/custom", exist_ok=True)

cap = cv2.VideoCapture(0)
i = 1

while cap.isOpened():
    ret, frame = cap.read()
    if ret:
        cv2.imshow("camera", frame)

        key = cv2.waitKey(1) & 0xFF
        if key == ord("s"):
            filename = f"dataset/custom/image_{i}.jpg"
            cv2.imwrite(filename, frame)
            i += 1
        elif key == ord("q"):
            break

cap.release()
cv2.destroyAllWindows()
```

Notes:
- `cv2.imwrite(filename, frame)` saves the current frame to disk under the given name.
- The counter variable (`i`) increments with every save so filenames don't collide (`image_1.jpg`, `image_2.jpg`, ...).
- **Class balance matters.** If you're collecting several classes/states, try to gather roughly equal numbers of images per class; imbalanced data biases the model toward the more frequent class.
- For more variety, capture images at different angles, distances, and lighting conditions.

### Step 2: Uploading and annotating in Roboflow

- Create a new Roboflow project and define your target classes (e.g. several classes matching your specific problem).
- Upload the collected images (you don't need to include all of them — for an initial test, a subset, e.g. 50 images, is enough to keep annotation fast).
- Draw a bounding box on each image and assign it the right class.
- Keep an eye on **class balance** while annotating; if one class is clearly under-represented, gather or select more images for it.
- In the Augmentation step, options like rotation, horizontal/vertical flip, brightness adjustment, shear, and more are available — the same as we saw earlier in Section 8.

### Step 3: Training the model in Colab

The process is identical to what was described in Sections 8 and 9:

```python
!pip install roboflow
from roboflow import Roboflow

rf = Roboflow(api_key="YOUR_API_KEY")
project = rf.workspace("YOUR_WORKSPACE").project("YOUR_PROJECT")
dataset = project.version(1).download("yolov8")
```

```python
from ultralytics import YOLO

model = YOLO("yolov8n.pt")

model.train(
    data="/content/your-project-1/data.yaml",  # full, absolute path
    epochs=50,
    imgsz=640,
)
```

> Common pitfall: if `data.yaml`'s path isn't given as a full absolute path, you'll hit a "file not found" error — the exact same issue covered in Section 9.

Once training finishes, as before, check the `results.png` plot to see the loss trend (both train and validation) and judge whether the model is still improving or has plateaued.

### Step 4: Using the final model — why `track`, not `predict`?

Once our custom multi-class model is trained, using it for a live application (e.g. on webcam input) calls for **`track`**, not `predict`. Here's why:

> The goal is to **follow** an object over time and preserve its identity across consecutive frames — not to predict each frame from scratch, disconnected from the last. With `track` (and `persist=True`), the model "remembers" that the object it saw in the previous frame is still the same object, rather than treating it as a freshly discovered one.

```python
model = YOLO("best.pt")   # our custom multi-class model

result = model.track(frame, persist=True)[0]
```

With this same pattern — **collect data via webcam → annotate in Roboflow → train in Colab → deploy with `track`** — you can implement virtually any custom Object Detection/Tracking project you can think of.

---

## 16. Introduction to the Object Counting Project

A very common application of Object Tracking is **counting how many objects pass** a given point or line in the frame — e.g. counting cars passing along a road, or counting people entering/exiting a warehouse. This has uses across many domains:

- **Urban traffic analysis:** counting vehicles over time to predict peak traffic hours.
- **Warehouse/store foot traffic:** counting people or goods moving in and out.
- Any other problem where you need to count how many times an object crosses a defined region.

### Two implementation approaches

There are two different paths for this project, both covered below:

1. **High-level approach:** using Ultralytics' ready-made `ObjectCounter` tool that ships alongside the YOLO models.
   - **Pro:** much shorter code, faster to implement.
   - **Con:** since the internal logic is a black box, there's limited ability to inspect or fine-tune exactly how it behaves — you can only adjust the parameters the package exposes.
2. **Low-level (custom) approach:** combining `model.track()` output directly with OpenCV to write the counting logic yourself, from scratch.
   - **Pro:** full control over the logic, and the ability to debug precisely.
   - **Con:** more code to write, and you need to understand the model's raw output details (e.g. `boxes.id`, `boxes.xywh`).

> A more general takeaway: this trade-off between high-level APIs and low-level code shows up across many machine learning libraries. A high-level API usually bundles several steps into one function call to speed up development, but at the cost of reduced ability to customize behavior.

---

## 17. Solution 1: Using Ultralytics' Built-in ObjectCounter

### Imports and initial setup

```python
from ultralytics import YOLO
from ultralytics.solutions import ObjectCounter
import cv2

model = YOLO("yolov8n.pt")

video_path = "dataset/sample_video.mp4"
cap = cv2.VideoCapture(video_path)

print(cap.isOpened())  # confirm the video opened successfully
```

### Defining the counting line

We need to decide exactly where in the frame counting should happen. Some guidance for placing this line:

- Don't place it too close to the top edge (where objects have just entered the frame) — they may not be fully detected yet.
- Don't place it too close to the bottom edge either, for the same reason in reverse.
- A reasonably central, stable pixel height is usually a good choice.

```python
ret, frame = cap.read()

line_points = [(20, 600), (1200, 600)]  # two endpoints, at a fixed pixel height
```

To confirm the line is positioned correctly, draw it on a sample frame and visualize it:

```python
import matplotlib.pyplot as plt

image = cv2.line(frame.copy(), line_points[0], line_points[1], color=(255, 0, 0), thickness=6)
plt.imshow(cv2.cvtColor(image, cv2.COLOR_BGR2RGB))
plt.show()
```

### Creating the ObjectCounter

```python
counter = ObjectCounter(
    view_img=False,          # if True, the package opens its own display window
    reg_pts=line_points,      # the counting line's endpoints
    classes_names=model.names,
    draw_tracks=True,         # show each object's movement trail
    view_in_count=True,
    view_out_count=True,
)
```

> Note: your installed Ultralytics version may name these parameters slightly differently (e.g. `line_points` instead of `reg_pts` in some versions) — always check your specific version's documentation.

### The main processing loop

```python
while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break

    tracks = model.track(frame, persist=True)
    frame = counter.start_counting(frame, tracks)

    cv2.imshow("counter", frame)
    if cv2.waitKey(1) & 0xFF == ord("q"):
        break

cap.release()
cv2.destroyAllWindows()
```

- Instead of feeding the model's output directly into a plain predict call, we first run `model.track(frame, persist=True)` to get tracking results, then pass that result into `counter.start_counting(frame, tracks)`.
- If you don't need to save the output video (just want to view it live), skip setting up a `VideoWriter` — simply display the result with `cv2.imshow`.
- To close the display window, use `cv2.waitKey` and check for the **Q** key, as before.

### Limitations of this approach

In practice, this tool's counting **isn't always accurate** — for example, an object that only crossed the line once might get counted twice or more. The main reason:

> Because we're relying on a high-level API, the internal logic for detecting "line crossing" and "avoiding duplicate counts" is hidden from us, and our ability to debug it precisely is limited.

More parameters are also available, such as `track_color` (the color of the tracking trail); you can check Ultralytics' documentation for the full list. But for complete control over the counting logic — and to fix such bugs precisely — we need the second, custom-built approach.

---

## 18. Solution 2: Building a Custom Counter with cv2 and YOLO Track

In this approach, instead of using the ready-made `ObjectCounter`, we write the counting logic ourselves from scratch, combining `model.track()` output with OpenCV functions. This gives us full control and precise debugging ability.

### Accessing each object's tracking ID

The key feature of Object Tracking is that each object gets a **unique ID**, which stays consistent across consecutive frames:

```python
result = model.track(frame, persist=True)[0]

ids = result.boxes.id   # list of IDs for objects detected in this frame
```

- In the **first few frames**, `result.boxes.id` may return `None`, since the model hasn't had a chance to assign IDs yet. This needs to be handled in your code (e.g. with `if result.boxes.id is not None:`).
- After a few frames, each object gets a stable ID (e.g. ID 1, 2, 3...) and keeps it as long as it remains visible in the frame.

### Accessing each bounding box's coordinates

```python
boxes_xywh = result.boxes.xywh   # per ID: x_center, y_center, width, height
```

Combining `ids` and `boxes_xywh` gives us, for each object, both its unique identity and its box's center coordinates.

### Defining the counting line/region and left/right zones

To determine whether an object is crossing toward the "right" or "left," we define a **reference vertical boundary (midpoint)** and compare each box's center against it:

```python
mid_x = 640  # example: the frame's horizontal midpoint

# if x_center > mid_x → right side
# if x_center < mid_x → left side
```

To visualize these boundaries, you can draw small guide circles:

```python
cv2.circle(frame, center=(mid_x + 40, 40), radius=5, color=(255, 0, 0), thickness=-1)  # right
cv2.circle(frame, center=(mid_x - 40, 40), radius=5, color=(255, 0, 0), thickness=-1)  # left
```

> `thickness=-1` draws the circle **filled**.

### The core counting logic (avoiding double-counting)

To prevent an object that's already been counted from being counted again, we keep a **list of already-counted IDs**:

```python
count_right = 0
count_left = 0
id_list = []  # IDs that have already been counted

# ... inside the loop, for each detected object:
if x_center > mid_x and crossed_line and object_id not in id_list:
    count_right += 1
    id_list.append(object_id)

if x_center < mid_x and crossed_line and object_id not in id_list:
    count_left += 1
    id_list.append(object_id)
```

- The `object_id not in id_list` condition is the key to preventing duplicate counting: since tracking keeps the same ID for the same object across frames, without this check, a single object would get counted again in every frame it happens to overlap the line.
- For detecting "has it crossed" (`crossed_line`), rather than comparing exactly against a single-pixel line (which, due to small detection jitter, might never match exactly), it's better to define a small **band**: e.g. "if `y_center` falls between `line_y - 50` and `line_y + 70`, consider it on the line."

```python
line_y = 600
band = 50

crossed_line = (line_y - band) < y_center < (line_y + band)
```

### Handling a common error: `None` IDs in the first frames

Since `result.boxes.id` can be `None` in the earliest frames, trying to iterate over it raises an error. The fix is a simple guard condition:

```python
if result.boxes.id is not None:
    # only run the counting logic once IDs exist
    ...
```

> General takeaway: hitting errors like this along the way is completely normal. The usual workflow is: write an initial version, hit an error, fix it, and keep moving forward (iterative debugging). This cycle repeats until the code is stable.

### Displaying the counters on screen

To show the running count on the frame itself, use `cv2.putText` (usually with a filled rectangle behind the text for readability):

```python
# filled rectangle background for better text readability
cv2.rectangle(frame, (20, 20), (250, 170), color=(0, 0, 0), thickness=-1)

font = cv2.FONT_HERSHEY_SIMPLEX

cv2.putText(frame, f"Right: {count_right}", (40, 70), font, 1, (255, 255, 255), 2)
cv2.putText(frame, f"Left: {count_left}", (40, 140), font, 1, (255, 255, 255), 2)
```

### Fixing common false-positive counts

In real testing, objects that never actually crossed the line (e.g. a roadside sign whose coordinates momentarily land near the line's region) can get counted by mistake. A simple fix — instead of relying on a **single line** — is to define a **band between two lines**, so a count is only valid when the object's center falls inside that region; the same band approach described above directly addresses this issue.

### Managing computational load with a confidence threshold

When the number of detected objects in a frame grows large (e.g. distant, small objects with low confidence), processing load increases too. Setting an appropriate confidence threshold helps manage this:

```python
result = model.track(frame, persist=True, conf=0.4)[0]
```

- A very low confidence threshold (e.g. `0.0` to `0.25`) means the model detects even distant, low-confidence objects — higher recall, but also higher computational load and risk of miscounting.
- A higher threshold (e.g. `0.4` and up) filters out weak predictions, but risks missing some real objects during moments when many objects suddenly appear at once.
- The right value depends entirely on your specific problem; it's a trade-off between accuracy, speed, and computational load — the same trade-off we saw back in the evaluation metrics section (Section 7).

### Summary of the custom approach

Combining the pieces above — getting `ids` and `xywh` from `model.track()`, defining a reference band for crossing detection, keeping a list of already-counted IDs to avoid duplicates, and rendering the counters on screen — lets you build a **fully custom, fully controllable object counter**. Unlike the high-level approach, every piece of its logic is entirely in your hands and easy to debug or adjust.

---

## Learning Path Summary

```
Core concepts (Object / Detection / Tracking)
        │
        ▼
Installing Ultralytics & environment setup
        │
        ▼
Understanding model I/O & annotation format
        │
        ▼
Hands-on exercise: drawing a bounding box with OpenCV
        │
        ▼
Running inference with pretrained detection models
        │
        ▼
Understanding evaluation metrics (IoU, Precision, Recall, AP, mAP)
        │
        ▼
Preparing & annotating a custom dataset (Roboflow)
        │
        ▼
Training the model on the custom dataset in Colab
        │
        ▼
Using the custom model for prediction
        │
        ▼
Validating the custom model & interpreting its metrics
        │
        ▼
Object Tracking on a video file
        │
        ▼
Working with a webcam via OpenCV (frame-by-frame reading)
        │
        ▼
Real-time Object Tracking on webcam output
        │
        ▼
Building a custom dataset via webcam & training a multi-class model
        │
        ▼
Introduction to the Object Counting project and its two approaches
        │
        ▼
Solution 1: Ultralytics' built-in ObjectCounter (high-level)
        │
        ▼
Solution 2: a custom counter built with cv2 and model.track (low-level)
```

## Suggested Resources

- [Official Ultralytics YOLO Docs](https://docs.ultralytics.com)
- [Ultralytics GitHub repository](https://github.com/ultralytics/ultralytics)
- The 2018 paper on object detection evaluation metrics (IoU, Precision/Recall, AP, AR) referenced in the evaluation lecture.

---

> This README was compiled from lecture transcripts for personal review and documentation purposes. Always consult the official Ultralytics documentation for accurate, up-to-date technical details.
