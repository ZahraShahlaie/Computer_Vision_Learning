# 🚗 Parking Space Detection Using Computer Vision
<img width="552" height="384" alt="image" src="https://github.com/user-attachments/assets/ac165bcc-17e4-4590-8256-058cabff8b1c" />


This project is a smart parking-status detection system built using classical computer vision methods.
<table>
  <tr>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/93209992-1afc-481f-a47d-e25cbf45376d" width="450"/>
    </td>
    <td align="center">
      <img src="https://github.com/user-attachments/assets/b6bab3e9-a2c6-41dc-b285-de7865b3f4ce" width="450"/>
    </td>
  </tr>
</table>
  

The main goal of the project is to automatically extract parking lines, determine the location of each parking spot, and detect whether each spot is empty or occupied by comparing images.



<img width="1182" height="170" alt="image" src="https://github.com/user-attachments/assets/1cdbf25c-9b18-42d7-a2d2-0ea023e47d80" />

<img width="1182" height="170" alt="image" src="https://github.com/user-attachments/assets/c2fb2594-0486-4549-b075-c600ca9d4b09" />


In a smart parking management system, two main questions need to be answered:

  

1. **Where exactly is each parking spot located?**

2. **Is that spot empty, or is it occupied by an object?**

  

The object occupying a parking spot isn't necessarily a car — any object such as a motorcycle, bicycle, or anything else taking up the space needs to be detected.

  

In many real-world systems, parking status is shown using indicator lights:

  

- 🔴 Red → the spot is occupied.

- 🟢 Green → the spot is empty.

  <img width="837" height="510" alt="image" src="https://github.com/user-attachments/assets/d268daa2-6fea-4f15-9d8e-7ec7603c39a6" />


The goal of this project is to implement a similar example using image processing.

  

---

# Main Project Goals

  

- Detect the lines present in the parking lot

- Extract the geometric structure of the parking area

- Determine the coordinates of each parking spot

- Save the section corresponding to each parking spot

- Compare the current image with a reference image of the empty parking lot

- Determine the status of each parking spot

- Display the result visually

  

---

# Existing Challenges

  

A parking lot image doesn't only contain parking lines. Various factors cause the algorithm to also detect extra, unwanted lines.

  

Some of the most important challenges include:

  

- Lines on the bodies of cars

- Car edges

- Shadows

- Short lines caused by various objects

- Image noise

- Lines at different angles

  

For example, the algorithm might detect a car's edge, or even part of a shadow, as a line, in addition to the actual parking lines.

  

That's why running the Hough Transform alone isn't enough — it needs to be followed by several filtering and processing steps.

  

---

  

# 🔄 Project Pipeline

  

```

Input Parking Image

  

        │

        ▼

  

Image Preprocessing

  

(Crop → Gray → Gaussian Blur)

  

        │

        ▼

  

Edge Detection

  

(Canny)

  

        │

        ▼

  

Line Detection

  

(Hough Line Transform)

  

        │

        ▼

  

Line Filtering

  

(Angle Classification)

  

        │

        ▼

  

Line Processing

  

(Sorting → Merge → Gap Filtering)

  

        │

        ▼

  

Parking Slot Extraction

  

(Intersection Detection)

  

        │

        ▼

  

Slot Classification

  

(Difference Image + Threshold)

  

        │

        ▼

  

Parking Status Visualization

  

(Empty / Occupied)

```

  

---

  

# Important Notes for Real-World Projects

  

In industrial projects, conditions are usually simpler than in this example.

  

## 1. A Reference Image Without Cars

  

Since we have access to the camera, we can take a reference image of the parking lot beforehand, at a time when it's completely empty.

  

This image serves as the best reference for determining the exact location of the parking lines, and afterward all we need to do is check whether a car exists inside each identified spot.

  

---

  

## 2. A Fixed Camera

  

In most surveillance systems, the camera is always installed in a fixed position.

  

So:

  

- The viewing angle doesn't change.

- The location of the parking lines stays fixed.

- Re-tuning is only needed if the camera gets moved.

  

This makes image processing much simpler.

  

---

  

# Removing Unnecessary Parts of the Image (Crop)

  

Before starting processing, it's best to remove parts of the image that carry no useful information.

  

