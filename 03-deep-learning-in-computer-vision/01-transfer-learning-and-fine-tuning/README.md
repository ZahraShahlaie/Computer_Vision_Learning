
# Transfer Learning, Pre-trained Models & Fine-Tuning in Keras

In earlier projects, image classification models were built **from scratch**: we designed the architecture, fed it data, and let it learn everything on its own.

This approach usually performs poorly on small datasets, because a large part of what a network learns is **feature extraction**, and learning good features requires a huge amount of data.

For this reason, real-world projects almost always rely on **pre-trained models** instead.

---

## Training From Scratch

Suppose we want to classify images of animals. A typical CNN architecture looks like this:

```text
Image
   │
Convolution Layers
   │
Feature Extraction
   │
Flatten
   │
Dense
   │
Softmax
   │
Prediction
```

Here:

- Convolutional layers extract features from the image.
- Dense layers handle the classification task.

So the network has two main jobs:

1. Extracting features
2. Classifying images

---

## Why Is Training From Scratch Hard?

Feature extraction is not a simple task. For a network to learn how to recognize:

- Edges
- Textures
- Shapes
- Different objects

it needs to see millions of images.

Training a network from scratch therefore usually requires:

- A very large dataset
- Long training time
- Powerful hardware
- A well-designed architecture

Most projects simply don't have these resources.

---

## What Is a Pre-trained Model?

A **pre-trained model** is a model that has already been trained on a very large dataset. These models have already learned how to extract meaningful features from images, so we don't need to redo that process from zero.

---

## The Core Idea of Transfer Learning

Instead of training a network from scratch ourselves, we reuse the knowledge a model has already learned.

```text
New Images

        │

Pre-trained CNN

        │

Feature Maps

        │

Dense Layer

        │

Prediction
```

Here:

- The pre-trained model only extracts features.
- We only train the classification part for our own problem.

---

## What Is Feature Extraction?

The most important part of a CNN is feature extraction. Early layers learn simple features such as:

- Edges
- Lines
- Corners
- Colors

Deeper layers extract more complex features, such as:

- Eyes
- Noses
- Car wheels
- Bird wings
- Building windows

Finally, the dense layer uses these features to make a decision.

---

## How Does Transfer Learning Work?

Suppose ResNet has already been trained on millions of images, and we only have 5,000 images of cats and dogs.

Instead of training the whole network:

- We use ResNet only to extract features.
- We add a new dense layer for our two classes.

As a result:

- Training is faster.
- Less data is required.
- Accuracy is usually higher.

---

## What Is ImageNet?

Almost all well-known models in the world have been trained on the **ImageNet** dataset.

Key facts about this dataset:

- More than **14 million images**
- About **1,000 different classes**
- Includes animals, objects, plants, humans, and more

Because of this huge diversity, models trained on ImageNet are usually very good at extracting features.

---

## Keras Applications

The Keras library provides a collection of ready-to-use models, including some of the most popular ones:

- VGG16
- VGG19
- ResNet50
- ResNet50V2
- InceptionV3
- Xception
- DenseNet
- MobileNet
- NASNet
- EfficientNet
- EfficientNetV2

All of these models can be downloaded and used with just a few lines of code.

---

## Criteria for Comparing Models

The Keras Applications table provides several pieces of information for each model. The most important ones are:

- **Model Size**: The size of the model file after training. More parameters generally mean a larger model.
- **Parameters**: The number of trainable parameters in the network. More parameters don't always mean a better model — sometimes newer models with fewer parameters perform much better.
- **Top-1 Accuracy**: In classification, the model outputs a probability for each class. Top-1 accuracy checks whether the **highest probability** belongs to the correct class.
- **Top-5 Accuracy**: Since ImageNet has a large number of classes (1,000), another metric is used. Top-5 accuracy checks whether the correct answer is among the model's **top five predicted probabilities**.

### Why Does Top-5 Matter?

Sometimes two classes are very similar, such as:

- Two dog breeds
- Two bird species
- Two car models

The model might assign the highest probability to the wrong class, while the correct class still appears among the top five. That's why Top-5 accuracy is a more suitable metric for comparing models on ImageNet.

