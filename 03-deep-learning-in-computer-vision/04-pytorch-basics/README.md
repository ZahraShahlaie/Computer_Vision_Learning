# Introduction to PyTorch

PyTorch is an open-source deep learning framework developed by Meta (Facebook AI).

The library gives developers full capabilities for:

- Building neural networks
- Training models
- Image processing
- Text processing
- Audio processing
- GPU computation

Today, many research papers and academic projects rely on PyTorch.

---

## TensorFlow or PyTorch?

Both frameworks are extremely powerful, and almost any deep learning model can be implemented with either one.

In general:

### TensorFlow

- Well suited for industrial projects
- Plenty of ready-made tools
- Easier deployment
- Good integration with Keras

### PyTorch

- Simpler, more Pythonic code
- Greater flexibility
- Popular in academic research
- Widely used in recent papers

In practice, learning both frameworks is a valuable asset for any AI practitioner.

---

## Installing PyTorch

On PyTorch's official website, an installation tool generates the right command based on your operating system and hardware.

To install, you need to specify:

- Operating system
- Stable or preview (nightly) build
- Package manager (e.g., pip)
- Programming language
- CUDA version (if using a GPU)

The site then generates the appropriate install command. Example:

```bash
pip install torch torchvision torchaudio
```

---

## Stable or Nightly?

There are two options when installing:

### Stable

- Well tested
- No major bugs
- Suitable for real-world projects

### Nightly

- Includes the newest features
- May be unstable
- Suited to PyTorch contributors/developers

For most projects, the **Stable** build is always recommended.

---

## Core PyTorch Libraries

PyTorch itself only provides the core computation engine. Additional libraries are provided for different domains. The most important ones are:

| Library     | Purpose                     |
| ----------- | ---------------------------- |
| torch       | Core deep learning engine    |
| torchvision | Image processing             |
| torchaudio  | Audio processing              |
| torchtext   | Text processing               |

In computer vision projects, we typically use **torchvision**.

---

## Why Use Separate Virtual Environments?

It's recommended to use two separate virtual environments for TensorFlow and PyTorch. Reasons include:

- Avoiding version conflicts
- Avoiding CUDA incompatibilities
- Avoiding dependency issues
- Easier project management

For example:

```text
tensorflow_env
```

and

```text
pytorch_env
```

---

## Importing the Libraries

At the start of a project, we import the libraries we need:

```python
import torch

from torch import nn

import torchvision

from torchvision import transforms

import matplotlib.pyplot as plt
```

Each of these libraries has a specific role.

---

## Role of Each Library

- **torch**: The core of PyTorch
- **nn**: All tools related to neural networks
- **torchvision**: Loading datasets and image processing
- **transforms**: Transforming images, such as:
  - Converting to a Tensor
  - Normalization
  - Resizing
  - Data augmentation

---

## Loading the MNIST Dataset

One of the ready-made datasets available in torchvision is the well-known **MNIST** dataset, which contains images of handwritten digits.

To load it, we use the following class:

```python
torchvision.datasets.MNIST(...)
```

---

## Important Parameters When Loading a Dataset

- **root**: The path where the dataset is stored.

```python
root="dataset"
```

If the dataset doesn't already exist, it will be downloaded to this path.

- **train**: Specifies which part of the dataset to load.

```python
train=True
```

Training data, or:

```python
train=False
```

Test data.

- **download**: Downloads the file if it doesn't already exist.

```python
download=True
```

- **transform**: Specifies what processing to apply to the image after it's loaded.

---

## `transforms.ToTensor()`

One of the most commonly used transforms in PyTorch.

```python
transforms.ToTensor()
```

This transform does two important things:

### 1. Converts the image to a Tensor

PyTorch only works with Tensors.

### 2. Normalizes the image

Pixel values are converted from the range:

```text
0 to 255
```

to the range:

```text
0 to 1
```

---

## Creating Training and Test Datasets

We typically create two datasets:

```text
Train Dataset
```

for training the model, and:

```text
Test Dataset
```

for evaluating the model.

---

## Data Structure in PyTorch

One of the important differences between PyTorch and TensorFlow is how images are stored.

### TensorFlow

Image dimension order:

```text
Height × Width × Channel
```

or

