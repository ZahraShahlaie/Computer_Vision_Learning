# Hough Transform

> **Is extracting edges alone enough to understand the content of an image?**

The answer is **no**.

Edge detection algorithms only identify the locations of sharp intensity changes, and their output is usually a binary image in which:

- White pixels represent edges.
- Black pixels represent regions without edges.

But this output gives us no information about the **objects present in the image**.

For example, if we have an image of a parking lot, after running the Canny algorithm, only the boundaries of the cars and the parking lines get identified; but we still can't tell:

- How many parking spots are there?
- Which spot is empty?
- Which spot is occupied?
- Exactly where are the parking lines located?

So to extract useful information from an image, we need to go one step further and, in addition to edges, detect geometric structures such as **lines, circles, and various shapes**.

One of the most important classic methods for doing this is the **Hough Transform**.

---

# Why Do We Need the Hough Transform?

Edge detection can only reveal the existence of intensity changes; it can't determine what structure these edges belong to.

For example, suppose Canny's output looks like this:

```
██████████████
```

In this case, we only know that a set of edge pixels exists.

But it's still unclear whether:

- These pixels form a straight line?
- They're part of a rectangle?
- Or they're just a scattered set of points?

The Hough Transform is designed precisely to answer this question.

---

# Applications of the Hough Transform

The Hough Transform is used to detect various geometric structures, including:

- Detecting straight lines
- Detecting road lines
- Detecting parking lines
- Detecting street curb lines
- Detecting lines in scanned documents and pages
- Detecting circles (Hough Circle)

---

### The Difference Between Edge Detection and the Hough Transform

Edge detection and the Hough Transform are two different stages in image processing, and each serves a specific purpose.

In **edge detection**, the goal is to identify regions of the image where intensity changes abruptly. The output of this step is usually a **binary image** that only shows the location of edges. This method merely identifies the presence of edges; it doesn't provide any information about their shape or type.

In contrast, the **Hough Transform** uses these extracted edges to detect **geometric shapes** such as lines, circles, and other parametric structures. This algorithm's output is no longer an image but a set of **geometric parameters** (such as the start and end coordinates of a line, or the center and radius of a circle). That's why the Hough Transform, in addition to shape detection, also enables the analysis, measurement, and decision-making about objects present in an image.

Put simply, **edge detection only identifies the location of edges, while the Hough Transform uses those same edges to identify and extract meaningful geometric structures.**

---

# The Core Idea Behind the Hough Transform

Suppose that after running the Canny algorithm, we have a set of edge pixels.

Now the question is:

**Which of these pixels lie on a straight line?**

To answer this question, the Hough Transform uses the concept of **parameter space**.

---

# Representing an Image in a Coordinate System

Every image is made up of a set of pixels.

Each pixel has specific coordinates:

```
(x , y)
```

For example:

```
(3,1)
(4,2)
(5,3)
```

If these points lie on a line, we should be able to describe them with a line equation.

---

# The Line Equation

The standard equation of a line is:

$$y = mx + c$$

where:

- **m** is the slope of the line
- **c** is the y-intercept

---

# Moving from Image Space to Parameter Space

In a regular image, each point is just a single pixel.

But the Hough Transform takes a different perspective.

Instead of examining points in **(x, y)** space, it transfers them into **(m, c)** space.

To do this, we rewrite the line's relationship as:

$$c = y - mx$$

Now each point in the image turns into a line in **m-c** space.

---

# The Core Idea Behind Line Detection

Suppose three points lie on a line.

For each point, the following relationship is written:

$$c = y - mx$$

In **m-c** space, each point produces a line.

If these three points really lie on the same line, the three resulting lines in parameter space will intersect at **one common point**.

This common point is exactly the values of:

- Slope (m)
- Intercept (c)

of the original line.

The more lines that intersect at a single point, the higher the likelihood that a real line exists in the image.

That's why the Hough Transform essentially searches for the **highest number of intersections** in parameter space.

---

# The Problem with y = mx + c

The relationship

$$y = mx + c$$

has one fundamental problem.

If the line is vertical:

```
|
|
|
|
```

its slope will equal infinity.

