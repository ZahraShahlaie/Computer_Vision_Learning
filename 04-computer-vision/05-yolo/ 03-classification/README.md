# YOLO Classification Tutorial — Image Classification


## Table of Contents

1. [What Is Image Classification? How It Differs from Detection/Segmentation](#1-what-is-image-classification-how-it-differs-from-detectionsegmentation)
2. [Classification Models in YOLO](#2-classification-models-in-yolo)
3. [Evaluation Metrics: Top-1 and Top-5 Accuracy](#3-evaluation-metrics-top-1-and-top-5-accuracy)
4. [Running Prediction with a Pretrained Model](#4-running-prediction-with-a-pretrained-model)
5. [Dataset Structure for Training Classification](#5-dataset-structure-for-training-classification)
6. [Training a Classification Model on Custom Data](#6-training-a-classification-model-on-custom-data)
7. [Managing Overfitting: Dropout, Weight Decay, and Freezing Layers](#7-managing-overfitting-dropout-weight-decay-and-freezing-layers)

---

## 1. What Is Image Classification? How It Differs from Detection/Segmentation

Among the various tasks YOLO supports (Detection, Tracking, Segmentation, and Classification), **Image Classification** is the simplest.

In Object Detection and Segmentation, the model needs to figure out **how many** objects are in the image, **where** they are (a bounding box or a mask), and **what class** each one belongs to. In Classification, none of that applies:

> The model just says, once: "somewhere in this whole image, I saw such-and-such" — that's it. It doesn't care how many objects there are, or where they're located.

In other words:

- **The model's output is just a single label for the entire image** (plus a confidence score for that label).
- The image is classified as one whole unit — not pixel-by-pixel (like Segmentation) and not object-by-object (like Detection).
- If an image contains multiple distinct objects (e.g. both a dog and a cat), Classification can't tell them apart — it simply wasn't designed for that. This is exactly where Object Detection comes in instead.

Because of this, Classification is best suited for problems where each input image already has **one clear, dominant subject** (e.g. a photo focused entirely on a single product or scene) and precise localization isn't needed.

---

## 2. Classification Models in YOLO

Ultralytics classification models are identified by the **`-cls`** suffix in the filename:

```python
from ultralytics import YOLO

model = YOLO("yolov8n-cls.pt")   # Nano version, specifically for classification
```

### An important difference: the base dataset

Unlike detection and segmentation models, which are pretrained on the **COCO** dataset (80 classes), YOLO's classification models are pretrained on **ImageNet**, which has around **1,000 classes**. This means the default classification model can predict a much wider range of categories than the detection/segmentation models.

### Model size variants

Like other YOLO tasks, several size variants are available: **Nano, Small, Medium, Large, X-Large**. The main differences:

- **Input image size** (e.g. a default of 224×224 or similar, depending on the variant).
- **Accuracy** — larger models are generally more accurate, but slower — the same speed/accuracy trade-off we saw in Detection.

---

## 3. Evaluation Metrics: Top-1 and Top-5 Accuracy

Since a classification model's output is a **probability vector** over all classes (e.g. an array of 1,000 values for ImageNet's 1,000 classes), evaluation typically uses two common metrics:

### Top-1 Accuracy

The model is counted as "correct" only if the **single highest-probability** prediction exactly matches the true class. This is the stricter metric — exactly what you'd normally expect from a model: get the first guess right.

### Top-5 Accuracy

The model is counted as "correct" if the true class appears anywhere among the **5 highest-probability predictions** — even if the top prediction itself was wrong. This is naturally an **easier** bar to clear than Top-1 (since the model gets more chances to be right), so it's always true that:

```
Top-5 Accuracy  ≥  Top-1 Accuracy
```

> Why care about Top-5 at all? Because in datasets with a large number of classes (like ImageNet's 1,000), some classes are visually very similar to each other (e.g. different dog breeds). Even a good model may hesitate between a few similar classes; Top-5 shows whether the model was at least "in the right neighborhood," even when its very top guess was off.

---

## 4. Running Prediction with a Pretrained Model

```python
from ultralytics import YOLO

model = YOLO("yolov8n-cls.pt")

results = model.predict(source="sample.jpg")
result = results[0]
```

### Accessing the prediction result

Classification output lives in `result.probs` — an object holding the probability vector over all classes, plus some handy summaries:

```python
result.probs.top1        # index of the highest-probability class (the model's final prediction)
result.probs.top1conf     # confidence value for that top class
result.probs.top5         # indices of the 5 highest-probability classes
result.probs.top5conf     # confidence values for those 5 classes

result.names[result.probs.top1]   # human-readable name of the predicted class (e.g. "golden_retriever")
```

- `result.probs.data` gives the raw probability vector (one value per class, e.g. 1,000 values); each entry is the probability the image belongs to that specific class.
- For most practical purposes, `top1` (the final result) and `top5` (for closer inspection) are all you need.

---

## 5. Dataset Structure for Training Classification

Unlike Object Detection and Segmentation, which require a **separate annotation file** per image (with box coordinates or polygon points), a YOLO classification dataset needs no annotation files at all. Instead, the **folder structure itself** serves as the annotation:

```
dataset/
├── train/
│   ├── class_1/
│   │   ├── image_001.jpg
│   │   ├── image_002.jpg
│   │   └── ...
│   ├── class_2/
│   │   ├── image_001.jpg
│   │   └── ...
│   └── ... (one folder per class)
└── val/
    ├── class_1/
    │   └── ...
    ├── class_2/
    │   └── ...
    └── ...
```

- Two top-level folders: **`train`** and **`val`** (just like the structure we saw for Detection/Segmentation).
- Inside each, there's **one subfolder per class**; the subfolder's name is treated as the class name.
- All images belonging to a given class go directly into that subfolder — there's no need to manually annotate each image, since the file's location alone determines its class.

> During training, Ultralytics automatically reads this folder structure: it reports the number of images and classes found separately for both `train` and `val` (e.g. "14,000 images across 6 classes for train, and 1,500 images across 6 classes for val").

### Checking image size before training

Before starting training, it helps to know what size your dataset's images actually are (since this affects the `imgsz` training parameter):

```python
import cv2
import matplotlib.pyplot as plt

image = cv2.imread("dataset/train/class_1/sample.jpg")
plt.imshow(image)
plt.show()

print(image.shape)   # e.g. (150, 150, 3)
```

---

## 6. Training a Classification Model on Custom Data

### Running training

```python
from ultralytics import YOLO

model = YOLO("yolov8n-cls.pt")

model.train(
    data="dataset/",       # path to the dataset's root folder (containing train/ and val/)
    epochs=100,
    imgsz=150,               # should match your dataset's actual image size
)
```

- Unlike Detection/Segmentation, where the `data` parameter points to a **`.yaml`** file, here `data` is simply the **path to the dataset's root folder** — as explained in Section 5, the folder structure itself already contains all the needed information (classes, and the train/val split).
- `imgsz` should ideally match your dataset's actual image dimensions.

### What Ultralytics reports when training starts

When you run it, Ultralytics typically shows:

- The number of images and classes found in the `train` folder (e.g. "14,000 images across 6 classes").
- The number of images and classes found in the `val` folder.
- The path where logs and results will be saved (similar to what we saw for Object Detection).
- If a previous run with the same settings exists, a warning about reusing a cache or needing an update.

### Practical tip: use a stronger GPU for faster training

If each epoch is taking a long time, you can switch to a more powerful GPU type in Colab to speed up training considerably. Reducing the number of epochs (e.g. from 100 to 50) for an initial, faster test run is also a reasonable practical approach.

---

## 7. Managing Overfitting: Dropout, Weight Decay, and Freezing Layers

After training finishes, checking the `results.png` plot usually shows two important curves: **loss on the training data**, and **loss/accuracy on the validation data**.

> A common observation in practice: if you notice a **large, meaningful gap** between training loss and validation loss (training performs great, validation doesn't), that's a classic sign of **overfitting** — the model is memorizing specifics of the training data rather than learning generalizable patterns.

Ultralytics exposes several tunable parameters to help reduce overfitting:

### Dropout

```python
model.train(
    data="dataset/",
    epochs=100,
    imgsz=150,
    dropout=0.25,   # e.g. 25% of neurons are randomly disabled at each step
)
```

By randomly disabling a portion of the network's neurons during training, dropout prevents the model from becoming overly dependent on specific pathways, and usually helps with generalization. You can try different values (e.g. 25% vs. 30%) and compare validation results.

### Weight Decay (L2 Regularization)

Another parameter that can help control overfitting is **weight decay** — a form of L2 regularization that penalizes large weights during training, nudging the model toward smaller, more stable weights:

```python
model.train(
    data="dataset/",
    epochs=100,
    imgsz=150,
    weight_decay=0.001,   # a small value, for example
)
```

### Freezing the early layers (transfer learning)

Another technique is to "freeze" the first few layers of the pretrained model — telling it not to update those layers during training:

```python
model.train(
    data="dataset/",
    epochs=100,
    imgsz=150,
    freeze=5,   # e.g. the first 5 layers won't be updated
)
```

The reasoning: the earliest layers of a pretrained model typically learn general, low-level features (like edges and textures) that are common across most vision problems. By freezing them, the model only fine-tunes the later, more specialized layers on your data — which can both reduce overfitting and speed up training.

### Wrapping up, with a realistic expectation

By experimenting with different combinations of these parameters (dropout, weight decay, freezing) and comparing Top-1/Top-5 accuracy alongside the train/validation loss gap, you can find the best settings for your dataset.

> An important expectation to set: just like with Segmentation, keep in mind that **YOLO wasn't originally designed for Classification** — this capability was added later. So if squeezing out the absolute highest accuracy on a pure classification problem is your goal, dedicated classification architectures (like ResNet, EfficientNet, Vision Transformers, and similar) will usually outperform it. But if you want an all-in-one tool with a consistent API alongside Detection/Segmentation/Tracking, YOLO's classification support is a perfectly usable option.

---

## Learning Path Summary for This README

```
What is Classification? How it differs from Detection/Segmentation
        │
        ▼
The -cls models in YOLO and the ImageNet base dataset
        │
        ▼
Evaluation metrics: Top-1 Accuracy and Top-5 Accuracy
        │
        ▼
Running prediction and reading result.probs
        │
        ▼
The dataset folder structure for Classification (no annotation files)
        │
        ▼
Training a model on custom data
        │
        ▼
Managing overfitting with dropout, weight decay, and freezing layers
```

## Suggested Resources

- [Official Ultralytics Docs — Classification](https://docs.ultralytics.com/tasks/classify/)
- [Ultralytics Docs — Classification Datasets](https://docs.ultralytics.com/datasets/classify/)

---

> This README was compiled from lecture transcripts for personal review and documentation purposes. Always consult the official Ultralytics documentation for accurate, up-to-date technical details.