For example:

  

- The sky

- Image margins

- Walls

- Parts of the image with no parking spots at all

  

This helps to:

  

- Reduce noise.

- Increase processing speed.

- Produce more accurate results.

  

In OpenCV, this is simply done by slicing (cropping) the image.

  

---

  

# Converting the Image to Grayscale

  

Edge detection algorithms don't need color information.

  

That's why the image is first converted from the BGR color space to grayscale.

This reduces the amount of information and increases processing speed.

  

---

  

# Reducing Noise with Gaussian Blur

  

Before running Canny, it's better to smooth the image slightly to remove fine noise.

  

We use a Gaussian blur filter for this.

  

A common choice: Kernel = 5×5 and Sigma = 0

  

---

  

# Edge Detection with Canny

  

After noise reduction, the **Canny edge detection** algorithm is used to extract edges.

  

In the output, only the boundaries of objects remain, and the image becomes ready to enter the Hough Transform stage.

  

---

  

# Tuning Canny's Parameters

  

Choosing Canny's parameters depends entirely on the image.

  

For example:

  

- If Gaussian blur is too strong, part of the parking lines may get lost.

- If the threshold value is too small, a lot of noise will end up in the image.

- If the threshold value is too large, some of the main lines get removed.

  

So these values are usually chosen through trial and error.

  

---

  

# Detecting Lines with Hough Line Transform

  

After extracting edges, it's time for the Hough Line Transform.

  

This algorithm finds straight lines from the image's edges.

  

In this example, the probabilistic version, **HoughLinesP**, is used, since in addition to detecting lines, it also gives us the start and end coordinates of each line.

  

Each line is returned as:

  

```text

(x1, y1, x2, y2)

```

  

---

  

# Choosing the Hough Transform Parameters

  

The Hough parameters also play a very important role.

  

One of the most important parameters is the **threshold** value.

  

If the threshold value is large:

  

- Long lines remain.

- Many short lines get removed.

  

In this example, since the parking lines are relatively long, a high threshold value removes much of the noise.

  

Also, using the **minimum line length** and **maximum line gap** parameters, broken lines can be connected to each other, and short lines can be removed.

  

---

  

# Displaying the Lines

  

After running the Hough Transform, the coordinates of each line are drawn on the image.

  

At this stage, we can see that the algorithm has found most of the parking lines; but there are still a few issues.

  

---

  

# Problems with the Hough Transform Output

  

## 1. Extra Lines

  

Sometimes lines get detected on cars, shadows, or object edges.

  

To remove these lines, we can:

  

- Check the lines' angles.

- Check the distance between lines.

- Remove very short lines.

  

---

  

## 2. Detecting Two Lines for One Parking Line

  

One of the most common issues with the Hough Transform is that it detects two lines for each parking line.

  

The reason for this is quite simple.

  

The Canny algorithm detects the two edges of a white line separately.

  

As a result, Hough treats each edge as an independent line too.

  

So instead of one line, we may see two lines very close together.

  

---

  

# Computing the Angle of Lines

  

To determine the direction of a line, we simply need to compute its angle.

  

If a line's two point coordinates are:

  

```text

(x1, y1)

(x2, y2)

```

  

we first compute the difference in coordinates.

  

```text

Δx = x2 - x1

  

Δy = y2 - y1

```

  

Then the angle is obtained from:

  

```text

θ = atan2(Δy, Δx)

```

  

The `atan2` function returns the angle in radians.

  

To convert it to degrees, we just apply:

  

```text

Degree = Radian × 180 / π

```

  

---

  

# Classifying Lines

  

After computing the angle, lines can be divided into three groups.

  

- Horizontal lines: angle close to 0 or 180 degrees » 0° ± 5° or 180° ± 5°

- Vertical lines: angle close to 90 degrees » 90° ± 5°

- Diagonal lines: all lines that don't fall into the previous two groups.

  

---

  

# The Benefit of Classifying Lines

  

With this classification, we can:

  

- Remove diagonal lines.

- Keep only the lines we need.

- Simplify later processing steps.

  

---

  

# Sorting, Merging, and Filtering the Parking Lines

  

Using the **Hough Line Transform**, we detected the lines present in the image, and then divided them into three groups based on angle:

  