As a result:

$$m = \infty$$

and computations become very difficult, or even impossible.

That's why a different form is used to represent lines.

---

# Polar Representation of Lines

The Hough Transform usually uses a polar representation:

$$\rho = x\cos\theta + y\sin\theta$$

where:

- **ρ (rho)** is the perpendicular distance from the line to the origin.
- **θ (theta)** is the line's angle relative to the horizontal axis.

This representation has no issues with vertical or horizontal lines and represents all lines uniformly.

---

# The Concept of ρ and θ

In this representation, each line is defined by two parameters:

**ρ**: the distance of the shortest perpendicular line from the origin to the line.

**θ**: the angle of that same perpendicular line relative to the horizontal axis.

So each line is defined by just one pair of values:

```
(ρ , θ)
```

---

# The Hough Transform Process

For each edge pixel:

1. We vary θ from **0 to 180 degrees**.
2. For each angle, the value of ρ is computed.
3. The resulting points are plotted in **ρ-θ** space.

In parameter space, each pixel produces a curve.

If several pixels belong to the same line, their curves intersect at one common point.

That common point represents the line present in the image.

---

# The Concept of the Accumulator

The Hough Transform uses a matrix called an **accumulator**.

Each time a curve passes through a cell, that cell's value is increased by one.

In the end, the cells with the highest number of votes represent the real lines in the image.

That's why the Hough Transform is sometimes also called a **voting algorithm**.

---

# Why Does θ Range from 0 to 180 Degrees?

To find all possible cases, it's enough for the angle to range from:

```
0°
to

180°
```

In practice, in OpenCV implementations this angle is converted to **radians**, since trigonometric functions like `sin()` and `cos()` accept input in radians.

---

# Advantages of the Hough Transform

- Detects straight lines with high accuracy
- Reasonably resistant to noise
- Can work with incomplete lines
- Performs well even when there are breaks in the edges
- Well suited for industrial and machine vision applications

---

# Limitations

- Relatively high computational cost
- Sensitive to parameter selection
- Better suited for regular geometric shapes
- In very cluttered images, the number of false votes may increase.

---

# Detecting Lines with Hough Line Transform in OpenCV

The goal of the Hough Transform is to detect **lines present in an image**.

After running the Hough Transform, we expect information about each of these lines to be extracted so we can draw them on the image or use them in later processing steps.

---

# Methods for Representing a Line

There are several different ways to represent a line.

### 1. Line Equation

The most common representation of a line is:

$$y = mx + c$$

where:

- **m** is the slope of the line
- **c** is the y-intercept

---

### 2. The Start and End Points of the Line

Another approach is to have just two points on the line:

```
(x1 , y1)
```

and

```
(x2 , y2)
```

With these two points, the entire line can be drawn.

---

### 3. Polar Representation (ρ and θ)

In the Hough Transform, a line is usually represented by these two parameters:

- **ρ (Rho)**: the perpendicular distance of the line from the origin
- **θ (Theta)**: the angle of the line

This is exactly the format OpenCV provides us as output.

---

# Steps for Running the Hough Transform

Before running the Hough Transform, the image's edges must be extracted.

The usual steps are as follows:

```
Original Image
        ↓
Grayscale
        ↓
Gaussian Blur
        ↓
Canny Edge Detection
        ↓
Hough Line Transform
```

---

# Why Do We Use Canny Before Hough?

The Hough Transform doesn't work directly on the original image.

This algorithm's input is the **edge image**.

That's why we first need to process the image with an algorithm like Canny.

If the edges aren't extracted properly:

- Real lines won't be detected.
- False lines will be created.
- The algorithm's accuracy will decrease.

So the quality of the Hough Transform depends heavily on the quality of the edge detection step.

---

# The Importance of Gaussian Blur

Gaussian blur is usually applied before running Canny.

This step causes:

- Image noise to decrease.
- Fewer false edges.
- Better performance from Canny.

If Gaussian blur is skipped, Canny's output usually contains a lot of noise, which reduces the Hough Transform's accuracy.

---

# The HoughLines Function in OpenCV

To detect lines, the following function is used:

```python
lines = cv2.HoughLines(
    image,
    rho,
    theta,
    threshold
)
```

---

# Function Output

For each line, the function's output is:

```python
(rho , theta)
```

meaning only two values are returned for each line:

- Distance from the origin (ρ)
- Angle of the line (θ)

---

# The Concept of the Rho Value

The unit of ρ is **pixels**.

For example:

```
ρ = 487
```

means this line's perpendicular distance from the origin is **487 pixels**.

---

# The Concept of the Theta Value

The angle θ specifies the line's angle.

For example:

```
θ = 0°
```

In this case, the detected line will be a vertical line.

In polar representation, unlike the form

$$y = mx + c$$

there's no issue detecting vertical lines.

---

# Parameters of HoughLines

## 1. Input Image

The first parameter is the edge image.

Usually, Canny's output is given directly to this function.

```python
lines = cv2.HoughLines(edges, ...)
```

---

## 2. Rho Step

The second parameter is the step size between the values checked for ρ.

For example:

```python
rho = 1
```

means the algorithm checks distances with a precision of one pixel.

The smaller this value, the higher the accuracy, but processing time also increases.

---

## 3. Theta Step

The third parameter is the step size between angles.

It's usually written as:

```python
np.pi / 180
```

meaning angles are checked in steps of **one degree**.

If this value is made smaller:

- Line detection accuracy increases.
- But processing time also increases.

---

## 4. Threshold

The last parameter is very important.

```python
threshold = 150
```

The threshold value determines:

> How many votes a line needs to receive to be recognized as a real line.

---

# The Effect of the Threshold Value

### A Large Threshold Value

If the threshold value is large:

- Only long, clear lines get detected.
- Short lines get removed.

---

### A Small Threshold Value

If the threshold value is small:

- Small lines also get detected.
- But the likelihood of detecting noise as a line increases.

So choosing the right threshold value depends on the type of image and the size of the lines.

---

# Drawing Lines in OpenCV

To draw lines, the following function is used:

```python
cv2.line(
    image,
    (x1, y1),
    (x2, y2),
    color,
    thickness
)
```

The parameters of this function are:

- **image**: the destination image the line is drawn on.
- **(x1, y1)**: the coordinates of the line's starting point.
- **(x2, y2)**: the coordinates of the line's ending point.
- **color**: the color of the line. In OpenCV, colors are specified as **BGR**, not RGB.
- **thickness**: the line thickness in pixels.

---

# Line Colors

In OpenCV, images are usually stored using the **BGR** color space.

So:

- Red: (0, 0, 255)
- Green: (0, 255, 0)
- Blue: (255, 0, 0)

If you've already converted the image to RGB, the order of colors will also change, so you need to pay attention to the image's color space when drawing lines.

---

# Converting (ρ, θ) to Drawable Points

The `HoughLines` function only returns two values:

```
ρ and θ
```

But the `cv2.line()` function needs two points to draw a line:

```
(x1 , y1)
```

and

```
(x2 , y2)
```

So we first need to use trigonometric relationships to convert the ρ and θ values into the line's start and end points.

This conversion is based on sine and cosine relationships, and it then allows the line to be drawn on the image.

---

# Important Notes

- Always check the quality of Canny's output first.
- Removing Gaussian blur usually increases noise and reduces the Hough Transform's accuracy.
- The `rho` value controls the algorithm's spatial precision.
- The `theta` value determines the angular precision.
- The `threshold` value determines how many votes are needed to detect a line.
- Choosing these parameters correctly has a direct impact on the quality of the extracted lines.

---

# Converting ρ and θ Parameters into a Line

In the Hough line transform, each line is represented by these two parameters:

- **ρ (Rho)**: the perpendicular distance of the line from the origin (in pixels)
- **θ (Theta)**: the angle of the perpendicular line relative to the x-axis (in radians)

Our goal is to convert these two values into two points on the line so we can draw it with `cv2.line()`.

---

## Computing the Line's Crossing Point

First, we compute the closest point on the line to the origin.

From trigonometric relationships, we have:

$$x_0 = \rho \cos(\theta)$$
$$y_0 = \rho \sin(\theta)$$

