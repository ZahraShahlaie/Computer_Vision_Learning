
# Edge Detection

We've already covered the concept of **gradient** and seen that wherever an image's intensity changes suddenly, there's a good chance an **edge** exists at that location.

Now we want to see how a simple filter can be designed to measure these intensity changes and detect the location of edges in an image.

In fact, almost all classic edge detection algorithms — such as **Prewitt, Sobel, and even Canny** — are built on exactly this idea.

---

# The Core Idea Behind Edge Detection

Suppose the intensity values of a few neighboring pixels are:

```text
10   10   12   15   90   95   95
```

Up until the value **90**, the changes are very small, but the intensity suddenly jumps from **15** to **90**.

This drastic change suggests that an **edge** likely exists at this location.

So if we can compute the intensity difference between neighboring pixels, we can locate edges.

---

# Approximating the Gradient with the Difference of Two Pixels

In the precise sense, a gradient comes from the derivative of intensity; but in digital images, it can be simply approximated as the difference between two neighboring pixels.

For example:

```text
I0 = 10
I1 = 14
```

The intensity difference equals:

```text
I1 - I0 = 4
```

If the difference is small, the gradient is small.

If the difference is large, the gradient will be large, and the likelihood of an edge is greater.

---

# Designing the Simplest Derivative Filter

To compute the difference between two pixels, we can use a very simple kernel:

```text
[-1   1]
```

or its equivalent:

```text
[1  -1]
```

As this filter slides across the image, it computes the intensity difference between two neighboring pixels.

As a result:

- Small difference → small output value
- Large difference → large output value

So the filter's output represents the strength of the edge.

---

# Why Isn't This Filter Enough?

This filter only computes the difference on one side.

But in mathematics, we know that the derivative at a point can be computed from the **right side** or the **left side**.

At many points, these two values aren't equal to each other.

So if we only use one difference, we may lose part of the edge information.

---

# Right Derivative and Left Derivative

Suppose we have a point on a graph.

The slope from the left may differ from the slope from the right.

The exact same thing happens in images.

If we only check the difference on one side, we only see part of the intensity changes.

For more accurate edge detection, both sides of the central pixel need to be examined.

---

# Expanding the Filter

To account for both sides, the kernel is expanded as follows:

```text
[-1   0   1]
```

In this filter:

- The value on the left is subtracted from the image.
- The value on the right is added.
- The central pixel serves only as a reference point.

This way, the intensity difference on both sides of the central pixel is examined.

This kernel forms the basis of many well-known edge detection filters.

---

# Why Is the Output of Edge Detection Usually Black?

After applying this filter, each pixel's value equals the gradient at that point.

If we're inside a uniform region:

```text
100 100 100
```

the difference is nearly zero.

So the output will also be zero.

In grayscale images, a value of zero corresponds to the color **black**.

That's why most parts of an edge-detected image appear black.

---

# Why Are Edges White?

At an edge location, the intensity difference is large.

For example:

```text
10   10   90
```

A large difference is created.

As a result, the filter's output will have a large value.

In a grayscale image, large values are displayed as bright colors, close to white.

So:

- Small gradient → dark
- Large gradient → bright

The stronger the edge, the brighter it appears.

---

# Edge Detection in the X Direction

The kernel

```text
[-1   0   1]
```

moves horizontally across the image.

So it computes the intensity difference along the horizontal direction.