- Horizontal lines

- Vertical lines

- Diagonal lines

  

Since the parking lines only consisted of horizontal and vertical lines, there were no diagonal lines in this image, so we only process the first two groups.

  

---

  

# Why Do We Need to Sort the Lines?

  

The lines returned by the Hough Transform have no particular order.

  

For example, the output might look like this:

  

- The first line belongs to the top of the image.

- The second line belongs to the bottom of the image.

- The third line is in the middle of the image.

  

As a result, we can't compute the gap between neighboring lines, since it's unclear which line is next to which.

  

That's why we first need to sort the lines.

  

---

  

# Sorting Horizontal Lines

  

For horizontal lines, their position is defined by the **Y** coordinate.

  

So lines are sorted by the value of **y1**, from smallest to largest.

  

As a result, the lines end up ordered from the top of the image toward the bottom.

  

---

  

# Sorting Vertical Lines

  

For vertical lines, each line's position is defined by the **X** coordinate.

  

So lines are sorted by the value of **x1**.

  

As a result, the lines end up ordered from the left of the image toward the right.

  

---

  

# Result of Sorting

  

Once the lines are sorted, each line ends up right next to its nearest neighbor.

  

This lets us:

  

- Compute the gap between neighboring lines.

- Find duplicate lines.

- Perform later processing steps much more simply.

  

---

  

# Why Does Hough Produce Several Lines for One Parking Line?

  

If we examine the output of the Hough Transform, we see that for nearly every parking line, several very close lines get detected.

  

The reason is that the Canny algorithm extracts both edges of a white line separately.

  

As a result, Hough treats each edge as an independent line.

  

So instead of one line, we usually see two or even three lines very close to each other.

  

---

  

# Merging Nearby Lines

  

To remove duplicate lines, we need to merge lines that are very close to each other.

  

The idea is quite simple.

  

If the distance between two lines is less than a certain threshold, we assume they both belong to the same parking line and combine them into a single line.

  

---

  

# How Lines Get Merged

  

Suppose several lines lie almost on top of one another.

  

In this case:

  

- The new line's position is obtained from the average of the lines' coordinates.

- The new line's start and end are chosen from the smallest and largest coordinate values.

  

This way, instead of several nearly identical lines, we're left with just one precise line.

  

---

  

# The Merge Algorithm

  

The steps for merging lines are as follows:

  

1. The first line is selected as the current line.

2. The next line is read.

3. The distance between the two lines is computed.

4. If the distance is less than the threshold value:

   - The two lines are merged.

   - The new line replaces the previous one.

5. If the distance is greater than the threshold:

   - The current line is saved.

   - The new line is selected as the current line.

6. This process continues until the end of the list.

  

---

  

# Merging Horizontal Lines

  

For horizontal lines, the distance criterion is the difference in **Y** coordinates.

  

If the difference in **Y** between two lines is less than a defined value, these two lines are treated as a single line.

  

When constructing the new line:

  

- The Y value is computed as the average of the two lines.

- The line's start and end are chosen from the smallest and largest X values.

  

---

  

# Merging Vertical Lines

  

For vertical lines, the exact same process is performed, except this time the distance is computed based on **X** coordinates.

  

When merging:

  

- The X value is obtained from the average of the two lines.

- The line's start and end are chosen from the smallest and largest Y values.

  

---

  

# Result of Merging

  

After running the merge step, the number of lines decreases significantly.

  

Now, instead of several duplicate lines, only one line remains for each parking boundary.

  

This step makes the output much cleaner and more usable.

  

---

  

# Computing the Gap Between Lines

  

After merging lines, it's time to examine the gaps between them.

  

The gap is computed differently for each group of lines.

  

## Vertical Lines

  

For vertical lines, the gap equals the difference in **X** coordinates between two consecutive lines.

  

$$Gap = |x_2 - x_1|$$

  

---

  

## Horizontal Lines

  

For horizontal lines, the gap equals the difference in **Y** coordinates between two consecutive lines.

  

$$Gap = |y_2 - y_1|$$

  

---

  

# Why Does the Gap Between Lines Matter?

  

In parking lots, the gap between lines is usually roughly constant.

  

For example, the gap between lines might be around 72 pixels.

  

