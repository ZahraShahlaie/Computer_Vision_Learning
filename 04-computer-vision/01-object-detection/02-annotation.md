# Annotation and Dataset Structure in Object Detection

## What Is Annotation?

In Image Classification tasks, it is usually sufficient to know which class an image belongs to.

For example:

* Dog
* Cat
* Car

However, in **Object Detection**, knowing only the class is not enough.

The model must not only identify **what object is present**, but also determine **where the object is located within the image**.

For this reason, additional information must be stored for each object. This information is called an **Annotation**.

A typical annotation contains two main components:

1. **Class Label** — the category of the object
2. **Bounding Box** — the location of the object within the image

---

# Dataset Structure

To simplify the learning process, only three classes were selected from a larger dataset:

* Airplane
* Face
* Motorcycle

The dataset is organized as follows:

```text
dataset
│
├── images
│   ├── airplane
│   ├── face
│   └── motorcycle
│
└── annotations
    ├── airplane.csv
    ├── face.csv
    └── motorcycle.csv
```

The **images** directory contains the images for each class.

The **annotations** directory contains the corresponding annotation files in CSV format.

---

# Relationship Between Images and Annotations

Each CSV file stores the annotations for the images belonging to a specific class.

For example:

```text
images/airplane
annotations/airplane.csv
```

The file `airplane.csv` contains annotation information for all airplane images.

A typical annotation file may look like:

| image_name    | x1 | y1 | x2  | y2  |
| ------------- | -- | -- | --- | --- |
| image_001.jpg | 96 | 44 | 368 | 211 |
| image_002.jpg | 55 | 39 | 290 | 180 |

Each row represents a single image and the coordinates of its bounding box.

---

# Bounding Box Coordinates

Bounding boxes can be stored using different formats.

## Method 1: Corner Coordinates

The bounding box is represented using two corner points:

```text
(x1, y1)
(x2, y2)
```

where:

* `(x1, y1)` is the top-left corner
* `(x2, y2)` is the bottom-right corner

Visually:

```text
(x1, y1)
     ●───────────────┐
     │               │
     │    Object     │
     │               │
     └───────────────●
                 (x2, y2)
```

---

## Method 2: Center and Size

The bounding box can also be represented using:

```text
center_x
center_y
width
height
```

This format is commonly used in some object detection frameworks such as YOLO.

In the dataset used here, the **corner-coordinate format** is used:

```text
[x1, y1, x2, y2]
```

---

# Reading Annotation Information

The annotation file is first loaded using Pandas:

```python
annotation_df = pd.read_csv(annotation_path)
```

The image name can then be extracted from the `image_name` column:

```python
file_name = annotation_df["image_name"].iloc[file_index]
```

Using this filename, the corresponding image path is constructed and the image is loaded.

---

# Extracting Bounding Boxes

After locating the row associated with a specific image, the bounding box coordinates can be extracted:

```text
[x1, y1, x2, y2]
```

These values define the exact position of the object within the image.

For example:

```text
x1 = 96
y1 = 44
x2 = 368
y2 = 211
```

---

# Visualizing Bounding Boxes

Bounding boxes can be drawn on images using OpenCV:

```python
cv2.rectangle(...)
```

The result is a rectangular box surrounding the object.

For example:

* A rectangle is drawn around an airplane.
* A rectangle is drawn around a face.
* A rectangle is drawn around a motorcycle.

This rectangle is simply a visual representation of the annotation information used during training.

---

# The Problem with Image Resizing

One of the most important challenges in Object Detection arises when images are resized.

Suppose the original image size is:

```text
1500 × 2000
```

but the model expects images of size:

```text
224 × 224
```

The image can be resized easily, but the bounding box coordinates still correspond to the original image dimensions.

As a result, the bounding box will no longer align correctly with the object.

```text
Original Image
      ↓
Resize Image
      ↓
Bounding Box becomes incorrect
```

---

# Solution: Relative Coordinates

A common solution is to store bounding box coordinates in a normalized form.

Each coordinate is divided by the image dimensions:

```python
x1 = x1 / image_width
y1 = y1 / image_height

x2 = x2 / image_width
y2 = y2 / image_height
```

After normalization, all coordinate values fall within the range:

```text
0 → 1
```

Example:

```text
x1 = 0.25
y1 = 0.18
x2 = 0.74
y2 = 0.63
```

These values now represent relative positions instead of absolute pixel locations.

---

# Reconstructing Bounding Boxes After Resizing

Once the image has been resized, the normalized coordinates can be converted back to pixel coordinates.

For example, if the new image size is:

```text
224 × 224
```

then:

```python
x1 = x1_relative * 224
y1 = y1_relative * 224

x2 = x2_relative * 224
y2 = y2_relative * 224
```

This reconstructs the bounding box in the resized image while preserving its correct position.

The main advantage of normalized coordinates is that:

* The image can be resized to any resolution.
* The bounding box remains valid.
* The object location is preserved consistently.

---

# Displaying the Class Name

In addition to drawing a bounding box, it is common to display the class name on the image.

For example:

```text
Airplane
```

This can be done using OpenCV:

```python
cv2.putText(...)
```

The text is typically placed near the upper corner of the bounding box so that the detected object can be easily identified.

For example:

```text
┌──────────────────────┐
│ Airplane             │
│ ┌──────────────────┐ │
│ │                  │ │
│ │    Airplane      │ │
│ │                  │ │
│ └──────────────────┘ │
└──────────────────────┘
```

The final visualization usually contains:

* The image
* The bounding box
* The class label

Together, these elements represent the annotation information that will later be used to train an Object Detection model.
