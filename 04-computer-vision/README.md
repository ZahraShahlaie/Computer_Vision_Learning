# 👁️ Advanced Computer Vision

This repository contains a structured collection of tutorials, articles, notebooks, experiments, and practical projects covering important topics in modern **Computer Vision and Deep Learning**.

This section progresses from fundamental computer vision tasks such as **object detection** and **semantic segmentation**, through **object tracking** and **Vision Transformers**, and finally applies these concepts to **real-world computer vision projects**.

Each subfolder focuses on a specific computer vision task, architecture, or practical application and combines theoretical concepts with hands-on implementations.

---

## 📚 Course Structure

### 01 - Object Detection

Learn how computer vision models can locate and classify multiple objects within an image.

Topics include:

* Introduction to Object Detection
* Classification vs Localization vs Detection
* Bounding Boxes
* Object Classes
* Confidence Scores
* Object Detection Pipeline
* Dataset Annotation
* Evaluation Metrics
* Intersection over Union (IoU)
* Precision and Recall
* Average Precision (AP)
* Mean Average Precision (mAP)
* R-CNN
* Fast R-CNN
* Faster R-CNN
* SSD
* YOLOv1
* YOLOv2
* YOLOv3

This section covers the evolution of object detection from region-based two-stage detectors to fast single-stage detection approaches.

---

### 02 - Semantic Segmentation

Learn how deep learning models perform **pixel-level predictions** and assign semantic classes to image regions.

Topics include:

* Introduction to Semantic Segmentation
* Image Classification vs Object Detection vs Segmentation
* Pixel-wise Classification
* Segmentation Masks
* Binary Segmentation
* Multi-class Segmentation
* Evaluation Metrics
* Pixel Accuracy
* Mean Pixel Accuracy (mPA)
* Intersection over Union (IoU)
* Mean Intersection over Union (mIoU)
* Dice Coefficient
* U-Net
* Encoder-Decoder Architecture
* Skip Connections
* Advanced U-Net Architectures
* U-Net++
* Residual U-Net
* DeepLab
* Atrous Convolution
* Atrous Spatial Pyramid Pooling (ASPP)
* DeepLabV3
* DeepLabV3+

The practical notebooks demonstrate the complete semantic segmentation workflow, from dataset preparation and mask processing to model training and evaluation.

---

### 03 - Object Tracking

Learn how objects can be detected and tracked across consecutive video frames.

Topics include:

* Video Processing with OpenCV
* Video Frames
* Object Tracking
* Object Detection vs Object Tracking
* Single Object Tracking
* Multiple Object Tracking
* Bounding Box Tracking
* Tracking-by-Detection
* Tracking Algorithms
* Feature-Based Tracking
* Deep Learning-Based Tracking
* Persistent Object IDs
* Multi-Object Tracking

This section also includes practical video-based tracking experiments, including a football player detection and tracking project.

The goal is to understand how computer vision systems can move from analyzing individual images to maintaining object identities over time.

---

### 04 - Vision Transformers

Explore Transformer-based architectures for computer vision and understand how the **Vision Transformer (ViT)** processes visual information.

Topics include:

* Introduction to Vision Transformers
* CNNs vs Transformers
* Image Patches
* Patch Embeddings
* Positional Embeddings
* Class Token
* Transformer Encoder
* Self-Attention
* Multi-Head Self-Attention
* Vision Transformer Architecture
* Advantages of Vision Transformers
* Limitations of Vision Transformers
* Computational Complexity
* Data Requirements
* Global Context Modeling

This section provides the foundation required for understanding modern Transformer-based computer vision architectures.

---

### 05 - Practical Computer Vision Projects

Apply the concepts learned throughout the previous sections to complete end-to-end computer vision projects based on real-world scenarios.

This section focuses on combining multiple computer vision techniques into practical pipelines rather than studying individual algorithms in isolation.

Current projects include:

#### 🔬 Colonoscopy Polyp Detection & Tracking

A YOLO-based computer vision pipeline for detecting polyps in colonoscopy video.

The project combines:

* YOLO11 object detection
* Real-time video inference
* Object tracking
* Persistent tracking IDs
* Automatic frame capture
* Timestamp-based frame naming
* Annotated output video generation

