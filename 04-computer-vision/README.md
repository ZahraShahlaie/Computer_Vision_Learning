# 👁️ Advanced Computer Vision

This section contains a structured collection of tutorials, articles, notebooks, experiments, and practical projects covering modern **Computer Vision and Deep Learning**.

The learning path progresses from **object detection and semantic segmentation** to **object tracking, Vision Transformers, and real-world computer vision applications**.

---

## 📚 Course Structure

### 01 - Object Detection

Learn how computer vision models locate and classify objects within images.

Topics include:

* Object Detection Fundamentals
* Classification vs Localization vs Detection
* Bounding Boxes and Confidence Scores
* Dataset Annotation
* IoU, Precision, Recall, AP, and mAP
* R-CNN
* Fast R-CNN
* Faster R-CNN
* SSD
* YOLOv1
* YOLOv2
* YOLOv3

This section covers the evolution from two-stage detectors to fast single-stage detection approaches.

---

### 02 - Semantic Segmentation

Learn how deep learning models perform **pixel-level predictions** and generate segmentation masks.

Topics include:

* Semantic Segmentation Fundamentals
* Pixel-wise Classification
* Binary and Multi-class Segmentation
* Pixel Accuracy, mPA, IoU, mIoU, and Dice
* U-Net
* Encoder-Decoder Architecture
* Skip Connections
* U-Net++
* Residual U-Net
* DeepLab
* Atrous Convolution
* ASPP
* DeepLabV3
* DeepLabV3+

Practical notebooks cover dataset preparation, mask processing, training, and evaluation.

---

### 03 - Object Tracking

Learn how objects can be detected and tracked across consecutive video frames.

Topics include:

* Video Processing with OpenCV
* Object Detection vs Tracking
* Single and Multi-Object Tracking
* Tracking-by-Detection
* Bounding Box Tracking
* Feature-Based Tracking
* Deep Learning-Based Tracking
* Persistent Object IDs
* Multi-Object Tracking

This section includes practical video-based tracking experiments, including football player detection and tracking.

---

### 04 - Vision Transformers

Explore Transformer-based architectures for computer vision and understand how **Vision Transformers (ViT)** process visual information.

Topics include:

* CNNs vs Transformers
* Image Patches
* Patch Embeddings
* Positional Embeddings
* Class Token
* Transformer Encoder
* Self-Attention
* Multi-Head Self-Attention
* Vision Transformer Architecture
* Global Context Modeling
* Advantages and Limitations of ViTs
* Computational Complexity
* Data Requirements

---

### 05 - Practical Computer Vision Projects

Apply the concepts from the previous sections to complete end-to-end computer vision projects.

#### 🔬 Colonoscopy Polyp Detection & Tracking

A YOLO11-based pipeline for detecting and tracking polyps in colonoscopy video.

Key components:

* YOLO11 object detection
* Real-time video inference
* Object tracking
* Persistent tracking IDs
* Automatic frame capture
* Timestamp-based filenames
* Annotated output video

Detected frames are automatically saved for later review.

> **Note:** This is a research/educational project and is not a certified medical device. It does not provide medical diagnosis.

#### 🚗 Road Lane Segmentation & Vehicle Detection/Tracking

An autonomous-driving perception pipeline combining:

* DeepLabV3+ with ResNet50
* YOLO11 object detection
* BoT-SORT multi-object tracking
* Re-identification (ReID)
* Real-time video processing
* Lane and vehicle visualization

The pipeline combines lane segmentation, vehicle detection, and tracking into a unified video output.

---

## 🔄 Computer Vision Workflow

The topics progress from fundamental tasks to practical applications:

**Object Detection → Semantic Segmentation → Object Tracking → Vision Transformers → Real-World Projects**

These concepts demonstrate how computer vision systems can progress from detecting objects to understanding pixels, tracking objects through time, modeling visual relationships, and building complete application-oriented pipelines.

---

## 🧩 Topics Covered

Throughout this section, you will work with:

* Object Detection
* Semantic Segmentation
* Object Tracking
* Multi-Object Tracking
* Video Processing
* Deep Learning
* CNN Architectures
* Encoder-Decoder Networks
* Attention Mechanisms
* Transformer Architectures
* Vision Transformers
* Dataset Preparation
* Data Augmentation
* Model Evaluation
* Real-Time Inference
* Medical Computer Vision
* Autonomous Driving Perception
* Real-World Computer Vision Projects

---

## 📂 Section Structure

```text
04-computer-vision/
│
├── 01-object-detection/
├── 02-segmentation/
├── 03-object-tracking/
├── 04-vision-transformers/
│
├── 05-projects/
│   ├── colonoscopy-polyp-detection-and-tracking/
│   └── lane-segmentation-vehicle-tracking/
│
└── README.md
```

Each topic contains dedicated articles, notebooks, experiments, and supporting materials.

---

## 🎯 Learning Outcomes

By completing this section, you will be able to:

* Build object detection pipelines
* Prepare and evaluate computer vision datasets
* Perform pixel-level image segmentation
* Work with segmentation masks
* Understand and implement object tracking
* Process and analyze video data
* Work with Transformer-based vision models
* Compare CNN and Transformer approaches
* Build real-time computer vision pipelines
* Combine multiple models into practical applications

---

## 🎯 Learning Path

**01 → Object Detection**

Learn how objects are located and classified.

⬇️

**02 → Semantic Segmentation**

Move from object-level predictions to pixel-level understanding.

⬇️

**03 → Object Tracking**

Extend visual understanding from images to video sequences.

⬇️

**04 → Vision Transformers**

Explore modern Transformer-based approaches for visual understanding.

⬇️

**05 → Practical Projects**

Combine the learned concepts into real-world computer vision applications.

---

## 🚀 Goal

The goal of this section is to build a strong understanding of **Computer Vision and Deep Learning**, progressing from fundamental concepts to advanced architectures and complete practical applications.

```text
Computer Vision Fundamentals
            │
            ▼
     Object Detection
            │
            ▼
   Semantic Segmentation
            │
            ▼
     Object Tracking
            │
            ▼
   Vision Transformers
            │
            ▼
  Practical Applications
```

---

## 📁 Repository Navigation

| Directory                 | Description                                                |
| ------------------------- | ---------------------------------------------------------- |
| `01-object-detection/`    | Object detection concepts, algorithms, and implementations |
| `02-segmentation/`        | Semantic segmentation, U-Net, DeepLab, and experiments     |
| `03-object-tracking/`     | Video processing and object tracking                       |
| `04-vision-transformers/` | Vision Transformer concepts and architectures              |
| `05-projects/`            | End-to-end practical computer vision projects              |

---

## 👩‍💻 Author

**Zahra Shahlaie**

Created for learning and educational purposes in:

* Computer Vision
* Deep Learning
* Artificial Intelligence
* Machine Learning

---

⭐ **This section is continuously evolving as new computer vision experiments and projects are added.**