As a result, edges whose intensity change occurs in the horizontal direction (the image's vertical edges) are detected better.

That's why this output is usually called **Edge X**.

---

# The Problem of Negative Values

When computing the gradient, the output may become negative.

For example:

```text
10 → 100
```

The gradient is positive.

But if we move in the opposite direction:

```text
100 → 10
```

the gradient will be negative.

Even though both cases indicate the presence of an edge.

So the sign of the gradient doesn't matter much; what matters is the **magnitude of the intensity difference**.

---

# Why Isn't uint8 Suitable?

If the filter's output is of type `uint8`:

- Negative values can't be stored.
- All values below zero get converted to zero.

As a result, part of the edges get lost.

That's why, when applying derivative filters, the output is usually treated as:

```python
CV_64F
```

or

```python
float64
```

so that both positive and negative values are preserved.

---

# Using the Absolute Value

After computing the gradient, the direction of the edge no longer matters.

So we use the absolute value:

```text
|Gradient|
```

As a result:

```text
+90 → 90

-90 → 90
```

Both represent the edge's strength equally.

In OpenCV, the following function is usually used:

```python
cv2.convertScaleAbs()
```

This function does two things at once:

- Computes the absolute value of the gradient.
- Converts the output back to an 8-bit image (`uint8`).

---

# Edge Detection in the Y Direction

To examine vertical changes, the following kernel is used:

```text
[-1]
[ 0]
[ 1]
```

This filter computes the intensity difference along the vertical direction.

As a result, the image's horizontal edges become more visible.

This output is usually known as **Edge Y**.

---

# Combining Edge X and Edge Y

Neither filter alone is capable of detecting all the edges in an image.

That's why the outputs of both filters are usually combined.

In OpenCV, this is done simply with:

```python
cv2.add(edge_x, edge_y)
```

This produces an image that shows most of the horizontal and vertical edges at the same time.

---

# The Limitation of This Simple Filter

The kernel

```text
[-1 0 1]
```

uses only three pixels.

That's why:

- It's very sensitive to noise.
- It may mistakenly detect noise as an edge.
- Some weak edges may not be detected well either.

<img width="434" height="418" alt="image" src="https://github.com/user-attachments/assets/09edc35d-e11f-414e-addf-0cbdd5143dae" />

That's why, in practice, more advanced filters such as **Prewitt** and **Sobel** are used, which, in addition to computing the gradient, also take more information from each pixel's neighborhood into account and are more resistant to noise.

---

# Improving Edge Detection with Threshold and Gaussian Blur

In the earlier approach, we used a simple filter for **edge detection**, which identified regions with strong intensity changes as edges by computing the intensity difference between neighboring pixels.

In the final image, we saw that in addition to the main edges, a large number of faint dots and lines were also detected as edges. These are usually caused by noise, minor brightness variations, or the internal texture of objects, and they aren't real edges.

In this section, we examine two very important strategies for reducing these errors:

- **Threshold** (thresholding the gradient)
- **Gaussian Blur** (removing noise before computing the gradient)

These two steps are used in nearly all professional edge detection algorithms as well.

---

# The Problem with the Simple Edge Filter

The simple filter computes the intensity difference for every pixel.

So any change in intensity, even a very small one, might be treated as an edge.

That's why the output image usually shows:

- Scattered white dots
- Very faint lines
- Edges caused by noise
- Edges created by shadows or surface texture

<img width="434" height="418" alt="image" src="https://github.com/user-attachments/assets/fa2ac0d1-1c38-42f6-b7c1-018280a8dc78" />

whereas our goal is only to extract the **main edges of objects**.

---

## The Concept of Threshold

Suppose the gradient's intensity ranges between 0 and 255.

If we set the threshold value to 50:

- Gradient less than 50 → zero (black)
- Gradient greater than 50 → 255 (white)

As a result, the output image becomes almost a binary image.

---

## Implementation in OpenCV

To perform thresholding, the following function is used:

```python
_, edge_binary = cv2.threshold(
    edge_image,
    50,
    255,
    cv2.THRESH_BINARY
)
```

Parameters:

- The gradient image
- The threshold value
- The type of thresholding

---

## Threshold Output

After applying threshold:

- Weak edges are removed.
- Main edges remain.
- The image becomes cleaner.
- Small noise no longer shows up as edges.

<img width="434" height="418" alt="image" src="https://github.com/user-attachments/assets/936491e0-206a-45e6-9f7c-25edfd36c66d" />

---

# Choosing an Appropriate Threshold Value

Choosing the right threshold value matters a great deal.

### If the Value Is Too Small

Almost all gradients are preserved.

As a result:

- Noise remains.
- The image becomes cluttered.

<img width="434" height="418" alt="image" src="https://github.com/user-attachments/assets/7729cd01-be13-41f1-ab39-e129921e4bc8" />

---

### If the Value Is Too Large

Many real edges are also removed.

As a result:

- Image detail gets lost.

<img width="434" height="418" alt="image" src="https://github.com/user-attachments/assets/6c4b89e8-6abe-4903-9641-023e0b69d7b7" />

---

So the right threshold value is usually chosen through experimentation.

---

# Examining the Gradient Histogram

Before thresholding, gradient values are spread across the full 0–255 range.

But after thresholding, only two values remain:

- 0
- 255

So the output will be completely binary.

<img width="547" height="428" alt="image" src="https://github.com/user-attachments/assets/c01e3845-668e-48e1-83dd-e76dd208f305" />
<img width="547" height="428" alt="image" src="https://github.com/user-attachments/assets/a67966bf-969e-4700-af83-3049904a0212" />

---

# The Second Strategy: Gaussian Blur

## Why Do We Need Gaussian Blur?

One of the most significant problems with derivative filters is that they're very sensitive to noise.

Even a single-pixel noise value might get detected as a strong edge.

That's why, before computing the gradient, the image is usually blurred slightly.

---

## Core Idea

Gaussian blur reduces sudden changes in intensity.

As a result:

- Fine noise gets removed.
- Small variations get smoothed out.
- Only real edges remain.

---

## Implementation

```python
image_blur = cv2.GaussianBlur(
    image,
    (3,3),
    0
)
```

Here, a small kernel is used so that:

- Noise gets removed.
- The image's main details are preserved.

---

# The Effect of Gaussian Blur

After applying Gaussian blur:

- Noise decreases.
- Gradients caused by noise are reduced.
- Edge detection becomes more accurate.

But there's one thing to keep in mind.

---

# The Effect of Gaussian Blur on Edges

Although Gaussian blur reduces noise, it also slightly softens the edges at the same time.

As a result:

- The gradient value at edges also decreases.

That's why, after applying Gaussian blur, the threshold value usually needs to be lowered a bit as well.

For example:

Without blur:

```text
Threshold = 80
```

With Gaussian blur:

```text
Threshold = 50
```

because the edge gradients become slightly smaller after blurring.

---

# The Correct Order of Steps

The complete process for improved edge detection is as follows:

```
Original Image

        │
        ▼

Gaussian Blur

        │
        ▼

Gradient Calculation

        │
        ▼

Threshold

        │
        ▼

Final Edge Image
```

This order ensures that noise is removed first, and only then are the real edges extracted.

---

# Advantages and Disadvantages of Threshold

### Advantages

- Removes weak edges
- Reduces noise
- Produces a binary output
- Increases clarity of main edges

### Disadvantages

- Depends heavily on the chosen threshold value
- If the value isn't chosen well, real edges may also be removed.

---

# Advantages and Disadvantages of Gaussian Blur

### Advantages

- Removes noise
- Reduces false edge detection
- Improves edge detection quality

### Disadvantages

- Softens edges
- Reduces gradient magnitude
- Some very fine details may be lost

---

# Why Isn't a Simple Filter Enough?

Suppose we want to determine whether a pixel lies on an edge.

In the simple filter, only the intensity difference between two pixels is examined.

This creates several problems:

- High sensitivity to noise
- Mistakenly detecting small changes as edges
- Not using information from surrounding pixels
- Instability on real-world images

Whereas in a real image, edges usually don't appear at just a single pixel — they continue across several neighboring pixels. So if we can examine more neighboring information, the result will be more accurate.

---

# The Prewitt Filter

## Core Idea

Instead of comparing just two pixels, the **Prewitt** filter uses a **3×3** window.

In other words:

- The left and right neighbors
- The top and bottom neighbors
- The diagonal neighbors

all participate in computing the gradient.

As a result, the effect of small noise is reduced, and real edges are detected better.

---

## Prewitt Kernels

### X Axis

```text
[-1   0   1]
[-1   0   1]
[-1   0   1]
```

### Y Axis

```text
[-1  -1  -1]
[ 0   0   0]
[ 1   1   1]
```

These two kernels measure intensity changes along the horizontal and vertical directions, respectively.

---

## How It Works

First, the image is convolved with the **X**-axis kernel to obtain the horizontal gradient.

Then the image is processed with the **Y**-axis kernel to compute the vertical gradient.

Finally, these two results are combined to form the final edge image.

---

## Why Is Prewitt Better Than the Simple Method?

Because instead of examining just one intensity difference, it uses information from the 9 surrounding pixels.

As a result:

- Small noise has less effect.
- Real edges are better preserved.
- The output is more stable.

It's worth noting that **Prewitt doesn't remove noise**; it only reduces its sensitivity to noise.

---

# Using Gaussian Blur Before Prewitt

In nearly all gradient-based methods, it's better to smooth the image slightly before computing the gradient.

That's why **Gaussian blur** is usually applied before running Prewitt.

Its benefits:

- Reduces noise
- Reduces sudden intensity changes
- Reduces false edges
- Improves output quality

That's why, in implementation, there's usually an option to enable or disable Gaussian blur.

---

# Why Do We Use Float Data?

When computing the gradient, values can:

- Become negative.
- Exceed 255.

whereas a regular image is of type `uint8` and only holds values between 0 and 255.

So the gradient is first computed using the `CV_64F` or `float64` data type.

Then, with the function:

```python
cv2.convertScaleAbs()
```

- The absolute value is taken.
- Negative values are eliminated.
- The output is converted back into the 0–255 range.

---

# Why Is Prewitt's Response Stronger?

If we look closely at the Prewitt kernel, there are three values equal to **1** and three values equal to **-1**.

So when computing the gradient, the intensity difference is used three times in the calculation.

As a result:

- The gradient value becomes larger.
- Edges appear brighter.
- The response strength increases compared to the simple filter.

That's why, when using threshold, the threshold value needs to be raised to match Prewitt's output.

For example, if a suitable threshold in the simple filter is around **60**, in Prewitt a value of **170 to 180** might give better results.

---

# The Sobel Filter

## Core Idea

The **Sobel** filter is an improved version of Prewitt.

In Prewitt, all neighbors carried roughly equal importance.

But Sobel assumes that pixels closer to the center matter more.

That's why it assigns greater weight to the central neighbors.

---

## Sobel Kernel

### X Axis

```text
[-1   0   1]
[-2   0   2]
[-1   0   1]
```

### Y Axis

```text
[-1  -2  -1]
[ 0   0   0]
[ 1   2   1]
```

We can see that the coefficient **2** gives more important information greater weight.

---

# Sobel's Advantage Over Prewitt

Because of better weighting:

- Edges are extracted more accurately.
- Noise has less effect on the result.
- The real gradient is computed more accurately.
- The output is smoother and more stable.

That's why Sobel is one of the most widely used gradient filters in OpenCV.

---

# Implementing Sobel in OpenCV

OpenCV has a ready-made function for Sobel.

First, the X-axis gradient is computed:

```python
cv2.Sobel(...)
```

Then the Y-axis gradient is obtained.

Both images are then converted to absolute values using:

```python
cv2.convertScaleAbs()
```

and finally combined together.

---

# Why Are the X and Y Axes Computed Separately?

You might wonder why we don't compute both directions at once.

The reason is that OpenCV's Sobel function computes the gradient for **one specific direction** at a time.

So:

1. The horizontal gradient is computed first.
2. Then the vertical gradient is computed.
3. Finally, the two images are combined.

This approach gives the best accuracy for extracting edges.

---

# Combining Gaussian Blur and Threshold

As mentioned, the best result is usually obtained when these three steps are performed in sequence:

1. Apply Gaussian blur
2. Compute the gradient (Prewitt or Sobel)
3. Apply threshold

Each has a different role:

- **Gaussian blur**: reduces noise and removes small intensity variations
- **Gradient filter**: extracts edges
- **Threshold**: removes weak gradients and keeps the main edges

---

# Why Does Gaussian Blur Improve Edge Detection?

Noise usually causes sharp changes in one or two pixels.

Gaussian blur averages out these sudden changes and reduces their intensity.

As a result:

- The gradient caused by noise becomes smaller.
- The likelihood of mistakenly detecting noise as an edge decreases.
- Very faint shadows have less influence on the result too.

That said, since Gaussian blur softens the image slightly, the gradient strength of real edges also decreases a bit. That's why, after applying Gaussian blur, the threshold value is usually set slightly lower than normal.

---

# Edge Detection with the Laplacian Filter

At this point, we need to understand exactly how the **Laplacian filter** differs from methods like **Prewitt** and **Sobel**. In the earlier methods, edges were detected using the **first derivative** — wherever the change in intensity was large, the gradient value was also large, and that point was identified as an edge. Laplacian, however, takes a different approach and computes the **second derivative** instead of the first.

**Laplacian** is one of the classic edge detection methods that works based on the **second derivative of the image**. Unlike Sobel and Prewitt, which first measure the intensity of changes, Laplacian examines changes in the gradient itself.

That's why this filter is able to detect very fine edges and small details in an image, but in return, it's more sensitive to noise.

---

# First Derivative and Second Derivative

Suppose an image's intensity in a region is roughly constant.

In this case:

- Intensity changes are very small.
- So the **gradient (first derivative)** will be nearly zero.

But when we reach an edge:

- Intensity changes suddenly.
- The gradient value increases.

This is exactly what the **Prewitt** and **Sobel** filters measure.

But Laplacian goes one step further.

Instead of measuring the gradient's value, it examines:

> **How quickly does the gradient itself change?**

In other words, it uses the **second derivative**.

---

# The Concept of the Second Derivative

If the first derivative shows that intensity is increasing, the second derivative determines:

- Where this increase started.
- At what point the greatest change occurred.
- When the changes stopped again.

In effect, the second derivative marks where the gradient's behavior changes, and this is exactly where an edge is usually located.

---

# Zero Crossing

One of the most important concepts in Laplacian is **zero crossing**.

In the second derivative, the output value around an edge usually switches from positive to negative or from negative to positive.

The point where this sign change occurs is called a **zero crossing**.

So in the Laplacian method, edges are detected based on **where the second derivative crosses zero**, not simply on the gradient being large.

---

# The Laplacian Kernel

One of the most common Laplacian kernels is:

```text
      0   -1    0
     -1    4   -1
      0   -1    0
```

Another version is also sometimes used:

```text
     -1  -1  -1
     -1   8  -1
     -1  -1  -1
```

In this kernel, the central pixel is compared to its surrounding neighbors, and changes in all directions are examined simultaneously.

---

# The Difference Between Laplacian and Sobel/Prewitt

**Prewitt** and **Sobel** filters compute the gradient separately along the **X** and **Y** axes, and then combine these two results.

But **Laplacian** examines changes in all directions simultaneously and doesn't need separate computation for the X and Y axes.

That's why it's simpler to run and can extract more detail from an image.

---

# Advantages of Laplacian

Using Laplacian offers several advantages:

- Detects very thin edges
- High sensitivity to image detail
- Examines all directions simultaneously
- No need for separate gradient computation along X and Y axes
- Well suited for extracting fine image structures

---

# Limitations of Laplacian

Alongside its advantages, this method also has limitations.

Since the second derivative is sensitive to the smallest changes in intensity, noise gets detected as an edge too.

So:

- Fine noise gets revealed easily.
- False edges may be created.
- **Gaussian blur** is usually applied before running Laplacian to reduce the effect of noise.

---

# Using Gaussian Blur

In nearly all practical applications, the image is blurred slightly before running Laplacian.

This causes:

- Random noise to be reduced.
- Zero crossings caused by noise to be removed.
- Real edges to be detected better.

That's why, in many implementations, the order of steps is as follows:

```text
Image
      ↓
Gaussian Blur
      ↓
Laplacian
      ↓
ConvertScaleAbs
      ↓
Threshold (optional)
```

---

# Implementation in OpenCV

Running Laplacian in OpenCV is very simple.

First, the image is fed to the Laplacian filter:

```python
lap = cv2.Laplacian(img, cv2.CV_64F)
```

Since the second derivative's output can include **negative** values, the `CV_64F` data type is used so these values aren't lost.

Then, to display the image:

```python
lap = cv2.convertScaleAbs(lap)
```

The `convertScaleAbs()` function computes the absolute value of the output and converts it into a displayable 8-bit image.

If needed, **threshold** can also be used to keep only the strong edges.

---

# Comparison with Sobel and Prewitt

Overall:

- **Prewitt** is more robust than very simple methods.
- **Sobel**, by giving more weight to central pixels, produces more accurate edges.
- **Laplacian** extracts more detail but is more sensitive to noise.

That's why, in many real-world applications, the image is first smoothed with **Gaussian blur** and then Laplacian is applied to it.

---

# Edge Detection with the Canny Edge Detector Algorithm

The previous methods mostly worked by computing the **image gradient** and examining intensity changes in different directions. Now let's look at the **Canny** edge detection algorithm.

The important thing to understand is that, unlike Sobel and Prewitt, Canny is **not a simple filter**; it's a **multi-stage algorithm** that performs several processing operations in sequence to extract edges that are more accurate, thinner, and more resistant to noise.

This algorithm was introduced in **1986** by **John Canny** and is still considered one of the most standard edge detection methods in image processing.

---

# The Main Steps of the Canny Algorithm

The Canny algorithm consists of several main steps:

1. Noise reduction with Gaussian blur
2. Computing the image gradient
3. Non-Maximum Suppression
4. Double thresholding
5. Edge tracking by hysteresis

---

# 1. Noise Reduction with a Gaussian Filter

The first step in Canny is reducing the image's noise.

The input image usually contains small amounts of noise, and this noise can cause false edges.

To prevent this problem, the image is first converted to **grayscale**, and then a Gaussian filter is applied to it.

The original Canny paper uses a Gaussian filter.

What Gaussian blur does:

- Removes small, sudden intensity changes
- Reduces noise
- Prepares the image for gradient computation

For example:

```
Input Image
      |
      ↓
Grayscale Image
      |
      ↓
Gaussian Blur
      |
      ↓
Smoothed Image
```

---

# 2. Gradient Calculation

After removing noise, we need to find the image's intensity changes.

Edges are locations in the image where intensity changes rapidly.

For this, Canny usually uses the Sobel filter to compute the gradient in both the X and Y directions:

---

# 3. Non-Maximum Suppression

After computing the gradient, edges are still thick.

Canny's goal is to turn these edges into thin, one-pixel-wide lines.

At this stage, it's checked whether a pixel has the maximum gradient value compared to its neighbors.

If a pixel's gradient value is the maximum along the edge direction:

✅ it's kept

Otherwise:

❌ it's removed

The result of this step:

- Thick edges → thin edges
- Reduction of redundant pixels

---

# 4. Double Threshold

At this stage, Canny takes two threshold values:

- A low threshold
- A high threshold

For example:

```
Low Threshold = 80
High Threshold = 180
```

Based on the gradient value, pixels are divided into three categories:

---

## 1. Strong Edge

If:

$$Gradient > High Threshold$$

then:

the pixel is definitely an edge.

For example:

```
Gradient = 220
High Threshold = 180
```

So:

```
Strong Edge
```

---

## 2. Weak Edge

If:

$$Low < Gradient < High$$

then:

we're still not sure whether this pixel is an edge or not.

For example:

```
Gradient = 120

Low = 80
High = 180
```

This pixel is:

```
Weak Edge
```

---

## 3. Non Edge

If:

$$Gradient < Low Threshold$$

then:

this pixel isn't an edge and gets removed.

---

# 5. Edge Tracking by Hysteresis

In the previous step, weak edges are still uncertain.

Canny needs to decide whether a weak edge really belongs to an edge or not.

To do this, its connection to strong edges is examined.

If a weak edge is connected to a strong edge:

✅ it's accepted as an edge.

But if it has no connection to any strong edge:

❌ it's removed.

This way, the algorithm only keeps real edges.

---

# A Simple Example

Suppose:

```
Low Threshold = 80
High Threshold = 180
```

A pixel with the value:

```
Gradient = 250
```

Since:

```
250 > 180
```

it's a:

Strong Edge

Another pixel:

```
Gradient = 120
```

Since:

```
80 < 120 < 180
```

it's a:

Weak Edge

Now Canny checks:

Is this weak edge connected to a strong edge?

If yes:

```
Keep Edge
```

If no:

```
Remove
```

---

# Using Canny in OpenCV

In OpenCV, the Canny algorithm is used with the following function:

```python
cv2.Canny(image, threshold1, threshold2)
```

---

# The Effect of Threshold Values

Choosing the threshold values matters a great deal.

## High Threshold

If the threshold value is large:

- Only very strong edges are detected.
- Weak edges are removed.
- Some image detail may be lost.

Example:

```python
cv2.Canny(img, 180, 250)
```

Result:

- Fewer edges
- A cleaner image

---

## Low Threshold

If the threshold value is small:

- More edges are found.
- But the likelihood of detecting noise as an edge increases.

Example:

```python
cv2.Canny(img, 50, 100)
```

Result:

- More detail
- Higher likelihood of noise

---

# The Relationship Between Low and High Thresholds

Usually:

$$High \approx 2 \times Low$$

is chosen.

For example:

```
Low = 50
High = 150
```

or:

```
Low = 80
High = 180
```

---

# Comparing Canny with Sobel and Prewitt

| Feature | Sobel | Prewitt | Canny |
|---|---|---|---|
| Type | Filter | Filter | Multi-stage algorithm |
| Noise reduction | No | No | Yes |
| Edge thickness | Thick | Thick | One pixel |
| Removal of extra edges | No | No | Yes |
| Accuracy | Moderate | Moderate | High |

---

# Summary

Edge detection is performed by measuring changes in an image's intensity. The simplest approach for this is computing the intensity difference between neighboring pixels using derivative kernels. To preserve both the positive and negative gradients of an image, the output is first computed using floating-point data (such as `float64`) and then converted to an 8-bit image by taking its absolute value.

By computing the gradient in both the **X** and **Y** directions and combining them, a large portion of the edges present in an image can be identified. This idea forms the basis of many well-known edge detection algorithms, such as **Prewitt, Sobel, and Canny**.

The simple edge detection filter, although capable of extracting edges, usually produces a large number of false edges due to its high sensitivity to noise.

To reduce this problem, there are two simple yet highly effective strategies:

1. Using **threshold** to remove weak gradients and keep only the main edges.
2. Using **Gaussian blur** to reduce noise before computing the gradient.

Combining these two steps makes the edge detection output cleaner and more stable. That's why, in many modern edge detection methods, the image's noise is first reduced and only then is edge extraction performed.

**Prewitt** and **Sobel** filters take more information from each pixel's neighborhood into account when computing the gradient compared to simple filters. That's why they have a greater ability to extract real image edges more accurately.

In practical applications, to achieve better results, **Gaussian blur** is usually applied first to reduce noise, then the image gradient is computed using either the **Prewitt** or **Sobel** filter, and finally **threshold** is used to remove weak edges and keep the main ones.

Combining these three steps is one of the most common and effective classic methods for edge extraction in image processing, and it forms the basis of many more advanced edge detection algorithms.

---

The **Laplacian** filter is one of the most important edge detection methods based on the **second derivative**. Instead of directly measuring gradient magnitude, this filter examines changes in the gradient itself, and using the concept of **zero crossing**, it identifies the locations of sharp intensity changes and, consequently, the image's edges.

Laplacian's main advantage is its strong ability to detect very fine edges and small details in an image. On the other hand, this same high sensitivity means noise is easily detected as an edge as well. That's why, in most practical applications, **Gaussian blur** is applied first to reduce noise, and then the Laplacian filter is run, to create a proper balance between preserving detail and reducing noise.

---

The **Canny** algorithm is a powerful edge detection method that, unlike simple filters, consists of several distinct processing stages:

1. Noise removal using **Gaussian blur**
2. Computing the image gradient
3. Thinning edges using **Non-Maximum Suppression**
4. Separating edges using **Double Threshold**
5. Connecting real edges using **Hysteresis Tracking**

Because of these steps, the **Canny** algorithm is able to extract edges that are accurate, thin, and resistant to noise. For this reason, it's one of the most widely used edge detection methods in the field of **computer vision**.