If a gap suddenly appears very small or very large, there's a good chance one of the lines was detected incorrectly.

  

---

  

# Removing Abnormal Lines Based on Gap

  

After computing the gaps between lines, abnormal lines can be removed.

  

To do this, we define an acceptable range.

  

For example:

  

- Minimum gap: 70 pixels

- Maximum gap: 75 pixels

  

If the gap between two lines falls outside this range, that line gets removed.

  

This method removes many extra lines without needing complex processing.

  

---

  

# Result of the Gap Filter

  

After applying this filter:

  

- Extra lines get removed.

- The gap between all lines becomes roughly uniform.

- Only the real parking lines remain.

  

In this example, one extra line that had an abnormal gap compared to the others was removed using this exact method.

  

---

  

# Displaying the Final Result

  

Finally, the vertical and horizontal lines are drawn on the original image.

  

The final output includes lines that:

  

- Have been correctly identified.

- Have had their duplicates removed.

- Are sorted by position.

- No longer contain abnormal gaps.

- Only show the real parking lines.

  

---

  

# What Have We Done So Far?

  

Up to this stage, in addition to detecting lines with the **Hough Line Transform**, several additional processing steps were performed to make the final output usable.

  

The steps performed were:

  

- Sorting horizontal and vertical lines

- Merging very close lines

- Removing duplicate lines

- Computing the gap between lines

- Filtering out abnormal lines based on gap

- Drawing the final lines on the image

  

---

  

# Extracting Parking Spots from the Detected Lines

  

Up to this point, using **Canny edge detection** and the **Hough Line Transform**, we were able to extract, sort, merge, and filter the parking lines. The result of this stage was a clean, organized set of vertical and horizontal lines that defined the parking lot's structure.

  

Now we move into the second phase of the project — the stage aimed at extracting the exact location of each parking spot and preparing it for detecting **whether it's empty or occupied**.

  

---

  

# Why Do We Need to Standardize the Lines?

  

If we look at the extracted lines, we notice that they don't all have the same length.

  

Some lines:

  

- Are a bit shorter.

- Weren't fully detected at their end.

- Have a few pixels of discrepancy at their start or end.

  

This makes it hard to accurately extract the location of the parking rectangles.

  

So before continuing, we need to make all the vertical lines the same size.

  

---

  

# Standardizing the Length of Vertical Lines

  

To do this, we simply need to:

  

1. Find the maximum `Y` value across all lines.

2. Find the minimum `Y` value across all lines.

3. Set the start and end of every line to these same two values.

  

As a result, all vertical lines:

  

- Start at a common point.

- End at a common point.

  

This way, all lines end up with the same length.

  

---

  

# Result of Standardization

  

After this step:

  

- All vertical lines are exactly the same height.

- They all start from the same point.

- They all end at the same point.

  

This step makes extracting the parking spots much simpler.

  

---

  

# What's the Next Step?

  

Now we have two groups of lines:

  

- Horizontal lines

- Vertical lines

  

The intersections of these lines form the corners of the parking spots.

  

So if we can find where the lines intersect, we've essentially solved most of the problem.

  

---

  

# Extracting the Intersection Points of the Lines

  

For each vertical line, all horizontal lines are checked.

  

Each vertical line has an `x` value, and each horizontal line has a `y` value.

  

So their intersection point is simply:

  

```text

(x_vertical , y_horizontal)

```

  

By repeating this process for all the lines, all the parking lot's corners are obtained.

  

---

  

# Why Do We Need the Intersection Points?

  

Four intersection points next to each other form the four corners of one parking spot.

  

For example:

  

```text

●────────●

│        │

│        │

●────────●

```

  

Once we have these four points, we can extract the rectangle corresponding to each parking spot.

  

---

  

# Displaying the Intersection Points

  

To confirm the accuracy of the calculations, all intersection points are drawn on the image.

  

At this stage, we can see that:

  

- All the parking lot's corners have been identified correctly.

- The positioning of the lines is entirely correct.

  

This step is only used for reviewing and validating the results.

  

---

  

# Extracting the Different Parking Sections

  

Once the intersection points are determined, each parking spot can be extracted separately from the image.

  

To do this, the four corner coordinates are used, and by cropping the image, each parking spot is saved as an independent image.

  