```text
H × W × C
```

Example:

```text
224 × 224 × 3
```

### PyTorch

Dimension order:

```text
Channel × Height × Width
```

or

```text
C × H × W
```

Example:

```text
3 × 224 × 224
```

This difference should always be kept in mind.

---

## The Structure of Each Sample in a Dataset

Each sample inside a Dataset is a tuple:

```text
(Image, Label)
```

So if we write:

```python
image, label = train_dataset[10]
```

then:

- The image is stored in the `image` variable
- The label is stored in the `label` variable

---

## Viewing Image Dimensions

To see the image's dimensions, we use:

```python
image.shape
```

For the MNIST dataset, the output looks like this:

```text
1 × 28 × 28
```

The first number represents the number of channels. Since MNIST images are grayscale, they only have one channel.

---

## What Is a Label?

A **label** is simply the correct class of the image. For example:

```text
0
```

means the image corresponds to the digit zero.

---

## Removing the Extra Dimension with `squeeze()`

To display an image with Matplotlib, the extra channel dimension needs to be removed. We use:

```python
image.squeeze(0)
```

The output will be:

```text
28 × 28
```

---

## What Is a DataLoader?

So far, we've only read the dataset. But a neural network can't process the entire dataset at once, so the data is split into smaller groups, called **batches**.

The class responsible for this is:

```python
DataLoader
```

---

## Creating a DataLoader

```python
train_loader = torch.utils.data.DataLoader(...)
```

This class takes the dataset and splits it into small batches.

- **Batch Size**: One of the most important DataLoader parameters.

```python
batch_size = 64
```

This means 64 images are fed into the network at each training step.

- **Shuffle**: For training data, we usually use:

```python
shuffle=True
```

This randomizes the order of images at every epoch. Benefits include:

- Prevents the model from learning the data order
- Improves generalization
- Reduces the risk of overfitting

For test data, we usually set:

```python
shuffle=False
```

since there's no need to shuffle the order.

---

## Batch Management: TensorFlow vs. PyTorch

In TensorFlow, batch size is usually specified while training the model:

```python
model.fit(...)
```

In PyTorch, batches are created by the DataLoader, and the network receives these batches directly. This gives PyTorch more flexibility in how data is managed.

---

## Building a Neural Network in PyTorch

In PyTorch, all neural network layers live inside the `torch.nn` module. First, we import it:

```python
from torch import nn
```

The word **nn** stands for **Neural Network**.

---

## Building a Sequential Model

The simplest way to build a network in PyTorch is with `Sequential`:

```python
model = nn.Sequential(
    ...
)
```

This structure is very similar to Keras — the output of each layer becomes the input to the next.

---

## Adding the First Convolutional Layer

For images, we use convolutional layers:

```python
nn.Conv2d(...)
```

The main parameters of this layer are:

```python
nn.Conv2d(
    in_channels,
    out_channels,
    kernel_size,
    padding
)
```

### 1) in_channels

Specifies the number of input channels of the image. For the MNIST dataset:

```
28 × 28 × 1
```

The images are grayscale, so:

```python
in_channels = 1
```

### 2) out_channels

The number of output feature maps — essentially the number of kernels in this layer. For example:

```python
out_channels = 16
```

means we'll have 16 kernels. As a result, the output will look like:

```
28 × 28 × 16
```

(assuming appropriate padding is used).

### 3) kernel_size

Specifies the kernel size. For example:

```python
kernel_size = 3
```

means a 3×3 kernel, the most common choice in CNNs.

### 4) padding

Specifies whether zeros are added around the image. For example:

```python
padding = 1
```

preserves the image size.

---

## Why Isn't the Activation Included in Conv?

In Keras, we would usually write:

```python
Conv2D(..., activation="relu")
```

In PyTorch, activation is usually defined as a separate layer:

```python
nn.Conv2d(...)
nn.ReLU()
```

This is because in PyTorch, each step is defined independently — first **Convolution**, then **Activation**.

---

## Adding ReLU

After each Conv layer, we typically use ReLU:

```python
nn.ReLU()
```

This is the same activation function we've seen before.

---

## Adding MaxPooling

After a Conv layer, we usually reduce the image size using:

```python
nn.MaxPool2d()
```

Typically we write:

```python
nn.MaxPool2d(2)
```

which means 2×2 pooling. As a result:

```
28×28
↓

14×14
```

---

## Second Convolutional Layer

Now we add another layer. This time, its input equals the previous layer's output, so:

```python
in_channels = 16
```

and we usually increase the number of channels, for example:

```python
out_channels = 32
```

So the network path looks like this:

```
Input
↓

Conv(16)

↓

ReLU

↓

MaxPool

↓

Conv(32)

↓

ReLU

↓

MaxPool
```

---

## Why Does the Number of Channels Increase?

The deeper we go, the more complex features the network extracts. So the number of feature maps usually increases, for example:

```
1

↓

16

↓

32

↓

64

↓

128
```

---

## Entering the Fully Connected Part

Up to this point, we've produced feature maps. But for classification, we need to feed them into a fully connected network. First, we need to flatten them.

In PyTorch:

```python
nn.Flatten()
```

Suppose the output of the Conv layers is:

```
7 × 7 × 32
```

Flattened, it becomes:

```
7 × 7 × 32

=

1568
```

meaning **1568 features**.

---

## Fully Connected Layer

In PyTorch, instead of Dense, the layer used is:

```python
Linear
```

Its structure:

```python
nn.Linear(
    in_features,
    out_features
)
```

For example:

```python
nn.Linear(7*7*32,128)
```

meaning:

```
1568 features

↓

128 neurons
```

We then add another ReLU:

```python
nn.ReLU()
```

---

## Output Layer

At the end of the network, the number of outputs must equal the number of classes. For MNIST, there are:

```
10 classes
```

So:

```python
nn.Linear(128,10)
```

---

## Should We Add Softmax?

No.

This is one of PyTorch's most important differences. In Keras, we would usually write:

```python
Dense(10,activation="softmax")
```

But in PyTorch, if we're going to use:

```python
CrossEntropyLoss
```

we should **not** include Softmax inside the model. Why? Because `CrossEntropyLoss` applies Softmax internally.

If we add Softmax again:

- The output becomes incorrect.
- We get a warning.
- Training is disrupted.

So the model's output should simply be:

```python
Linear(128,10)
```

---

## Viewing the Model's Structure

In Keras, we had:

```python
model.summary()
```

PyTorch has no such built-in method. To view a summary, we use the `torchsummary` package.

Install it:

```bash
pip install torchsummary
```

Then:

```python
from torchsummary import summary
```

and:

```python
summary(model,(1,28,28))
```

The input is:

```
Channel
Height
Width
```

Remember that in PyTorch, the dimension order is:

```
(C,H,W)
```

---

## Defining the Loss

For classification problems, we use:

```python
nn.CrossEntropyLoss()
```

```python
criterion = nn.CrossEntropyLoss()
```

This function performs both:

- Softmax
- Cross entropy

at the same time.

---

## Defining the Optimizer

In PyTorch, optimizers live inside:

```python
torch.optim
```

For example:

```python
optimizer = torch.optim.Adam(
    model.parameters(),
    lr=0.001
)
```

An important detail:

```python
model.parameters()
```

registers all the network's weights with the optimizer so it can update them.

---

## A Big Difference From Keras

In Keras, all we needed was:

```python
model.fit(...)
```

But in PyTorch, there's no built-in training method — we have to write the training loop ourselves.

---

## Steps of the Training Process

For each epoch, the following steps take place:

```
Data

↓

Model

↓

Prediction

↓

Loss

↓

Backward

↓

Optimizer Step

↓

Update Weights
```

---

## Putting the Model in Training Mode

Before training, we need to write:

```python
model.train()
```

This puts layers like:

- Dropout
- BatchNorm

into training mode.

---

## Iterating Over Batches

We use the DataLoader:

```python
for images, labels in train_loader:
```

Each iteration gives us 64 images and 64 labels.

---

## Making a Prediction

```python
outputs = model(images)
```

The model receives the images and predicts an output.

---

## Computing the Loss

```python
loss = criterion(outputs,labels)
```

This function computes the difference between the model's output and the true labels.

---

## Zeroing the Gradients

Before every batch, we need to write:

```python
optimizer.zero_grad()
```

Because in PyTorch, gradients accumulate by default, and if they're not cleared, training will produce incorrect results.