---

## Choosing the Right Model

When choosing a pre-trained model, several factors should be considered:

- Accuracy
- Number of parameters
- Model size
- CPU inference speed
- GPU inference speed
- Memory usage

Sometimes a model with fewer parameters is:

- Faster
- Less memory-intensive
- More accurate

So the largest model is not always the best choice.

---

## Using a Pre-trained Model in Keras

Suppose we want to use EfficientNetV2. First, we load it from Keras:

```python
from tensorflow.keras.applications import EfficientNetV2B0
```

---

## The `weights` Parameter

The most important parameter when loading a model is **weights**.

### Option 1

```python
weights=None
```

In this case:

- Only the architecture is loaded.
- Weights are randomly initialized.
- The model has no prior training.

### Option 2

```python
weights="imagenet"
```

In this case:

- Weights trained on ImageNet are loaded.
- The model already knows how to extract image features.

This is the most common way to apply transfer learning.

---

## What Is `include_top`?

Pre-trained models usually have a classification layer at the end, trained to predict the 1,000 ImageNet classes. The `include_top` parameter determines whether we need this layer.

### `include_top=True`

- The entire network is loaded.
- The ImageNet classification layer is included.
- Useful when we want to predict exactly the same 1,000 ImageNet classes.

### `include_top=False`

- Only the feature-extraction part is loaded.
- The classification layer is removed.
- This is the most common choice in real-world projects, since we add our own dense layer suited to our problem.

---

## Preprocessing Images

Each pre-trained model was trained with a specific type of preprocessing. For example:

- Some models normalize images between 0 and 1.
- Some normalize between -1 and 1.
- Some subtract the mean of the RGB channels.

For this reason, Keras provides a dedicated preprocessing function for each model:

```python
preprocess_input()
```

Always use the preprocessing function that matches the specific model you're using.

---

## Freezing the Model

Sometimes we only want to use the model for feature extraction, meaning its weights should not change. This is called **freezing the model**.

```python
layer.trainable = False
```

In this case:

- The model's weights stay fixed.
- Only the new dense layer is trained.

---

## What Is Fine-Tuning?

Sometimes our data is very different from ImageNet, for example:

- Medical images
- Satellite images
- Industrial images
- Face recognition

In such cases, it's better to retrain part of the model. This process is called **fine-tuning**.

---

## Two Fine-Tuning Approaches

### 1. Training the Entire Network

All layers of the model are retrained.

- **Pro**: Highest accuracy
- **Con**: Long training time, requires more data

### 2. Training Part of the Network

- Early layers are frozen.
- Only the final few layers are retrained.

This is the most common fine-tuning approach.

---

## Why Only Train the Final Layers?

Early layers extract roughly the same features across almost any image, such as:

- Edges
- Lines
- Colors

Final layers, however, learn more specialized features. So we only update that part of the network for the new problem.

---

## Building the Final Model

After loading the pre-trained model, we just need to add a few layers on top:

```text
Input Image

↓

Pre-trained Model

↓

Feature Maps

↓

Flatten

↓

Dense

↓

Softmax

↓

Prediction
```

The model is then trained just like any other Keras model:

```python
model.compile(...)

model.fit(...)

model.evaluate(...)
```

There's no difference from working with any other Keras model.

---

## Summary

In real-world projects, neural networks are usually not trained from scratch, since learning to extract features requires huge amounts of data and time. Instead, **pre-trained models** — trained on large datasets like **ImageNet** — are used.

In **transfer learning**, we use these models as feature extractors and only train the classification part for our new problem. If the model itself needs to adapt to the new data, we use **fine-tuning**, retraining part or all of the model's weights.

In short:

- **Pre-trained Model**: A model already trained on a large dataset.
- **Transfer Learning**: Reusing a pre-trained model's knowledge to solve a new problem.
- **Feature Extraction**: Using the model only to extract features.
- **Fine-Tuning**: Updating a pre-trained model's weights to improve accuracy on new data.

These techniques are the foundation of many modern computer vision applications, including image classification, face recognition, medical imaging, satellite imagery, structural damage detection, and many industrial use cases.