---

  

# Preventing the Extraction of Incorrect Regions

  

Not all rectangles constructed from the intersection points are necessarily parking spots.

  

Some of them:

  

- Have very small height.

- Belong to extra parts of the image.

- Were created due to noise.

  

To remove these cases, minimum width and height are defined.

  

For example:

  

```python

min_width

min_height

```

  

If a rectangle's size is smaller than these values, that region gets removed.

  

This ensures only real parking spots get extracted.

  

---

  

# Information Saved for Each Parking Spot

  

Two types of information are saved for each extracted section.

  

### 1. Coordinates

  

The exact coordinates of the rectangle in the original image.

  

### 2. Cropped Image

  

The image corresponding to that specific parking spot.

  

This way, later in the project, in addition to the image itself, we know exactly which part of the original image this parking spot corresponds to.

  

---

  

# Result of Parking Spot Extraction

  

After running this stage, all the parking spots are extracted separately.

  

For example, if the image contains 14 parking spots, the output will also contain 14 independent sections.

  

Each section corresponds exactly to one parking spot and doesn't overlap with any other part of the image.

  

---

  

# Numbering the Parking Spots

  

To simplify later steps, each parking spot is assigned a number. This number is then drawn at the center of the corresponding rectangle.

  

This numbering is only used to display the results more clearly and doesn't play any role in the main algorithm.

  

---

  

# What Have We Done So Far?

  

Up to this point, we managed to:

  

- Extract the parking lines.

- Standardize the lines.

- Obtain the intersection points.

- Extract each parking spot separately.

- Save the coordinates of each parking spot.

- Number all the parking spots.

  

The project is now ready to enter its final stage.

  

In the next step, the current parking lot image is compared with a **reference image of the empty parking lot** to determine:

  

- Which spots are empty.

- Which spots are occupied by a car or any other object.

  

This way, the system will be able to automatically detect the status of each parking spot and display the result for use in a smart parking system.

  

---

# Detecting Whether a Parking Spot Is Empty or Occupied by Comparing Images

  

Suppose we have two images of a parking lot.

  

- The first image was taken when the parking lot was completely empty.

- The second image shows the same parking lot at a different time, with a number of cars parked in it.

  

If we subtract these two images from each other, only the parts that changed between the two images remain.

  

In effect:

  

```text

Difference = Image_with_Cars - Empty_Parking_Image

```

  

The larger the difference between the two images at a given location, the more likely a new object (such as a car) is present there.

  

---

  

# Computing the Difference Between Two Images

  

In OpenCV, computing the difference between two images is straightforward.

  

First, the two images are the same size, and then the intensity difference of each pixel is computed.

  

The output of this step is an image that only shows the changes between the two images.

  

In this image:

  

- Parts that haven't changed appear nearly black.

- Parts where a car is parked appear bright.

  

---

  

# Using Threshold

  

The output of the image difference usually contains a small amount of noise as well.

  

To remove this noise, **threshold** is applied to the difference image. At this stage, only differences larger than a certain value are kept, and the rest are removed.

  

If the threshold value is too small:

  

- Image noise also gets detected as a change.

  

If the threshold value is too large:

  

- Part of a car might get removed.

  

So this value is usually tuned through several rounds of experimentation to achieve the best result.

  

As a result, the image gets converted into a binary image:

  

- Value **0** → no change

- Value **255** → significant change

  

This makes detecting cars much simpler.

  

---

  

# Building a Function to Compare Images

  

So this process doesn't need to be repeated each time, all the steps are placed inside a function.

  

This function performs the following tasks:

  

1. Receives two images

2. Converts the images to grayscale

3. Computes the difference between the two images

4. Applies threshold

5. Returns the binary difference image

  

This way, whenever we have two new images, we just need to call this function.

  

---

  

# Reusing the Parking Sections

  

In the previous steps, we had already extracted all the parking spots using the horizontal and vertical lines.

  

Now we apply those same coordinates to the difference image.

  

That is, instead of finding the parking spots again, we crop the same previous sections out of the difference image.

  

As a result, for each parking spot, we get a small image of the difference between the two images.

  

---

  

# Examining Each Parking Spot

  

Now each section corresponds to just one parking spot.

  

