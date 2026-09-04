# YOLO Pose Estimation Tutorial — Keypoint Detection


## Table of Contents

1. [What Is Pose Estimation? And What Is It Used For?](#1-what-is-pose-estimation-and-what-is-it-used-for)
2. [Pose Models in YOLO and Running Prediction](#2-pose-models-in-yolo-and-running-prediction)
3. [Working with the Keypoints Output](#3-working-with-the-keypoints-output)
4. [Building a Rep Counter with a Webcam](#4-building-a-rep-counter-with-a-webcam)
5. [Annotating Custom Pose Data in Roboflow](#5-annotating-custom-pose-data-in-roboflow)
6. [The Pose Estimation Annotation File Format](#6-the-pose-estimation-annotation-file-format)

---

## 1. What Is Pose Estimation? And What Is It Used For?

**Pose Estimation** (sometimes also called "keypoint detection") is another task in the YOLO family. Unlike Detection, which looks for a bounding box, or Segmentation, which looks for a pixel-level mask, Pose Estimation's goal is to locate a set of **predefined, specific keypoints** on an object.

The most common example: detecting keypoints on the **human body** — nose, eyes, ears, shoulders, elbows, wrists, knees, ankles, and so on. For each of these points, the model predicts its (x, y) coordinates in the image.

> Note: "Pose Estimation" isn't a literal translation of "keypoint detection" (it more literally means "estimating body posture"), but in practice both terms are used interchangeably for this task.

### Why is this useful? A few practical examples

- **Gesture control:** e.g. measuring the distance between two keypoints (like the tip of the thumb and the index finger) to adjust system volume, or opening/closing the hand to toggle mute.
- **Counting exercise reps:** e.g. counting push-ups or pull-ups by tracking the movement of specific joints (which we'll implement in Section 4).
- **VR/AR applications:** tracking a user's body movements to control an avatar or a virtual environment.
- More generally, any problem where **the movement or relative position of a few specific points on an object** matters — not just whether the object exists or its general location.

---

## 2. Pose Models in YOLO and Running Prediction

### Loading the model

Ultralytics pose estimation models are identified by the **`-pose`** suffix:

```python
from ultralytics import YOLO

model = YOLO("yolov8n-pose.pt")   # Nano version, specifically for pose estimation
```

Like other tasks, several size variants exist here too (Nano through X-Large), with the same familiar speed/accuracy trade-off.

### Running prediction

```python
results = model.predict(source="sample.jpg")
result = results[0]
```

### Displaying the result

```python
result.show()   # shows the image with both bounding boxes and keypoints
```

Displaying the result shows the model doing two things simultaneously:
1. **Regular object detection**: drawing a bounding box around each person (or other object of interest).
2. **Keypoint detection**: marking that same object's keypoints (nose, eyes, ears, shoulders, elbows, wrists, knees, ankles, etc.).

### Controlling display with `plot` parameters

```python
plt.imshow(result.plot(boxes=False, conf=False))
plt.show()
```

- `boxes=False`: hides the bounding boxes, showing only keypoints.
- `conf=False`: hides the confidence values printed on the image.

> Reminder: since `result.plot()` returns the image in BGR, you'll need `cv2.cvtColor(..., cv2.COLOR_BGR2RGB)` for colors to display correctly with `matplotlib` — the same note that applies across other tasks too.

---

## 3. Working with the Keypoints Output

### Accessing keypoints

```python
result.keypoints
```

This object holds the full keypoint information for every detected object in the image.

### The shape of the data

```python
result.keypoints.xy.shape
# e.g.: (8, 17, 2)
```

You can interpret these three dimensions as:

- **First dimension (e.g. 8):** the number of detected objects in the image (e.g. 8 people).
- **Second dimension (17):** the number of keypoints defined per object — the standard COCO Pose format (which YOLO uses) defines **17 keypoints** per person (nose, eyes, ears, shoulders, elbows, wrists, hips, knees, ankles).
- **Third dimension (2):** the `x` and `y` values (normalized, between 0 and 1) for each point.

```python
result.keypoints.conf.shape
# e.g.: (8, 17)
```

Each of the 17 points also has its own separate confidence value — meaning the model can be more confident about detecting some points (like a shoulder) than others (like an ankle).

### Points that weren't detected

If the model can't detect a specific point (e.g. because it's obscured in the image):

- That point's **confidence** value comes out very low.
- Its coordinates usually stay near `(0, 0)`.
- In `result.plot()`, that specific point isn't drawn (since its confidence is below the default threshold) — but it still occupies a slot in the output matrix (`xy`/`conf`), just with a value near zero.

> Important: since every object must always have exactly 17 points (a fixed format), even undetected points still take up a slot in the output matrix — just with very low confidence. This means when processing the output, you should always check each point's confidence to confirm it's a valid detection.

### Accessing boxes (as before)

```python
result.boxes.data     # full info for each bounding box (coordinates + confidence)
result.boxes.conf      # just the confidence value for each object
```

This part is exactly what we saw in Object Detection — Pose Estimation performs detection alongside keypoint detection.

---

## 4. Building a Rep Counter with a Webcam

In this section, we build a practical app: counting pull-up repetitions using a webcam and a pose model.

### The general idea

By comparing the position (the Y coordinate) of two keypoints — say, the **shoulder** and the **elbow** — we can tell when a person transitions from one state to another. Each time this transition happens, we increment a counter.

### Keypoint indices in the COCO Pose format

To know exactly which index in `result.keypoints` corresponds to which body point, we need to check the documentation. Based on the standard COCO Pose format that YOLO uses:

| Index | Body point |
|---|---|
| 0 | Nose |
| 5 | Left shoulder |
| 6 | Right shoulder |
| 7 | Left elbow |
| 8 | Right elbow |

> Left/right numbering always refers to **the subject's own left/right** in the image, not the viewer's.

### The main webcam-reading and pose-detection loop

```python
import cv2
from ultralytics import YOLO

model = YOLO("yolov8n-pose.pt")
cap = cv2.VideoCapture("pull_up_sample.mp4")   # or 0 for a live webcam

while cap.isOpened():
    ret, frame = cap.read()
    if not ret:
        break

    results = model.predict(frame, show=False)
    result = results[0]

    # ... extract and process keypoints below

    cv2.imshow("pull up", frame)
    if cv2.waitKey(1) & 0xFF == ord("q"):
        break

cap.release()
cv2.destroyAllWindows()
```

### Extracting the points we need

```python
keypoints_xy = result.keypoints.xy[0]   # the first detected object (e.g. the first person)

left_shoulder = keypoints_xy[5]
left_elbow = keypoints_xy[7]

right_shoulder = keypoints_xy[6]
right_elbow = keypoints_xy[8]
```

Each of these variables is an `(x, y)` pair.

### Drawing the points on screen to verify correctness

To confirm you've got the right point for the right joint, it's a good idea to first draw them in different colors on the frame:

```python
x, y = left_shoulder
cv2.circle(frame, (int(x), int(y)), radius=6, color=(255, 255, 255), thickness=-1)   # white: left shoulder

x, y = left_elbow
cv2.circle(frame, (int(x), int(y)), radius=6, color=(0, 255, 0), thickness=-1)        # green: left elbow
```

> Important: the `x, y` coordinates from the model are **floats**, but `cv2.circle` expects integer coordinates. Without this conversion, you'll get an error — so always apply `int(x)` and `int(y)` before drawing.

### The core counting logic

Since in image coordinate systems **the Y axis increases from top to bottom** (unlike typical math graphs), when the shoulder moves "above" the elbow (i.e. physically moves up as the body rises), the shoulder's `y` value becomes **smaller** than the elbow's `y` value.

```python
counter = 0

# ... inside the loop, after extracting the points:
if left_shoulder[1] < left_elbow[1] and right_shoulder[1] < right_elbow[1]:
    counter += 1
```

### A common bug: counting multiple times during a single crossing

If you implement the logic above naively, you'll notice the counter jumps up **multiple times** during what should be a single pull-up — because the condition above holds true for **every frame** the shoulder stays above the elbow (and that spans several frames for one rep).

**The fix:** keep a **state variable** tracking whether you're currently "inside" a rep or not, and only increment the counter at the moment of a **state transition** (not on every frame where the condition holds):

```python
counter = 0
in_position = False   # are we currently in the "up" position?

# ... inside the loop:
is_up = left_shoulder[1] < left_elbow[1] and right_shoulder[1] < right_elbow[1]

if is_up and not in_position:
    counter += 1
    in_position = True
elif not is_up:
    in_position = False
```

With this logic, the counter increments only once at the **moment of transition** from down to up, rather than on every frame the person stays in the up position.

### Displaying the counter on screen

```python
cv2.putText(
    frame,
    f"pull up: {counter}",
    (1080, 60),
    cv2.FONT_HERSHEY_SIMPLEX,
    1,
    (0, 0, 0),
    2,
)
```

- The `(1080, 60)` coordinates should be adjusted to match your actual frame dimensions (e.g. 1280×720).
- If the placement doesn't look right, simply adjust the coordinates.

---

## 5. Annotating Custom Pose Data in Roboflow

To train a pose model on your own custom problem (e.g. detecting finger positions instead of the whole human body), you need to annotate your own data.

### General steps in Roboflow

1. **Define the class (object) and its keypoints.** For example, say you want to detect whether a hand is open or closed:
   - Class `close` (closed fist)
   - Class `hand` (open hand)
   - For each class, define the keypoints you want — e.g. `F3` (index finger) and `F1` (palm), or `F2` (thumb). Naming these keypoints is entirely up to you.
2. **Annotating each image:**
   - First, mark the object's type (class) in the image and draw a bounding box around it — exactly like regular Object Detection.
   - Then, for classes where the keypoints are **actually visible** (e.g. the `hand` class, where fingers can be seen), click on each defined keypoint one by one.
   - For classes where the keypoints **aren't visible** (e.g. the `close` class, where the fist is closed and fingers aren't visible), those points are simply skipped — their annotation is automatically recorded as "doesn't exist" (details of this format are in Section 6).
3. Repeat this process for every image in the dataset.
4. As with the previous sections (Detection/Segmentation), the **Generate** step lets you add **augmentation** (e.g. brightness changes or added noise) to artificially expand the dataset — e.g. going from 10 raw images to 20 after one round of augmentation.
5. Finally, download the annotated dataset, or fetch it directly via code, for use in Colab or your local environment — exactly the same process as Detection and Segmentation.

---

## 6. The Pose Estimation Annotation File Format

The annotation file in Pose Estimation contains both **bounding box** information and **keypoint** information — and its structure is slightly more complex than regular Detection.

### The general structure of each annotation line

```
class_id  x_center  y_center  width  height  [x1 y1 (v1)]  [x2 y2 (v2)]  ...
```

- The **first 5 values** are exactly what we saw in Object Detection: class ID, center, width, and height of the bounding box (all normalized between 0 and 1).
- The **remaining values** correspond to the keypoints defined for that class, and can follow one of two formats:

### Format one: two values per keypoint (`dim=2`)

Only `x` and `y` are stored for each point (no visibility info):

```
class_id  x_c y_c w h   x1 y1   x2 y2   ...
```

### Format two: three values per keypoint (`dim=3`)

In addition to `x` and `y`, a third value called **visibility** is also stored:

```
class_id  x_c y_c w h   x1 y1 v1   x2 y2 v2   ...
```

The `v` (visibility) value can be one of three states:

| `v` value | Meaning |
|---|---|
| `0` | This keypoint wasn't marked in the annotation at all (doesn't exist / wasn't present in the image) |
| `1` | The keypoint was marked, but may be occluded or somewhat unclear (e.g. hidden behind another object) |
| `2` | The keypoint was marked and is clearly visible in the image |

> In practice, most common annotations only use `0` (point doesn't exist at all) and `2` (point is fully clear); the `1` value is mostly used for edge cases like partial occlusion.

### A real example: an image with two visible keypoints

```
1  0.52 0.48 0.30 0.55   0.55 0.40 2   0.48 0.60 2
```

- `1`: the class ID (e.g. `hand`).
- `0.52 0.48 0.30 0.55`: the bounding box coordinates.
- `0.55 0.40 2`: the first point (e.g. index finger) — `x=0.55, y=0.40`, visibility `2` (fully clear).
- `0.48 0.60 2`: the second point (e.g. palm) — visibility `2`.

### Another example: an image where the keypoints aren't visible

If the annotated class is one where keypoints simply aren't visible (e.g. the `close` class, a closed fist where fingers can't be seen), the values for those points are filled with `0`:

```
3  0.50 0.50 0.20 0.20   0 0 0   0 0 0
```

- `3`: the class ID (e.g. `close`).
- `0.50 0.50 0.20 0.20`: the bounding box coordinates (placed at the image center).
- `0 0 0` and `0 0 0`: both keypoints with zero values — meaning "this point doesn't exist and isn't visible."

> Important: since every class in a dataset must always have a **fixed number of keypoints** (the format must stay consistent), even when an object has no visible keypoints at all, its annotation still needs the same number of values (filled with zeros) — this is a structural convention to keep the format uniform, not a real constraint on the data itself.

### The `data.yaml` file and the `kpt_shape` parameter

In Pose Estimation, the `data.yaml` file has an extra, important parameter beyond the usual `train`/`val` paths and class names, called **`kpt_shape`**, which specifies how many keypoints each object has and how many values (2 or 3) each keypoint stores in the annotation:

```yaml
train: ../train/images
val: ../valid/images
nc: 2
names: ['close', 'hand']
kpt_shape: [2, 3]   # 2 keypoints, each with 3 values (x, y, visibility)
```

- The first number in `kpt_shape` (here, `2`) equals the number of keypoints you defined per object (e.g. index finger and palm).
- The second number (here, `3`) indicates which annotation format you're using — `2` for the no-visibility format, `3` for the format that includes visibility.

---

## Learning Path Summary for This README

```
What is Pose Estimation? Practical uses (gesture control, rep counting, VR/AR)
        │
        ▼
The -pose models in YOLO and running prediction
        │
        ▼
Working with result.keypoints (xy, conf, shape, undetected points)
        │
        ▼
Building a live rep counter with a webcam (counting logic, avoiding duplicate counts)
        │
        ▼
Annotating custom pose data in Roboflow (defining custom classes and keypoints)
        │
        ▼
The annotation file format (bounding box + keypoints + visibility) and the kpt_shape parameter
```

## Suggested Resources

- [Official Ultralytics Docs — Pose Estimation](https://docs.ultralytics.com/tasks/pose/)
- [Ultralytics Docs — Pose Dataset Format](https://docs.ultralytics.com/datasets/pose/)

---

> This README was compiled from lecture transcripts for personal review and documentation purposes. Always consult the official Ultralytics documentation for accurate, up-to-date technical details.
