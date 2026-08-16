# Object Detection

## Introduction

In **Image Classification**, the goal is to determine which class an image belongs to.

For example, suppose we have a dataset containing the following classes:

- 🐶 Dog
    
- 🚗 Car
    
- 🐱 Cat
    

Each image typically has a single **label**, and the model learns to predict the class of a given image.

```text
Input Image
     ↓
   Model
     ↓
 Prediction
     ↓
    Dog
```

Therefore, the main question in Classification is:

> **What is this image?**

For example:

```text
Image → Dog
```

However, Classification does not tell us **where the object is located inside the image**.

This limitation leads us to **Object Localization** and eventually to **Object Detection**.

---

# 1. The Limitation of Image Classification

Consider an image containing a dog:

```text
┌───────────────────────┐
│                       │
│         🐶            │
│                       │
│                       │
└───────────────────────┘

Label → Dog
```

A Classification model can determine that the image belongs to the `Dog` class.

However, it does not provide information about the exact location of the dog.

In other words:

```text
Classification
      ↓
What is in the image?
      ↓
Dog
```

But we also want to know:

```text
Where is the dog?
```

To answer this question, we need **Localization**.

---

# 2. What Is an Object?

In Computer Vision, an **Object** is an entity in an image that we are interested in detecting or analyzing.

Examples include:

- Dog
    
- Car
    
- Person
    
- Cup
    
- Laptop
    
- Bicycle
    

For example, if an image contains a dog, the dog is considered an **Object**.

The goal of Object Detection is to identify these objects and determine their locations in the image.

---

# 3. What Is Object Detection?

**Object Detection** is a Computer Vision task that combines two main objectives:

### 1. Classification

Determine **what the object is**.

```text
What is it?
     ↓
   Dog
```

### 2. Localization

Determine **where the object is located**.

```text
Where is it?
     ↓
Bounding Box
```

Therefore:

```text
                 Object Detection
                        │
             ┌──────────┴──────────┐
             ↓                     ↓
       Classification         Localization
             ↓                     ↓
        What is it?             Where is it?
             ↓                     ↓
            Dog               Bounding Box
```

In simple terms:

> **Object Detection tells us both what an object is and where it is located.**

---

# 4. What Is a Bounding Box?

A **Bounding Box** is a rectangular box drawn around an object to indicate its location and extent within an image.

Conceptually:

```text
┌─────────────────────────────┐
│                             │
│      ┌───────────────┐      │
│      │               │      │
│      │     🐶 Dog    │      │
│      │               │      │
│      └───────────────┘      │
│                             │
└─────────────────────────────┘
```

The Bounding Box answers the question:

> **Where is the object located?**

---

# 5. Bounding Box Representation

There are several ways to represent a Bounding Box.

Two common representations are:

## 5.1 Center-Based Representation

A bounding box can be represented using:

```text
(x, y, w, h)
```

where:

- `x` → horizontal position
    
- `y` → vertical position
    
- `w` → width
    
- `h` → height
    

Conceptually:

```text
        (x, y)
           ●
           │
           │
           └───────────────┐
                           │
                           │
                           └───────────────
```

The exact meaning of `x` and `y` depends on the convention used by the dataset or model. In many formats, they represent the center of the bounding box.

---

## 5.2 Corner-Based Representation

Another common representation uses two opposite corners:

```text
(x1, y1)
     ●───────────────────┐
     │                   │
     │      Object       │
     │                   │
     └───────────────────●
                     (x2, y2)
```

The bounding box is then represented as:

```text
(x1, y1, x2, y2)
```

where:

- `(x1, y1)` → one corner
    
- `(x2, y2)` → opposite corner
    

Both representations are commonly used in Object Detection systems.

---

# 6. Classification vs. Object Detection

The main difference can be summarized as follows.

## Image Classification

Classification predicts the class of the image:

```text
Image
  ↓
Model
  ↓
Label
```

Example:

```text
Image → Dog
```

The model answers:

> **What is the image about?**

---

## Object Detection

Object Detection predicts both the class and location of each detected object:

```text
Image
  ↓
Model
  ↓
┌───────────────────┐
│ Class             │
│ Bounding Box      │
└───────────────────┘
```

Example:

```text
Dog
(x, y, w, h)
```

Therefore:

|Task|Main Output|
|---|---|
|Classification|Class / Label|
|Localization|Object Location|
|Object Detection|Class + Bounding Box|

---

# 7. What Is Annotation?

For an Object Detection model to learn, images alone are not sufficient.

The model needs information describing:

- **What objects are present**
    
- **Where those objects are located**
    

This information is called **Annotation**.

For Classification:

```text
Image → Label
```

For Object Detection:

```text
Image
 ├── Label
 └── Bounding Box
```