If a car is present in that section, a large part of the image will be white.

  

If the spot is empty, nearly all of the image will remain black.

  

So it's enough to measure the intensity of each section.

  

---

  

# Using the Average Intensity

  

The simplest approach is to compute the average of all the pixels in each section.

  

If a car is present in that section:

  

- We'll have a large number of white pixels.

- So the average will be a large value.

  

But if the spot is empty:

  

- Almost all pixels are black.

- So the average will be a very small value.

  

---

  

# Deciding with a Threshold

  

After computing the average, we just need to compare it against a threshold value.

  

If: Mean > Threshold, that means an object is present in that parking spot.

  

Otherwise: Mean <= Threshold, that spot is empty.

  

---

  

# Displaying the Result on the Image

  

To display the status of each parking spot, the center of each section is first computed.

  

Then, based on the detection result, a marker is drawn at that location.

  

For example:

  

- 🔴 A red circle → the spot is occupied.

- 🟢 A green circle → the spot is empty.

  

This way, the status of the entire parking lot can be seen at a glance.

  

---

  

# The Benefit of Numbering the Parking Spots

  

Since each parking spot was numbered in the previous step, in addition to color, the number of each parking spot can also be displayed.

  

For example:

  

- Parking spot #3 → empty

- Parking spot #7 → occupied

  

This capability is very useful in parking management systems.

  

---

  

# Two Approaches for Comparing Images

  

There are two general approaches for implementing this project.

  

### First Approach (Used in This Project)

  

The entire image is first compared with the reference image.

  

Then the difference image is divided into the different parking sections.

  

This approach is simpler and runs faster.

  

---

  

### Second Approach

  

Each parking spot is first extracted individually as a section.

  

Then each section from the current image is compared with the corresponding section in the reference image.

  

This approach is more flexible and can achieve higher accuracy in some projects, but it requires more comparisons.

  

---

  

# One Important Challenge

  

One issue with this approach is changing environmental conditions.

  

For example, it's possible that:

  

- The reference image was taken at 9 a.m.

- The new image was captured at 3 p.m.

  

In this case, even if the parking lot is completely empty, changes in ambient lighting, shadows, or weather conditions can create a significant difference between the two images.

  

As a result, the system might make mistakes.

  

---

  

# Suggested Solutions

  

Several approaches can be used to reduce this problem.

  

1. **Periodically updating the reference image**: every few minutes or every few hours, a new image of the empty parking lot is saved so the reference image always stays close to current conditions.

  

2. **Using multiple reference images**: instead of having just one empty image, a set of reference images is saved under different conditions, such as:

  

- Morning

- Noon

- Evening

- Night

- Sunny weather

- Cloudy weather

  

Then, during processing, the image with the least difference from current conditions is selected and used as the reference.

  

This approach usually noticeably increases the system's accuracy.

  

---

  

# Summary

  

In this project, we saw that in computer vision problems, using a single algorithm alone usually isn't enough to achieve the desired result, and combining several processing stages plays an important role in increasing the system's accuracy. In this example, by combining **Canny edge detection** for extracting edges, the **Hough Line Transform** for detecting lines, and a series of post-processing steps, we were able to identify the parking lines more accurately and extract the geometric structure of the parking area.

  

After extracting the lines, the parking spots were automatically constructed. To do this, the vertical lines were first standardized, and then, by computing the intersection points of the horizontal and vertical lines, the corners of each parking spot were obtained. Using these points, each parking spot was cropped separately from the image, and its coordinates were saved. In addition, by numbering the extracted spots, it became possible to independently examine the status of each parking spot in later steps.

  

In the final stage, the occupied or empty status of each parking spot was determined by comparing the current image with the reference image of the empty parking lot. After computing the difference between the images and applying threshold to remove unnecessary changes, each parking section was examined independently. Then, by computing the average intensity or the amount of pixel change in each region and comparing it against a threshold value, the status of each parking spot was determined, and the result was displayed on the image using green (empty) and red (occupied) markers.

  

This project showed that even a simple approach based on classical image processing can achieve acceptable performance for real-world problems such as detecting empty parking spaces. This structure can also serve as a foundation for developing more advanced systems based on deep learning, object detection, and smart parking management models.