---

## Backpropagation

Next, gradients need to be computed:

```python
loss.backward()
```

This is the backpropagation algorithm.

---

## Updating the Weights

Finally:

```python
optimizer.step()
```

updates the network's weights.

---

## Computing Loss for Each Epoch

We get one loss value per batch. To compute the total loss for an epoch, first:

```python
running_loss = 0
```

then:

```python
running_loss += loss.item()
```

At the end of the epoch, we usually take the average:

```python
running_loss / len(train_loader)
```

---

## Computing Accuracy

To compute accuracy, we can use the `torchmetrics` library.

Install it:

```bash
pip install torchmetrics
```

Then:

```python
from torchmetrics.classification import Accuracy
```

For a multi-class problem:

```python
metric = Accuracy(
    task="multiclass",
    num_classes=10
)
```

---

## Extracting the Predicted Class

The model's output doesn't contain each class's probability — it returns **logits**. To get the final predicted class, we use:

```python
torch.argmax(outputs,dim=1)
```

This returns the index of the largest value, which is the predicted class.

---

## Updating Accuracy

For each batch, we call:

```python
metric.update(preds,labels)
```

At the end of the epoch:

```python
metric.compute()
```

returns the overall accuracy.

---

## Resetting the Metric

Before starting a new epoch, we need to write:

```python
metric.reset()
```

Otherwise, values from the previous epoch will remain part of the calculation.

---

## Displaying Results

At the end of each epoch, we usually print:

- The epoch number
- The loss value
- The accuracy value

For example:

```
Epoch 1

Loss : 0.085

Accuracy : 98.3%
```

This lets us track the model's learning progress during training.

---

## Key Differences Between PyTorch and TensorFlow

| TensorFlow / Keras                                | PyTorch                                                                          |
| --------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `model.fit()` handles training                     | The training loop must be written manually                                     |
| Softmax is usually defined inside the model         | Softmax should not be added inside the model when using `CrossEntropyLoss`      |
| `model.summary()` is available                     | `torchsummary` is used to view the model's structure                            |
| Many steps happen automatically                     | Every step — forward pass, loss, backward, update — is controlled manually      |

---

## Evaluating the Model

In machine learning, high accuracy on training data alone isn't enough. The real goal is for the model to perform well on data it hasn't seen before. That's why we usually split the data into two parts:

- **Train Dataset** → for training the model
- **Test Dataset** → for evaluating the model

---

## `train()` vs. `eval()`

During training, we use:

```python
model.train()
```

This puts the model into **training mode**. During evaluation, we need to switch the model to **evaluation mode**:

```python
model.eval()
```

This causes layers like:

- Dropout
- Batch Normalization

to behave differently, as expected at test time. So we should always write `model.eval()` before evaluation.

---

## Evaluating on Test Data

The evaluation code is very similar to the training loop. The only difference is that we no longer:

- Perform backpropagation
- Update the weights

So we only compute the model's output:

```python
for images, labels in test_loader:

    outputs = model(images)

    loss = criterion(outputs, labels)
```

---

## Computing Test Loss

As during training, a loss value is computed for each batch. To get the total loss over the test data, we sum the values:

```python
test_loss = 0

test_loss += loss.item()
```

At the end, we usually compute the average loss:

```python
test_loss /= len(test_loader)
```

---

## Computing Accuracy

To compute accuracy, we first need to extract the predicted class. If the model's output is logits:

```python
preds = torch.argmax(outputs, dim=1)
```

Then we compute accuracy with TorchMetrics:

```python
metric.update(preds, labels)
```

At the end:

```python
accuracy = metric.compute()
```

It's also a good idea to reset the metric before starting evaluation:

```python
metric.reset()
```

---

## Why Isn't Softmax Inside the Model?

In TensorFlow, the final layer usually looks like this:

```python
Dense(10, activation="softmax")
```

In PyTorch, this usually isn't the case. Instead, we write:

```python
nn.Linear(...)
```

and use the following loss:

```python
nn.CrossEntropyLoss()
```

The reason is that **`CrossEntropyLoss` applies the Softmax operation internally**. So if we add Softmax inside the model as well, the computations will be incorrect.

---

## What Is the Model's Output?

