# 👁️ Advanced Computer Vision

This section contains a structured collection of tutorials, articles, notebooks, experiments, implementations, and practical projects covering modern **Computer Vision and Deep Learning**.

The learning path progresses from fundamental computer vision tasks to modern deep learning architectures, YOLO-based object detection, and real-world computer vision applications.

---

## 📚 Course Structure

### 01 - Object Detection

Learn the fundamental concepts and algorithms used to detect and localize objects in images.

Topics include:

- Object Detection Fundamentals
- Classification vs Localization vs Detection
- Bounding Boxes
- Confidence Scores
- Dataset Annotation
- IoU
- Precision and Recall
- Average Precision (AP)
- Mean Average Precision (mAP)
- R-CNN
- Fast R-CNN
- Faster R-CNN
- SSD
- YOLOv1
- YOLOv2
- YOLOv3

This section covers the evolution of object detection from two-stage detectors to fast single-stage approaches.

---

### 02 - Semantic Segmentation

Learn how deep learning models perform pixel-level prediction and generate segmentation masks.

Topics include:

- Semantic Segmentation Fundamentals
- Pixel-wise Classification
- Binary Segmentation
- Multi-class Segmentation
- Pixel Accuracy
- Mean Pixel Accuracy (mPA)
- IoU
- Mean IoU (mIoU)
- Dice Score
- U-Net
- Encoder-Decoder Architecture
- Skip Connections
- U-Net++
- Residual U-Net
- DeepLab
- Atrous Convolution
- Atrous Spatial Pyramid Pooling (ASPP)
- DeepLabV3
- DeepLabV3+

Practical notebooks cover dataset preparation, mask processing, model training, inference, and evaluation.

---

### 03 - Object Tracking

Learn how objects can be detected and tracked across consecutive video frames.

Topics include:

- Video Processing with OpenCV
- Object Detection vs Tracking
- Single Object Tracking
- Multi-Object Tracking
- Tracking-by-Detection
- Bounding Box Tracking
- Feature-Based Tracking
- Deep Learning-Based Tracking
- Persistent Object IDs
- Multi-Object Tracking
- Tracking Evaluation

This section includes practical video-based tracking experiments and applications.

---

### 04 - Vision Transformers

Explore Transformer-based architectures for computer vision and understand how Vision Transformers process visual information.

Topics include:

- CNNs vs Transformers
- Image Patches
- Patch Embeddings
- Positional Embeddings
- Class Token
- Transformer Encoder
- Self-Attention
- Multi-Head Self-Attention
- Vision Transformer (ViT)
- Global Context Modeling
- Advantages and Limitations of ViTs
- Computational Complexity
- Data Requirements

This section focuses on the transition from convolution-based architectures to Transformer-based visual representation learning.

---

### 05 - YOLO

A dedicated section for studying the **YOLO (You Only Look Once)** family of real-time object detection models.

Topics include:

- YOLO Fundamentals
- YOLO Architecture
- YOLO Detection Pipeline
- YOLO Dataset Structure
- Annotation Formats
- Training and Validation
- Inference
- Model Evaluation
- Confidence and IoU Thresholds
- mAP
- Real-Time Object Detection
- YOLO Model Variants
- YOLO-based Tracking
- YOLO Deployment
- Practical YOLO Experiments

This section contains dedicated tutorials, notebooks, experiments, and implementations related to YOLO-based computer vision systems.

---

### 999 - Projects

A collection of practical and application-oriented computer vision projects that combine concepts from the previous sections.

Projects may integrate multiple computer vision tasks, deep learning models, video processing pipelines, tracking algorithms, and real-time inference.

#### 🔬 Colonoscopy Polyp Detection & Tracking

A YOLO11-based pipeline for detecting and tracking polyps in colonoscopy video.

Key components:

- YOLO11 Object Detection
- Real-Time Video Inference
- Object Tracking
- Persistent Tracking IDs
- Automatic Frame Capture
- Timestamp-Based Filenames
- Annotated Output Video

Detected frames are automatically saved for later review.

> **Note:** This is a research/educational project and is not a certified medical device. It does not provide medical diagnosis.

#### 🚗 Road Lane Segmentation & Vehicle Detection/Tracking

An autonomous-driving perception pipeline combining:

- DeepLabV3+ with ResNet50
- YOLO11 Object Detection
- BoT-SORT Multi-Object Tracking
- Re-Identification (ReID)
- Real-Time Video Processing
- Lane Segmentation
- Vehicle Detection
- Vehicle Tracking
- Visualization

The pipeline combines lane segmentation, vehicle detection, and tracking into a unified video-processing system.

---

## 🔄 Computer Vision Learning Workflow

The topics progress from fundamental concepts to specialized models and practical applications:

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
           YOLO
            │
            ▼
     Practical Projects
```
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