Whenever a polyp is detected, the corresponding annotated frame is automatically saved for later review.

This project demonstrates how object detection and tracking can be combined into a practical medical-imaging workflow.

> **Important:** This is a research/educational project and is not a certified medical device. It does not provide medical diagnosis or determine polyp type or malignancy.

#### 🚗 Road Lane Segmentation & Vehicle Detection/Tracking

An end-to-end autonomous-driving perception pipeline combining:

* DeepLabV3+ semantic segmentation
* ResNet50 backbone
* YOLO11 object detection
* BoT-SORT multi-object tracking
* Re-identification (ReID)
* Real-time video processing
* Lane-mask and vehicle-overlay visualization

The pipeline processes driving video and combines lane segmentation with vehicle detection and tracking to create a unified perception output.

These projects demonstrate how the concepts covered throughout this section can be integrated into larger real-world computer vision systems.

---

## 🔄 Computer Vision Workflow

The topics in this section progress through several major levels of visual understanding:

**Image Understanding → Detection → Segmentation → Tracking → Transformers → Real-World Applications**

These tasks demonstrate how computer vision systems can progress from understanding individual images to:

1. Detecting objects and their locations
2. Predicting pixel-level information
3. Tracking objects through video sequences
4. Modeling global visual relationships using Transformer architectures
5. Combining multiple techniques into practical end-to-end systems

---

## 🧩 Topics Covered

Throughout this section, you will work with:

* Image Understanding
* Object Detection
* Semantic Segmentation
* Object Tracking
* Multi-Object Tracking
* Video Processing
* Deep Learning
* CNN-Based Architectures
* Encoder-Decoder Networks
* Attention Mechanisms
* Transformer Architectures
* Vision Transformers
* Model Evaluation
* Dataset Preparation
* Data Augmentation
* Real-Time Inference
* Computer Vision Pipelines
* Medical Computer Vision
* Autonomous Driving Perception
* Real-World Computer Vision Projects

---

## 📂 Section Structure

The `04-computer-vision` directory is organized into five major topics:

```text
04-computer-vision/
│
├── 01-object-detection/
│   └── Object detection concepts, algorithms,
│       evaluation metrics, and practical implementations
│
├── 02-segmentation/
│   └── Semantic segmentation concepts,
│       U-Net, DeepLab, and segmentation experiments
│
├── 03-object-tracking/
│   └── Video processing, tracking algorithms,
│       and practical tracking projects
│
├── 04-vision-transformers/
│   └── Vision Transformer concepts,
│       architecture, advantages, and limitations
│
├── 05-projects/
│   ├── colonoscopy-polyp-detection-and-tracking/
│   │   └── YOLO-based polyp detection,
│   │       tracking, and automatic frame capture
│   │
│   └── lane-segmentation-vehicle-tracking/
│       └── Lane segmentation, vehicle detection,
│           and multi-object tracking
│
└── README.md
```

Each topic contains dedicated articles, notebooks, experiments, and supporting materials.

The `05-projects` directory is specifically dedicated to integrating the knowledge from the previous sections into complete computer vision applications.


---

## 🔬 Project 1 — Colonoscopy Polyp Detection & Tracking

The colonoscopy project demonstrates a practical medical-imaging application of object detection and tracking.

### Pipeline

```text
Colonoscopy Video / Live Feed
            │
            ▼
     YOLO11 Detector
            │
            ▼
      Object Tracker
            │
            ├───────────────┐
            │               │
      Polyp Detected?       │
            │               │
           Yes              │
            │               │
            ▼               │
   Save Annotated Frame     │
   with Timestamp           │
            │               │
            └───────────────┘
                    │
                    ▼
          Annotated Output Video
```

The system automatically saves frames in which a polyp is detected, allowing the generated frames to be reviewed afterward.

### Main Concepts

* YOLO11 object detection
* Real-time inference
* Object tracking
* `persist=True`
* Confidence thresholding
* Timestamp-based frame capture
* Annotated video generation

The project also explores domain-specific data augmentation choices for medical imagery.

> **Medical Disclaimer:** The project is intended for research and educational purposes only. It is not a certified medical device and should not be used for clinical diagnosis or treatment decisions.

---

## 🚗 Project 2 — Road Lane Segmentation & Vehicle Tracking