Since there's no Softmax inside the model, the output isn't a probability — it's **logits**. For example, the model's output might look like:

```
[-2.1, 5.8, -1.4, 0.9, ...]
```

These numbers are not probabilities.

---

## Converting Output to Probabilities

If we want to see the probability of each class, we need to apply Softmax ourselves:

```python
probabilities = nn.softmax(dim=1)(outputs)
```

Now the output will look something like:

```
[0.001,
 0.003,
 0.982,
 0.010,
 ...]
```

The sum of all these numbers equals 1.

---

## Should We Apply Softmax in Production Too?

Yes.

During training, there's no need for Softmax inside the model. But if we want to:

- Display the probability of each class
- Know the model's confidence
- Show the output to a user

we can apply Softmax after getting the model's output, since Softmax has no trainable parameters.

---

## Saving the Model

After training, we usually save the model so it doesn't need to be retrained later.

In PyTorch:

```python
torch.save(model, "model.pt")
```

or

```python
torch.save(model, "model.pth")
```

Both extensions are commonly used.

---

## Loading the Model

To load the model:

```python
model = torch.load("model.pt")
```

or

```python
model = torch.load("model.pth")
```

---

## The Limitation of Sequential

So far, we've built the model using Sequential, for example:

```
Conv
↓

ReLU
↓

MaxPool
↓

Conv
↓

ReLU
↓

MaxPool
↓

Flatten
↓

Linear
↓

Linear
```

This architecture is completely linear. But networks aren't always this simple.

---

## Why Do We Need Classes?

Many modern architectures, such as:

- ResNet
- DenseNet
- U-Net
- Transformer
- Vision Transformer

have multiple paths, skip connections, and complex blocks. These can no longer be built with Sequential. In these cases, we use object-oriented programming.

---

## Building a Model with a Class

First, we inherit from `nn.Module`:

```python
class CNN(nn.Module):
```

This gives our class all of PyTorch's built-in capabilities.

---

## The `__init__` Function

Inside this function, all the layers are defined. For example:

```python
self.conv1 = nn.Conv2d(...)
```

or

```python
self.pool1 = nn.MaxPool2d(...)
```

or

```python
self.fc1 = nn.Linear(...)
```

This part only defines the model's structure.

---

## The `forward` Function

In this function, we specify how data moves through the network. For example:

```python
x = self.conv1(x)

x = self.relu1(x)

x = self.pool1(x)

x = self.conv2(x)

x = self.flatten(x)

x = self.fc1(x)

return x
```

This is where the actual path of data through the network is defined.

---

## Benefits of Using a Class

Building a model with a class has several advantages:

- Enables building very complex networks
- Allows implementing skip connections
- Allows building residual blocks
- Enables reusing blocks
- Improves code readability
- Makes projects easier to extend

For these reasons, nearly all professional PyTorch models are written using **classes**.

---

## Summary

We covered the fundamental concepts of PyTorch and its overall structure. We learned how to install PyTorch, import its core libraries, and load the ready-made MNIST dataset using `torchvision`.

We also examined an important difference in how images are stored between PyTorch and TensorFlow, and saw that in PyTorch, image dimensions are ordered as **Channel × Height × Width**.

We then explored the `DataLoader` class, a tool responsible for preparing data as batches suitable for training a neural network. This class is one of the most important parts of the data pipeline in PyTorch and is used in almost every deep learning project.

We walked through the complete process of building and training a convolutional neural network in PyTorch. First, using `nn.Sequential`, we built an architecture containing **Conv2D, ReLU, MaxPooling, Flatten, and Linear** layers, and learned that in PyTorch, activation functions are defined as separate layers. We then saw that for multi-class classification problems, `CrossEntropyLoss` is sufficient on its own, and there's no need to add `Softmax` at the end of the model, since this loss function applies it internally.

We also covered how to define an **optimizer** and its role in updating weights, and observed one of the biggest differences between PyTorch and Keras: PyTorch has no `model.fit()`, so the entire training process — including the **forward pass, loss computation, backpropagation, weight updates, and metrics like accuracy** — must be implemented manually. While this approach requires more code, it gives us far greater control over the training process, which is one of the reasons PyTorch is so popular in research and advanced deep learning projects.