At this stage, the coordinates of the point `(x0, y0)` on the line are obtained.

---

## Computing the Line's Direction

A single point alone isn't enough to draw a line, since infinitely many lines can pass through one point. So the line's direction also needs to be determined.

The line's direction is obtained from the vector:

$$(-\sin(\theta),\ \cos(\theta))$$

This vector isn't perpendicular to the line — it lies along the line itself.

---

## Generating Two Points to Draw the Line

Once we have the point `(x0, y0)` and the direction vector, it's enough to move along the line in both directions.

That's why the following relationship is usually used:

```python
x1 = int(x0 + 1000 * (-np.sin(theta)))
y1 = int(y0 + 1000 * ( np.cos(theta)))

x2 = int(x0 - 1000 * (-np.sin(theta)))
y2 = int(y0 - 1000 * ( np.cos(theta)))
```

The number `1000` is only used to extend the line so it crosses the image. This value can be chosen larger or smaller.

Finally, the line is drawn with:

```python
cv2.line(image, (x1, y1), (x2, y2), color, thickness)
```

---

# Probabilistic Hough Transform

The second method for line detection in OpenCV is the **probabilistic Hough transform**, implemented with the following function:

```python
cv2.HoughLinesP()
```

Unlike `HoughLines`, this method no longer returns ρ and θ — instead, it directly provides the start and end coordinates of each line.

The output for each line is:

```python
[x1, y1, x2, y2]
```

That's why there's no longer any need for trigonometric calculations to draw lines — these values can be given directly to `cv2.line()`.

---

# The Difference Between HoughLines and HoughLinesP

**HoughLines**

- Output: `ρ` and `θ`
- Requires trigonometric conversion
- Lines are usually drawn as unbounded lines.

**HoughLinesP**

- Output: `(x1, y1, x2, y2)`
- No trigonometric calculations needed
- Only actual line segments in the image are drawn.

---

# Why Is HoughLinesP Faster?

In regular Hough line detection, nearly all the image's pixels are examined, and the Hough space is traversed completely.

But the **probabilistic Hough transform** uses probabilistic sampling. That is, instead of examining every pixel, it examines only a portion of them, and once a valid line is found, it continues tracking it.

As a result:

- Less computation is required.
- Execution speed increases.
- Less memory is used.
- It's better suited for large images.

---

# Parameters of HoughLinesP

```python
cv2.HoughLinesP(
    edges,
    rho,
    theta,
    threshold,
    minLineLength,
    maxLineGap
)
```

