# 🎯 Segmentation with Deep Learning

This repository contains a structured collection of tutorials, articles, and notebooks covering the fundamental concepts and modern deep learning approaches for **Semantic Segmentation** in Computer Vision.

The section starts with the basic concepts of image segmentation and evaluation metrics, then introduces **U-Net** as a fundamental architecture for semantic segmentation. It continues with important concepts such as **Skip Connections** and advanced U-Net architectures, and finally explores **DeepLab** and its approaches for improving segmentation performance.

---

## 📚 Course Structure

### 01 - Introduction to Semantic Segmentation

Learn the fundamental concepts of image segmentation and understand how semantic segmentation differs from other computer vision tasks.

Topics include:

* What is Image Segmentation?
* Image Classification vs Object Detection vs Segmentation
* Semantic Segmentation
* Pixel-wise Classification
* Segmentation Masks
* Multi-class Segmentation
* Binary Segmentation
* Semantic Segmentation Pipeline

---

### 02 - Evaluation Metrics

Learn how semantic segmentation models are evaluated using pixel-level metrics.

Topics include:

* Pixel Accuracy
* Mean Pixel Accuracy (mPA)
* Intersection over Union (IoU)
* Mean Intersection over Union (mIoU)
* Dice Coefficient

---

### 03 - U-Net

Study the **U-Net** architecture and understand why it became one of the most important architectures for biomedical image segmentation and other pixel-level prediction tasks.

Topics include:

* Encoder-Decoder Architecture
* Contracting Path
* Bottleneck
* Expanding Path
* Downsampling
* Upsampling
* Convolutional Layers
* Feature Maps
* Skip Connections
* U-Net Architecture
* Advantages and Limitations of U-Net

---

### 04 - Skip Connection

Understand the concept of **Skip Connections** and how they help preserve spatial information during the segmentation process.

Topics include:

* What is a Skip Connection?
* Information Loss During Downsampling
* Spatial Information
* Encoder-to-Decoder Connections
* Skip Connections in U-Net
* Skip Connection vs Residual Connection
* Why Skip Connections Improve Segmentation

---

### 05 - Advanced U-Net Architectures

Explore improved and extended versions of the original U-Net architecture.

Topics include:

* Limitations of Original U-Net
* Residual U-Net
* Nested U-Net / U-Net++
* Architectural Improvements

---

### 06 - DeepLab

Study the **DeepLab** family of semantic segmentation architectures and understand how they address the limitations of conventional encoder-decoder approaches.

Topics include:

* DeepLab Overview
* Dilated / Atrous Convolution
* Receptive Field
* Atrous Spatial Pyramid Pooling (ASPP)
* Encoder-Decoder Architecture
* DeepLabV3
* DeepLabV3+

---

## 📂 Dataset

The practical notebooks in this section use the **Brain Tumor Segmentation Dataset** from Kaggle for demonstrating the semantic segmentation workflow.

Dataset:

🔗 [Brain Tumor Segmentation Dataset](https://www.kaggle.com/datasets/atikaakter11/brain-tumor-segmentation-dataset)

The dataset is used in the notebooks to demonstrate:

* Image and segmentation mask preparation
* Data preprocessing
* Training semantic segmentation models
* Pixel-wise prediction
* Model evaluation
* Visualization of segmentation results

The dataset is primarily used for the practical implementation of the segmentation concepts and architectures covered in this section.


---

## 🛠️ Requirements

* Python 3.x
* OpenCV
* NumPy
* Matplotlib
* TensorFlow / Keras
* PyTorch

---

## 🚀 Goal

This repository is designed to build a strong understanding of **Semantic Segmentation** and modern deep learning architectures for pixel-level prediction.

By completing this section, you will understand the evolution of semantic segmentation approaches, from fundamental concepts and evaluation metrics to encoder-decoder architectures such as **U-Net**, advanced U-Net variants, and **DeepLab**.

You will also gain practical knowledge required to:

* Prepare segmentation datasets
* Work with segmentation masks
* Build encoder-decoder architectures
* Understand spatial information preservation
* Apply skip connections
* Train semantic segmentation models
* Evaluate pixel-level predictions
* Compare different segmentation architectures

---

## 📌 Author

**Zahra Shahlaie**

Created for learning and educational purposes in **Computer Vision, Deep Learning, and Artificial Intelligence**.
