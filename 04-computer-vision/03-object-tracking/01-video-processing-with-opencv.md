
# 🎥 Video Structure and Video Processing with OpenCV

Video is one of the most important types of data we deal with in **Computer Vision** projects. In many applications such as **Object Tracking, Object Detection, Semantic Segmentation, Face Detection, Pose Estimation**, and Real-Time systems, instead of a single static image, we are faced with a continuous stream of images.

In this section, we will first become familiar with the structure of video and then explore how to read, process, and display video using **OpenCV**.

---

## 📌 What is Video?

If we examine a video carefully, we realize that video is essentially nothing more than a collection of images.

These images are displayed one after another at a very high speed, and this causes the human eye to perceive a continuous motion.

Each image in a video is called a **Frame**.

Therefore:

> **Video = Sequence of Frames**

For example, if a ball is moving, the position of the ball changes slightly in each Frame:

```text
Frame 1        Frame 2        Frame 3        Frame 4

  ⚽              ⚽              ⚽              ⚽
   |               |               |               |
  ●               ●               ●               ●
```

When these images are displayed in succession at an appropriate speed, we observe the ball moving.

---

# 🎞️ What is a Frame?

Each Frame is essentially an independent image.

Therefore, many of the processing tasks we have previously performed on images can also be applied to the Frames of a video.

For example, on each Frame we can perform:

* Object Detection
* Object Tracking
* Semantic Segmentation
* Face Detection
* Image Processing
* Pose Estimation

Consequently, one of the key ideas in video processing is:

> **A video can be divided into its Frames, and each Frame can be processed like an image.**

---

# ⚡ What is FPS?

One of the most important characteristics of a video is **FPS**, which stands for:

**Frames Per Second**

FPS specifies how many Frames are present in each second of the video.

For example:

```text
30 FPS
```

means that in each second there are:

```text
30 Frames
```

Or:

```text
60 FPS
```

means that in each second there are:

```text
60 Frames
```

---

## 🔍 Difference Between Different FPS Values

Suppose an object moves from point A to point B.

If the video has a low FPS, fewer images of this motion are captured:

```text
A -----> -----> B
   Frame   Frame
```

But if the FPS is higher, we have more images of the object's path:

```text
A --> --> --> --> --> --> B
   Frame Frame Frame Frame
```

As a result, the motion can appear **smoother and more continuous**.

Generally, increasing FPS provides us with more temporal information about the motion, but typically the number of Frames, and consequently the amount of data, also increases.

---

# 📦 Relationship Between FPS and Video Size

Suppose we have two videos with the same duration:

```text
Video A → 30 FPS
Video B → 60 FPS
```

For the same duration, the 60 FPS video has a greater number of Frames.

Therefore, under similar conditions, more data is stored, and it can have a larger file size.

So we can say:

```text
FPS ↑
↓
Number of Frames ↑
↓
Amount of Video Data ↑
```

---

# 🧩 How is Video Duration Calculated?

If we know the total number of Frames in a video and the FPS is also specified, we can calculate the video duration:

```text
Duration = Total Frames / FPS
```

For example, if:

```text
Total Frames = 449
FPS = 25
```

The video duration is approximately:

```text
449 / 25 ≈ 17.96 seconds
```

---

# 🖼️ Important Video Properties

When working with video, we usually need several key properties:

* FPS
* Total number of Frames
* Width
* Height
* Duration

This information is very important for video processing.

For instance, if we want to run an **Object Detection** or **Segmentation** model on the video Frames, knowing the Frame dimensions is very important.

---

# 🐍 Reading Video with OpenCV

To work with video in OpenCV, we use the class:

```python
cv2.VideoCapture()
```

First, we import OpenCV:

```python
import cv2
```

Then, we specify the video path:

```python
video_path = "test/video.mp4"

cap = cv2.VideoCapture(video_path)
```

Here, we have created an object of `VideoCapture` and provided the video path to it.

From now on, we will work with the video using the variable `cap`.

---

# ✅ Checking if Video Opened Correctly

To check whether the video was successfully opened, we use:

```python
cap.isOpened()
```

Example:

```python
if not cap.isOpened():
    print("Video could not be opened.")
else:
    print("Video opened successfully.")
```

If the video was successfully opened, we receive `True`.

Otherwise, the return value is `False`.

---

# 📊 Getting Video Properties

OpenCV provides the ability to retrieve various video properties using:

```python
cap.get()
```

For example:

### Getting FPS

```python
fps = cap.get(cv2.CAP_PROP_FPS)

print("FPS:", fps)
```

---

### Getting the Total Number of Frames

```python
frame_count = cap.get(cv2.CAP_PROP_FRAME_COUNT)

print("Frame Count:", frame_count)
```

---

### Getting the Video Width and Height

```python
width = cap.get(cv2.CAP_PROP_FRAME_WIDTH)
height = cap.get(cv2.CAP_PROP_FRAME_HEIGHT)

print("Width:", width)
print("Height:", height)
```

---

# ⏱️ Calculating Video Duration

After obtaining the FPS and the number of Frames, we can calculate the video duration:

```python
duration = frame_count / fps

print("Duration:", duration)
```

General formula:

```text
Duration = Frame Count / FPS
```

---

# 🔢 Frame Numbering

Video Frames are usually numbered starting from zero.

That is:

```text
Frame 0
Frame 1
Frame 2
Frame 3
...
```

Therefore, the first Frame has number:

```text
0
```

---

# 📸 Reading Video Frames

To read the Frames of a video, we use the method:

```python
cap.read()
```

This method returns two values:

```python
ret, frame = cap.read()
```

Where:

* `ret` indicates whether the Frame was successfully read or not.
* `frame` is the actual image of the Frame.

A common way to read all Frames:

```python
while True:

    ret, frame = cap.read()

    if not ret:
        break

    # Process frame
```

When we reach the end of the video, `ret` will become `False`, and we exit the loop.

---

# 💾 Saving Frames as Images

Since each Frame is an image, we can save it using OpenCV.

For example:

```python
cv2.imwrite(f"frame_{frame_id}.jpg", frame)
```

In this case, the Frames can be saved as follows:

```text
frame_0.jpg
frame_1.jpg
frame_2.jpg
frame_3.jpg
...
```

This demonstrates that a video can be converted into a collection of images.

---

# 🖥️ Displaying Video with OpenCV

To display the Frames, we use:

```python
cv2.imshow()
```

Example:

```python
while True:

    ret, frame = cap.read()

    if not ret:
        break

    cv2.imshow("Video", frame)
```

In each step, one Frame is read and displayed in the Window.

By reading and displaying subsequent Frames quickly, we ultimately observe a moving video.

---

# ⌨️ Exiting the Video with the Q Key

To be able to exit the program while the video is playing by pressing a key, we can use:

```python
cv2.waitKey()
```

For example:

```python
if cv2.waitKey(1) & 0xFF == ord('q'):
    break
```

Here, if the user presses the key:

```text
Q
```

the condition is met, and the loop is terminated.

---

# ⏳ Why is `waitKey()` Important?

One important point when displaying video is controlling the playback speed of the Frames.

If we display the Frames without any delay, the video might play much faster than its actual speed.

For example, if the video is:

```text
25 FPS
```

we should display approximately every:

```text
1 / 25 second
```

one Frame.

Since `cv2.waitKey()` receives the delay value in **milliseconds**, we can calculate the appropriate time:

```text
1000 / FPS
```

For a 25 FPS video:

```text
1000 / 25 = 40 ms
```

Therefore:

```python
cv2.waitKey(40)
```

can be used for a playback speed closer to the original speed of such a video.

---

# 🎬 Complete Example of Displaying Video

A simple example of playing a video with OpenCV:

```python
import cv2

video_path = "test/video.mp4"

cap = cv2.VideoCapture(video_path)

if not cap.isOpened():
    print("Video could not be opened.")
    exit()

fps = cap.get(cv2.CAP_PROP_FPS)

delay = int(1000 / fps)

while True:

    ret, frame = cap.read()

    if not ret:
        break

    cv2.imshow("Video", frame)

    if cv2.waitKey(delay) & 0xFF == ord("q"):
        break

cap.release()
cv2.destroyAllWindows()
```

In this code:

1. The video is opened with `VideoCapture`.
2. The FPS of the video is retrieved.
3. The appropriate display time for each Frame is calculated.
4. Frames are read one by one.
5. Each Frame is displayed.
6. Pressing `Q` stops video playback.
7. Finally, resources are released.

---

# 🔓 Releasing Resources

After finishing work with the video, we must release the resources that OpenCV has allocated.

For this, we use:

```python
cap.release()
```

Also, to close all OpenCV Windows:

```python
cv2.destroyAllWindows()
```

Therefore, typically at the end of the program we have:

```python
cap.release()
cv2.destroyAllWindows()
```

---

# 📷 Using the Webcam

`VideoCapture` is not only for video files.

We can also use it to capture images from a Webcam.

For example:

```python
cap = cv2.VideoCapture(0)
```

The number `0` usually refers to the first camera connected to the system.

In this case, instead of reading a video file, Frames are received in Real-Time from the camera.

So:

```python
cv2.VideoCapture("video.mp4")
```

is for reading a video file

and:

```python
cv2.VideoCapture(0)
```

is used to capture images from the Webcam.

---

# 🧠 The Connection Between Video and Computer Vision

Now that we understand that a video consists of a series of Frames, many Computer Vision algorithms can be applied to video.

The general structure is as follows:

```text
Video
  ↓
Frame 1
Frame 2
Frame 3
Frame 4
  ↓
Computer Vision Processing
  ↓
Detection / Segmentation / Tracking / ...
```

For example, in **Object Detection**, we can detect objects on each Frame:

```text
Video
  ↓
Frame
  ↓
YOLO
  ↓
Object Detection
  ↓
Next Frame
  ↓
YOLO
  ↓
...
```

However, when we want to follow an Object across different Frames, we enter the concept of **Object Tracking**.

Therefore, understanding the structure of video and how to process Frames is an important introduction to the topic of **Object Tracking**.

---

# 📝 Summary

The most important points we learned in this section:

* A video consists of a sequence of images.
* Each image in a video is called a **Frame**.
* **FPS** specifies the number of Frames present in each second.
* Increasing FPS provides us with more information about the motion.
* The total number of Frames and the FPS are used to calculate the video duration.
* We can open a video in OpenCV using `cv2.VideoCapture()`.
* We can check the opening status of the video using `cap.isOpened()`.
* We can retrieve properties like FPS, Frame count, Width, and Height using `cap.get()`.
* We can retrieve Frames one by one using `cap.read()`.
* We can display Frames using `cv2.imshow()`.
* We can control the display speed and receive keyboard input using `cv2.waitKey()`.
* We can exit the display loop by pressing the `Q` key.
* We release video resources using `cap.release()`.
* We close OpenCV windows using `cv2.destroyAllWindows()`.
* `VideoCapture` can be used not only for video files but also for capturing images from a Webcam.

In the continuation of these topics, we can use these same Frames to perform more advanced tasks such as **Object Detection, Object Tracking, and other Real-Time processing on video**.
