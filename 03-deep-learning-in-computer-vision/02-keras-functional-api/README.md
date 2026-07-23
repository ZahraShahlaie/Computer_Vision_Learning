# Functional API in Keras

The Sequential approach works well for many simple problems, since layers are added **one after another** in a linear stack.

But in real-world projects, network architectures aren't always this simple. Sometimes we need to:

- Send the output of one layer to several different layers.
- Create multiple paths inside the network.
- Combine the outputs of several branches.
- Implement architectures like **ResNet, U-Net, Inception**, and others.

For situations like these, we use the **Functional API**.

---

## What Is Sequential?

In the Sequential class, the network is completely linear. Each layer only receives the output of the previous layer.

```
Input
   │
Conv
   │
Conv
   │
MaxPooling
   │
Conv
   │
Flatten
   │
Dense
   │
Output
```

The code is also very simple:

```python
model = Sequential()

model.add(Conv2D(...))
model.add(MaxPooling2D(...))
model.add(Flatten())
model.add(Dense(...))
```

There are no branches here — the path through the network is completely straightforward.

---

## The Limitation of Sequential

Suppose we want an architecture like this:

```
        Conv
       /
Input
       \
        Conv

         │
      Concatenate

         │
       Output
```

Or suppose we want to send the output of one layer directly into several subsequent layers. This simply isn't possible with Sequential.

---

## What Is the Functional API?

The Functional API is a way of building neural networks where **we explicitly define the connections between layers**.

In this approach, each layer behaves like a function. In other words:

```
output = layer(input)
```

For example:

```python
x = Dense(128)(inputs)
```

This means:

- A Dense layer is created.
- Its input is `inputs`.
- Its output is stored in the variable `x`.

We then pass that same output to the next layer:

```python
x = Dense(64)(x)
```

Finally, we define the network's output:

```python
outputs = Dense(10)(x)
```

---

## The Main Difference Between Sequential and the Functional API

With Sequential, we simply add layers:

```python
model.add(...)
```

With the Functional API, we specify exactly where each layer gets its input from. This gives the programmer full control over the connections between layers.

---

## Steps to Build a Model with the Functional API

### Step 1: Define the Input

First, the network's input is specified:

```python
inputs = Input(shape=(32,32,3))
```

In this example:

- Image height = 32
- Image width = 32
- Number of channels = 3 (RGB)

### Step 2: Build the Layers

Each layer is applied to the output of the previous one:

```python
x = Conv2D(...)(inputs)

x = MaxPooling2D(...)(x)

x = Conv2D(...)(x)

x = Flatten()(x)

x = Dense(...)(x)
```

Note that at each step, the output of the previous step is used as the input to the new one.

### Step 3: Define the Output

Finally, the network's output is defined:

```python
outputs = Dense(10, activation="softmax")(x)
```

### Step 4: Build the Model

At the end, we just need to specify the beginning and end of the network:

```python
model = Model(
    inputs=inputs,
    outputs=outputs
)
```

That's all it takes to build the model.

---

## Why Do We Use Variables Like `x` or `block`?

In the Functional API, the output of each layer is usually stored in a variable. For example:

```python
x = Conv2D(...)(inputs)

x = Conv2D(...)(x)

x = MaxPooling2D(...)(x)
```

or

```python
block1 = Conv2D(...)(inputs)

block1 = Conv2D(...)(block1)

block1 = MaxPooling2D(...)(block1)
```

In practice, the variable only holds **the latest output of the network**. So even if we write:

```python
x = ...
x = ...
x = ...
```

multiple times, there's no problem — each new output simply replaces the previous one. This is why many professional projects use a single variable named `x` throughout.

---

## Example: A CNN

Suppose we have the following architecture:

```
Input

↓

Conv

↓

Conv

↓

MaxPooling

↓

Conv

↓

Conv

↓

MaxPooling

↓

Flatten

↓

Dense

↓

Output
```

With the Functional API, we just connect each layer to the output of the previous one:

```python
inputs = Input(...)

x = Conv2D(...)(inputs)

x = Conv2D(...)(x)

x = MaxPooling2D(...)(x)

x = Conv2D(...)(x)

x = Conv2D(...)(x)

x = MaxPooling2D(...)(x)

x = Flatten()(x)

x = Dense(...)(x)

outputs = Dense(...)(x)

model = Model(inputs, outputs)
```

The result is exactly the same network as the Sequential version.

---

## Does the Functional API Improve the Model?

No.

If the architecture is exactly the same:

- The number of parameters is the same.
- Training speed is roughly the same.
- Model accuracy is the same.

The only difference is **how the network is defined**. So the Functional API isn't a replacement for Sequential — it's a more flexible way to build complex architectures.

---

## Why Specifying Each Layer's Input Matters

Suppose we have three blocks:

```
Input

↓

Block1

↓

Block2

↓

Block3

↓

Output
```

Correct definition:

```python
block1 = ...

block2 = ...(block1)

block3 = ...(block2)
```

Here, information flows through all three blocks.

But if we write it incorrectly:

```python
block3 = ...(block1)
```

In this case, the output of Block2 is completely ignored. The actual path through the network becomes:

```
Input

↓

Block1

↘

 Block3

↓

Output
```

Block2 has effectively been removed from the network. This is why, when using the Functional API, we need to be careful about exactly which output feeds into each layer's input.

---

## The Functional API and ResNet

One of the most important uses of the Functional API is implementing architectures like **ResNet**.

In very deep networks, gradients can weaken as they pass through many layers, reducing how well some layers learn. ResNet largely solves this problem using **skip connections**.

```
Input
   │
Conv
   │
Conv
   │
 +───────────────+
 │               │
 +────── Add ◄───+
        │
      Output
```

In this architecture, the output of one layer is sent directly to layers further ahead. This kind of structure can't be built with Sequential, which is why the Functional API is used instead.

---

## Training the Model

Once the model is built, the rest of the process is exactly the same as with Sequential:

```python
model.compile(...)

model.fit(...)

model.evaluate(...)
```

There's no difference in how the model is trained.

---

## A Note on Dropout

In convolutional layers, high Dropout values are usually not chosen, since these layers are responsible for feature extraction. Typical suitable values are:

- Between **0.1 and 0.2** for convolutional layers
- Around **0.5** for dense layers

Of course, the right value depends on the specific problem and dataset.

---

## Summary

The Functional API is a powerful way to design neural networks in Keras, letting the programmer define connections between layers however they like. Unlike Sequential, which only supports a single linear path, the Functional API allows multiple paths, sending one layer's output to several destinations, or merging several branches together.

If the network architecture is simple and linear, the Functional API and Sequential produce identical results. But for advanced architectures like **ResNet, U-Net, Inception, Siamese Networks**, and many modern designs, the Functional API is essential.

In short:

- **Sequential** is suitable for simple, linear networks.
- **Functional API** is designed for complex, multi-branch networks with non-linear connections, giving the programmer far greater flexibility.