- **edges**: the edge-detected image (usually Canny's output).
- **rho**: the step size for checking distance ρ, in pixels. (Usually: rho = 1, meaning Hough space is examined with a precision of one pixel.)
- **theta**: the step size for checking angles, in radians. (Usually: theta = np.pi / 180, meaning angles are examined in steps of one degree.)
- **threshold**: the minimum number of votes needed for a line to be recognized as valid.
  - Small value → more lines are found, but noise also increases.
  - Large value → only strong lines remain.
- **minLineLength**: the minimum acceptable length for a line. (If a line's length is less than this value, it gets removed. For example: minLineLength = 50 means lines shorter than 50 pixels are ignored.)
- **maxLineGap**: the maximum allowed gap between two line segments for them to still be considered a single line. (If part of a line got removed during edge detection, this parameter allows two separate segments to be connected. For example: maxLineGap = 20 means if the gap between two line segments is less than 20 pixels, they're treated as a single line.)

---

# The Effect of the Parameters

- Decreasing `threshold` increases the number of lines detected, as well as noise.
- Increasing `threshold` keeps only stronger lines.
- Increasing `minLineLength` removes short lines and noise.
- Increasing `maxLineGap` connects broken segments of a line, but if it's too large, unrelated lines may also get connected.

That's why the best values for these parameters are usually chosen through **trial and error**, tailored to the type of image.

---

# Detecting Circles with Hough Circle Transform in OpenCV

The **Hough Circle Transform** algorithm is designed for **detecting circular objects** such as coins, wheels, the pupil of an eye, pipes, and any object with a circular shape.

This algorithm's output isn't just a declaration that a circle exists; it provides complete information about each circle, including:

- The center coordinates `(x, y)`
- The radius `(r)`

With this information, we can draw the circle on the image, compute its size, or use it in later image processing steps.

---

# The Core Idea Behind the Hough Circle Transform

The logic of the Hough circle transform is very similar to the **Hough line transform**.

For line detection, the algorithm uses the **line equation**:

$$y = mx + c$$

where:

- `m` is the slope of the line
- `c` is the y-intercept

But for circle detection, the **circle equation** is used instead of the line equation:

$$(x-a)^2 + (y-b)^2 = r^2$$

where:

- `(x, y)` is the coordinates of a point on the circle's circumference.
- `(a, b)` is the coordinates of the circle's center.
- `r` is the circle's radius.

So if we can find the center and radius, the circle is fully determined.

---

# How Does the Hough Circle Algorithm Find a Circle?

Suppose there's a coin in the image.

This coin's edge consists of a large number of pixels.

Each of these pixels could belong to a circle.

The Hough circle algorithm examines all these points and, for each point, computes all possible centers of a circle.

If a large number of these calculations converge on a common center, the algorithm concludes that:

> These points likely belong to a circle.

Finally, the coordinates of the circle's center and its radius are extracted.

---

# Parameter Space

In the original image, each pixel only has coordinates `(x, y)`.

But in the Hough transform, these points get transferred into parameter space.

In parameter space, instead of image coordinates, the circle's parameters — namely:

- Center X
- Center Y
- Radius

are examined.

Each pixel on the edge generates a set of possible centers.

If a large number of these centers intersect at one common point, that point is selected as the **circle's center**.

That's why the Hough transform is also called a **voting** method.

---

# The Concept of Voting

Similar to the Hough line transform, a voting system is also used here.

Each edge pixel votes for possible centers.

If a center's number of votes exceeds a certain value, the algorithm concludes that:

> A circle exists at this location.

The more votes there are, the more confident the algorithm becomes.

---

# Why Is Circle Detection Harder Than Line Detection?

Circle detection is usually more difficult because:

- It has more parameters.
- The number of possible cases is much larger.
- Noise can more easily create false circles.
- Many objects have both internal and external edges.

For example, a coin might have internal edges in addition to its outer edge, which the algorithm might mistakenly detect as circles too.

That's why Hough Circle is more sensitive than Hough Line.

---

# The Importance of Removing Noise Before Hough Circle

That's why a smoothing filter is almost always applied before running Hough Circle.

The most common choices are:

- Gaussian blur
- Median blur

These filters:

- Reduce the image's noise.
- Remove weak internal edges.
- Preserve the circle's main edge.
- Reduce the likelihood of detecting false circles.

That's why nearly all OpenCV examples blur the image first and then run Hough Circle.

---

# Hough Gradient

In OpenCV, the standard method for circle detection is performed using `cv2.HOUGH_GRADIENT`.

In this method:

1. First, the image's gradients and edges are computed.
2. Then this gradient information is used to find possible circle centers.
3. Finally, the best circles are selected.

This method is much faster and more accurate than the classic Hough approach, which is why it's used by default in OpenCV.

---

# Important Parameters of Hough Circle

## 1. dp

This parameter determines the ratio of the voting space's resolution to the original image.

- `dp = 1` means the same size as the image.
- Larger values reduce the resolution of the voting space.

A value of 1 is usually a good choice.

---

## 2. minDist

The minimum distance between the centers of two circles.

If this value is small:

- Multiple detections might be produced for a single circle.

If a suitable value is chosen:

- Duplicate circle detection is prevented.

---

## 3. param1

The threshold used in the edge detection stage (Canny).

This parameter determines which edges enter the Hough stage.

---

## 4. param2

The most important parameter of Hough Circle.

This parameter determines the minimum number of votes needed for a circle to be accepted.

- Small value ⇒ more circles get detected (along with more errors)
- Large value ⇒ only confident circles are accepted.

---

## 5. minRadius

The smallest acceptable radius.

If we know there are no small circles in the image, increasing this value can eliminate many errors.

---

## 6. maxRadius

The largest acceptable radius.

By limiting this value, detection of large, false circles is prevented.

---

# Why Does Setting the Radius Matter?

Suppose the smallest coin in the image has a radius of around 50 pixels.

In this case, it doesn't make sense for the algorithm to check circles with a radius of 10 pixels.

By setting:

- `minRadius`
- `maxRadius`

the search space becomes smaller, and both the speed and accuracy of the algorithm improve.

---

# The Importance of Image Dimensions

All of Hough Circle's parameters depend on the image size.

If the image gets resized, either smaller or larger:

- The circles' radii will change.
- The distance between centers will change.
- The appropriate threshold value will change.

So parameters need to be re-tuned whenever the image size changes.

---

# Hough Circle Output

The Hough circle function returns three values for each circle:

- The center's X coordinate
- The center's Y coordinate
- The circle's radius

So each circle is represented as:

```python
(x, y, r)
```

which is enough to draw the circle.

---

# Drawing a Circle in OpenCV

To draw a circle, the following function is used:

```python
cv2.circle(image,
           (x, y),
           radius,
           color,
           thickness)
```

---

# Why Do We Need to Convert Coordinates to Integers?

The `cv2.HoughCircles()` function may return coordinates and radius as floating-point numbers.

But the `cv2.circle()` function only accepts integers.

That's why, before drawing, values are usually rounded and converted to type `int`.

---

# Important Tips for the Best Results

- Blur the image before running Hough Circle.
- Choose the `minDist` value based on the actual distance between the circles.
- Carefully tune `param2` to eliminate false circles.
- If the approximate size of the circles is known, be sure to limit `minRadius` and `maxRadius`.
- Re-tune the parameters after resizing the image.
- There's no fixed value that works for all images; the best values are chosen through trial and error, tailored to the image.

---

# Advantages

- Accurate detection of circular objects
- Provides center coordinates and radius
- Well suited for coins, wheels, pipes, eye pupils, and other circular objects
- Reasonable speed using Hough Gradient
- Ability to control results by tuning parameters

---

# Limitations

- High sensitivity to noise
- Requires careful parameter tuning
- Heavily dependent on image quality
- Likelihood of detecting internal or false circles in complex objects
- Requires proper preprocessing to achieve the best results

---

# Summary

Edge detection algorithms like Canny only identify the location of edges and don't provide information about the structures present in an image. To extract information such as lines, methods like the **Hough Transform** need to be used.

The core idea behind the Hough Transform is to transfer each edge pixel from image space into a **parameter space**. In this space, if a large number of curves intersect at a common point, that point indicates the presence of a line in the image.

In practice, instead of using the classic line equation (`y = mx + c`), a polar representation (ρ, θ) is used so that all lines, including vertical ones, can be detected without any issue. That's why the Hough Transform is one of the most important and widely used classic algorithms for line detection in image processing and computer vision.

After extracting edges using Canny, this algorithm examines the edge points in parameter space and, for each line, obtains two parameters: **ρ (distance from the origin)** and **θ (the line's angle)**.

In OpenCV, this algorithm is implemented via the `cv2.HoughLines()` function. The accuracy of line detection depends on the quality of the edge detection step, as well as choosing appropriate values for the `rho`, `theta`, and `threshold` parameters. Once these parameters are extracted, the lines can be drawn on the image by converting them into two-point coordinates.

In contrast, the **probabilistic Hough transform** directly returns the start and end coordinates of each line, and in addition to simplifying implementation, it also runs faster due to its use of probabilistic sampling. By properly tuning parameters such as `threshold`, `minLineLength`, and `maxLineGap`, a good balance can be struck between line detection accuracy and noise removal.

The **Hough Circle Transform** algorithm is one of the most important classic computer vision algorithms for detecting circular objects. This method uses the circle equation, parameter space, and a voting system to find the centers and radii of circles. Since circle detection is more sensitive than line detection, using noise-removal filters such as **Gaussian blur** or **median blur**, along with properly tuning parameters like `minDist`, `param2`, `minRadius`, and `maxRadius`, plays a very important role in increasing accuracy and reducing errors.