The autonomous-driving project demonstrates how multiple perception models can operate together on driving video.

### Pipeline

```text
                    ┌──────────────────────┐
Driving Video ─────►│ DeepLabV3+ / ResNet50│
                    └──────────┬───────────┘
                               │
                               ▼
                         Lane Segmentation
                               │
                               │
                    ┌──────────▼───────────┐
Driving Video ─────►│       YOLO11         │
                    └──────────┬───────────┘
                               │
                               ▼
                       Vehicle Detection
                               │
                               ▼
                         BoT-SORT / ReID
                               │
                               ▼
                       Persistent IDs
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
        Lane Mask Overlay                Vehicle Boxes
              │                                 │
              └────────────────┬────────────────┘
                               ▼
                       Combined Output Video
```

The project demonstrates the integration of:

* Semantic segmentation
* Object detection
* Multi-object tracking
* Re-identification
* Real-time video processing

---

## 🎯 Learning Outcomes

By completing this section, you will understand the fundamentals and evolution of several major computer vision tasks and architectures.

You will also gain practical knowledge required to:

* Build object detection pipelines
* Prepare and annotate datasets
* Convert datasets between annotation formats
* Evaluate detection models
* Perform pixel-level image segmentation
* Work with segmentation masks
* Build encoder-decoder architectures
* Understand object tracking pipelines
* Track multiple objects across video frames
* Process and analyze video data
* Apply tracking algorithms
* Understand Transformer-based vision models
* Analyze Vision Transformer architectures
* Compare CNN-based and Transformer-based approaches
* Build real-time computer vision pipelines
* Combine multiple models in a single application
* Apply computer vision techniques to real-world problems

---

## 🎯 Learning Path

The recommended learning path for this section is:

### 01 → Object Detection

Understand how objects are located and classified.

⬇️

### 02 → Semantic Segmentation

Move from object-level predictions to pixel-level understanding.

⬇️

### 03 → Object Tracking

Extend visual understanding from individual images to video sequences and persistent object identities.

⬇️

### 04 → Vision Transformers

Explore modern Transformer-based approaches for visual representation and understanding.

⬇️

### 05 → Practical Projects

Combine the learned concepts into complete real-world computer vision applications.

---

## 🚀 Goal

The main goal of this section is to build a strong understanding of **advanced Computer Vision and Deep Learning**, while gradually moving from theoretical concepts to practical end-to-end applications.

The learning process follows this progression:

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
            │
            ▼
End-to-End Vision Pipelines
```

By the end of this section, learners should be able to understand, implement, evaluate, and combine modern computer vision techniques in practical scenarios.

---

## 📚 Project-Based Learning

The `05-projects` directory represents the application layer of this computer vision section.

Instead of focusing on a single algorithm, these projects require combining multiple components:

```text
Dataset
   ↓
Preprocessing
   ↓
Model Training
   ↓
Inference
   ↓
Detection / Segmentation
   ↓
Tracking
   ↓
Video Processing
   ↓
Result Generation
```

This approach helps bridge the gap between individual tutorials and complete computer vision systems.

---


## 📁 Repository Navigation

| Directory                 | Description                                                          |
| ------------------------- | -------------------------------------------------------------------- |
| `01-object-detection/`    | Object detection theory, algorithms, evaluation, and implementations |
| `02-segmentation/`        | Semantic segmentation, U-Net, DeepLab, and related experiments       |
| `03-object-tracking/`     | Video processing, tracking concepts, and tracking implementations    |
| `04-vision-transformers/` | Vision Transformer concepts and modern Transformer-based vision      |
| `05-projects/`            | End-to-end practical computer vision projects                        |

---

## 📖 Recommended Progression

For the best learning experience, follow the repository in this order:

**Detection → Segmentation → Tracking → Transformers → Projects**

The first four sections establish the theoretical and technical foundations, while the fifth section focuses on applying those skills to practical computer vision problems.

---

## 👩‍💻 Author

**Zahra Shahlaie**

Created for learning and educational purposes in:

* Computer Vision
* Deep Learning
* Artificial Intelligence
* Machine Learning
* Real-World AI Applications

---

⭐ **This section is continuously evolving as new computer vision experiments and projects are added.**