For example:

```text
Image: dog_01.jpg

Class:
Dog

Bounding Box:
x = 120
y = 80
w = 250
h = 300
```

Therefore, an Object Detection annotation provides the information required to identify and localize an object.

---

# 8. Training Data and Ground Truth

During training, the model needs correct answers to learn from.

These correct answers are called **Ground Truth**.

For example:

```text
Training Image
      ↓
Ground Truth
      ├── Class: Dog
      └── Bounding Box: (120, 80, 250, 300)
```

The Ground Truth represents the expected output for a training example.

The model produces its own predictions and compares them with the Ground Truth.

```text
Ground Truth
      ↕
Prediction
```

The goal is to make the predictions as close as possible to the Ground Truth.

---

# 9. What Does an Object Detection Model Predict?

Given an input image:

```text
Input Image
     ↓
   Model
```

the model needs to predict information about the detected objects.

In a simplified single-object scenario:

```text
             ┌──→ Classification
Image → Model│        ↓
             │       Class
             │
             └──→ Regression
                      ↓
                 Bounding Box
```

For example:

```text
Prediction:

Class = Dog

Bounding Box = (125, 82, 248, 301)
```

Therefore, Object Detection involves both:

```text
Classification
      +
Bounding Box Regression
```

---

# 10. Why Does Object Detection Use Classification and Regression?

This is one of the fundamental concepts in Object Detection.

## Classification

The model must determine the class of the detected object.

For example:

```text
Dog → 0.90
Cat → 0.05
Car → 0.05
```

The model assigns scores or probabilities to different classes, depending on the architecture and output formulation.

---

## Bounding Box Regression

The model must also predict numerical values describing the object's location and size.

For example:

```text
x = 120
y = 80
w = 250
h = 300
```

Since these are continuous numerical values, predicting the bounding box is commonly formulated as a **Regression** problem.

Therefore, a simplified Object Detection model can be viewed as solving:

```text
Object Detection
       │
       ├── Classification
       │
       └── Bounding Box Regression
```

> The exact formulation differs between modern detection architectures, but this classification + localization view is a useful foundation for understanding the problem.

---

# 11. A Simplified Object Detection Architecture

A basic conceptual architecture can be built around a CNN.

First, the input image is passed through a feature extraction network:

```text
Input Image
     ↓
Convolutional Layers
     ↓
Feature Extraction
     ↓
Feature Maps
```

The extracted features are then used by different prediction heads.

```text
                    ┌──→ Classification Head
                    │          ↓
Feature Maps ───────┤        Class
                    │
                    └──→ Regression Head
                               ↓
                         Bounding Box
```

This gives us two main types of information:

```text
Classification Head → What is it?
Regression Head     → Where is it?
```

---

# 12. Classification Head

The **Classification Head** is responsible for predicting the class of an object.

For example:

```text
Feature Maps
     ↓
Classification Head
     ↓
Dog
```

The output might look like:

```text
Dog → 0.95
Cat → 0.03
Car → 0.02
```

The class with the highest score can then be selected as the predicted class.

---

# 13. Regression Head

The **Regression Head** predicts the numerical parameters of the bounding box.

```text
Feature Maps
     ↓
Regression Head
     ↓
(x, y, w, h)
```

For example:

```text
x = 120
y = 80
w = 250
h = 300
```

These values describe the predicted location and size of the object.

---

# 14. Prediction vs. Ground Truth

During training, we have two important concepts.

### Ground Truth

The correct information provided by the annotations:

```text
True Class
True Bounding Box
```

### Prediction

The information produced by the model:

```text
Predicted Class
Predicted Bounding Box
```

We compare them:

```text
             Ground Truth
                  ↕
               Prediction
```

The closer the prediction is to the Ground Truth, the better the model is performing.

---

# 15. Loss in Object Detection

The model needs a way to measure how wrong its predictions are.

This is done using **Loss Functions**.

In a simplified Object Detection formulation, we can consider at least two major sources of error:

```text
Classification Loss
        +
Bounding Box Loss
```

---

## 15.1 Classification Loss

Classification Loss measures how incorrect the predicted class is.

For example:

```text
Ground Truth:
Dog

Prediction:
Cat
```

The classification error will be relatively high.

---

## 15.2 Bounding Box Loss

Bounding Box Loss measures the difference between the predicted bounding box and the Ground Truth bounding box.

```text
Ground Truth:
(x, y, w, h)

Prediction:
(x', y', w', h')
```

Different Object Detection architectures use different bounding-box loss functions.

Modern detectors often use IoU-based losses or related formulations rather than simply comparing the four coordinates independently.

---

# 16. Total Detection Loss

Conceptually, the total loss can be represented as:

```text
Total Loss =
Classification Loss
+
Bounding Box Loss
```

Or using weighting factors:

```text
Total Loss =
λ₁ × Classification Loss
+
λ₂ × Bounding Box Loss
```

where `λ₁` and `λ₂` control the relative contribution of each component.

The training process attempts to minimize the overall detection loss.

> In real-world detectors, the total loss may contain additional components, such as objectness or confidence loss, depending on the architecture.

---

# 17. Object Detection Training Process

The complete training process can be summarized as follows.

### Step 1 — Input Image

```text
Training Image
      ↓
     Model
```

### Step 2 — Feature Extraction

```text
Model
  ↓
Feature Maps
```

### Step 3 — Prediction

The model predicts class and bounding box information:

```text
Feature Maps
     │
     ├──→ Classification
     │       ↓
     │     Class
     │
     └──→ Regression
             ↓
        Bounding Box
```

### Step 4 — Compare With Ground Truth

```text
Prediction
     ↕
Ground Truth
```

### Step 5 — Calculate Loss

```text
Classification Loss
        +
Bounding Box Loss
        ↓
   Total Loss
```

### Step 6 — Backpropagation

The loss is propagated backward through the network:

```text
Total Loss
    ↓
Backpropagation
    ↓
Gradients
```

### Step 7 — Update Weights

The optimizer uses the gradients to update the model parameters:

```text
Gradients
    ↓
Optimizer
    ↓
Updated Weights
```

The process is repeated over many training iterations and epochs.

```text
Image
  ↓
Prediction
  ↓
Loss
  ↓
Backpropagation
  ↓
Weight Update
  ↓
Better Prediction
  ↓
...
```

---

# 18. Training Pipeline Summary

The entire training process can be visualized as:

```text
                    TRAINING

                 Training Image
                       ↓
                Feature Extraction
                       ↓
                   Feature Maps
                    ↙         ↘
                   ↓           ↓
          Classification    Regression
                   ↓           ↓
                 Class      Bounding Box
                    ↘         ↙
                       ↓
                Compare with
                 Ground Truth
                       ↓
                  Calculate Loss
                       ↓
                 Backpropagation
                       ↓
                    Optimizer
                       ↓
                  Update Weights
                       ↓
                     Repeat
```

---

# 19. Inference: Using a Trained Model

After training, the model no longer needs Ground Truth for prediction.

We provide a new image:

```text
New Image
    ↓
Trained Model
```

The model produces predictions such as:

```text
Class = Dog

Bounding Box =
(x, y, w, h)
```

We can then visualize the result by drawing the predicted bounding box on the image.

For example:

```text
┌──────────────────────────────┐
│                              │
│      ┌──────────────┐        │
│      │     DOG      │        │
│      │      🐶      │        │
│      └──────────────┘        │
│                              │
└──────────────────────────────┘
```

The final detection result contains information such as:

- Object Class
    
- Bounding Box
    
- Confidence Score
    

---

# 20. What Is Object Localization?

Before dealing with multiple objects, it is useful to understand **Object Localization**.

Suppose an image contains a single object:

```text
Image
  ↓
One Object
  ↓
Class + Bounding Box
```

The task is to:

1. Identify the object.
    
2. Determine its location.
    

Conceptually:

```text
One Object
    ↓
Class + Location
```

This provides a useful stepping stone toward understanding Object Detection.

---

# 21. Localization vs. Object Detection

The two concepts are closely related, but they are commonly distinguished by the number of objects being handled.

### Object Localization

Focuses on identifying and locating a **single main object**.

```text
One Object
    ↓
Class + Location
```

### Object Detection

Handles **multiple objects** and predicts the class and location of each one.

```text
Multiple Objects
       ↓
Class + Location
       ↓
Object 1 → Box 1
Object 2 → Box 2
Object 3 → Box 3
...
```

This distinction is especially useful when learning the evolution of Object Detection algorithms.

---

# 22. Multi-Object Detection

Now consider an image containing multiple objects:

```text
┌──────────────────────────────┐
│                              │
│   🐶              🚗         │
│                              │
│        ☕                    │
│                              │
└──────────────────────────────┘
```

The image contains:

```text
Dog
Car
Cup
```

The detector should identify each object and predict its corresponding bounding box:

```text
Dog → Class + Bounding Box
Car → Class + Bounding Box
Cup → Class + Bounding Box
```

Therefore, the number of predictions is not fixed to one.

Instead:

```text
Object 1 → Class + Box
Object 2 → Class + Box
Object 3 → Class + Box
...
Object N → Class + Box
```

This is the core idea behind **Multi-Object Detection**.

---

# 23. Why Is Object Detection More Difficult Than Classification?

In Classification, the main output is typically one class for the image:

```text
Image
  ↓
Model
  ↓
Dog
```

Object Detection is more challenging because the model must determine:

```text
What objects are present?
          +
Where are they located?
          +
How many objects are there?
```

Therefore:

```text
Image
  ↓
Find Objects
  ↓
Classify Objects
  ↓
Localize Objects
  ↓
Predict Bounding Boxes
  ↓
Filter Final Detections
```

The model must handle objects at different positions, scales, and sometimes overlapping locations.

---

# 24. Popular Object Detection Algorithms

Over the years, many Object Detection architectures have been developed.

Some important examples include:

- **R-CNN**
    
- **Fast R-CNN**
    
- **Faster R-CNN**
    
- **SSD**
    
- **YOLO**
    

These methods use different strategies for generating object predictions and bounding boxes.

For example:

### R-CNN Family

The R-CNN family introduced a region-based approach to Object Detection.

```text
Image
  ↓
Candidate Regions
  ↓
Feature Extraction
  ↓
Classification + Localization
```

### SSD

**SSD (Single Shot MultiBox Detector)** performs detection in a single network pass and predicts objects at multiple feature-map scales.

```text
Image
  ↓
CNN
  ↓
Multi-Scale Feature Maps
  ↓
Predictions
  ↓
Objects + Bounding Boxes
```

### YOLO

**YOLO (You Only Look Once)** approaches Object Detection as a unified prediction problem and is well known for its speed and suitability for real-time applications.

```text
Image
  ↓
YOLO
  ↓
Object Predictions
  ↓
Class + Bounding Box + Confidence
```

These architectures will be studied in more detail in later sections.

---

# 25. Key Concepts

At this point, the most important concepts are:

|Concept|Meaning|
|---|---|
|**Object**|An entity of interest in an image|
|**Classification**|Determines what the image or object is|
|**Localization**|Determines where an object is|
|**Bounding Box**|Represents the spatial extent of an object|
|**Annotation**|Ground-truth information assigned to an image|
|**Ground Truth**|Correct target values used during training|
|**Prediction**|Output generated by the model|
|**Regression**|Predicts continuous bounding-box values|
|**Classification Head**|Predicts object classes|
|**Regression Head**|Predicts bounding-box parameters|
|**Loss**|Measures prediction error|
|**Backpropagation**|Computes gradients for learning|
|**Optimizer**|Updates model parameters|
|**Inference**|Using a trained model to make predictions|

---

# 26. Classification vs. Localization vs. Detection

A useful way to understand the progression is:

```text
Classification
      │
      │
      ▼
What is in the image?
      │
      │
      ▼
Localization
      │
      │
      ▼
What is it + Where is it?
      │
      │
      ▼
Object Detection
      │
      │
      ▼
What objects are present
and where is each one?
```

Or more simply:

```text
Classification
      ↓
"What?"

Localization
      ↓
"What + Where?"

Object Detection
      ↓
"What + Where?"
for multiple objects
```

---

# 27. Learning Path

A useful learning path for Object Detection is:

```text
Image Classification
        ↓
Understanding Objects
        ↓
Object Localization
        ↓
Bounding Boxes
        ↓
Annotations
        ↓
Classification + Regression
        ↓
Object Detection
        ↓
Multi-Object Detection
        ↓
R-CNN
        ↓
Fast R-CNN
        ↓
Faster R-CNN
        ↓
SSD
        ↓
YOLO
```

This progression makes it easier to understand why modern Object Detection models were developed and what problems each generation attempted to solve.

---

# 28. Final Summary

**Image Classification** answers:

> **What is this image?**

For example:

```text
Image → Dog
```

**Object Localization** extends this idea by determining both the class and the location of a single object:

```text
Dog + Bounding Box
```

**Object Detection** goes one step further and identifies multiple objects together with their locations:

```text
Dog → Box
Car → Box
Cup → Box
```

Therefore, the core idea of Object Detection can be summarized as:

```text
                Object Detection
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
       Classification       Localization
             ↓                   ↓
         What is it?          Where is it?
             │                   │
             └─────────┬─────────┘
                       ↓
             Class + Bounding Box
```

During training, the model learns from annotated images by comparing its predictions with the Ground Truth and minimizing the detection loss.

```text
Training Image
      ↓
Feature Extraction
      ↓
Predictions
      ↓
Compare with Ground Truth
      ↓
Loss
      ↓
Backpropagation
      ↓
Optimizer
      ↓
Updated Weights
      ↓
Repeat
```

The fundamental idea is:

> **Classification tells us what is in an image, while Object Detection tells us what objects are present and where they are located.**

This foundation leads naturally to more advanced detection architectures such as **R-CNN, Faster R-CNN, SSD, and YOLO**.
