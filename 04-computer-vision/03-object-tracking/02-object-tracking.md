# Object Tracking — Tracking Objects in Video

**Object tracking** is an important topic in **computer vision**, typically applied to video data.

In tracking, the goal is to identify one or more objects and follow their movement from one frame to the next throughout a video.

In simple terms:

> **Object tracking means finding an object across different frames and following its path of movement throughout the video.**

For example, suppose we want to follow a player's movement in a soccer video:

```text
Frame 1        Frame 2        Frame 3        Frame 4

  🧍              🧍              🧍              🧍
   ↓               ↓               ↓               ↓
  ID 1            ID 1            ID 1            ID 1
```

The player is in a new position in every frame, but the system needs to recognize that all of these correspond to **the same player**.

---

## The Relationship Between Object Tracking and Video

As we saw earlier:

> **Video = Sequence of Frames**

meaning a video is made up of a large number of images, or **frames**, displayed one after another at a certain rate. This rate is defined by **FPS (frames per second)**.

For example, if a video is:

```text
25 FPS
```

it means we have 25 frames every second. So in tracking, we're essentially dealing with a sequence of frames:

```text
Video
  ↓
Frame 1
  ↓
Frame 2
  ↓
Frame 3
  ↓
Frame 4
  ↓
...
```

Tracking's job is to follow the object of interest across these frames.

---

## What's the Core Problem in Object Tracking?

Suppose we've found a player in the first frame:

```text
Frame 1

┌───────────────┐
│     Player    │
│       🧍      │
└───────────────┘
```

In the next frame, the player has moved:

```text
Frame 2

              ┌───────────────┐
              │    Player     │
              │       🧍      │
              └───────────────┘
```

Now we face an important question:

> Is the player we found in frame 2 the same player as in frame 1?

This is exactly one of the central problems in object tracking. There may be several similar objects in the image, and the system needs to be able to tell them apart.

---

## Tracking Is More Than Just Detection

To better understand tracking, we can break the overall process into a few steps:

```text
Detection
    ↓
Association
    ↓
Prediction / Matching
    ↓
ID Assignment
```

Together, these steps let us follow objects throughout a video.

---

## 1. Detection — Identifying Objects

The first step is **detection**. In each frame, we need to identify the objects we want to track.

For example, in a soccer match, we can detect the players:

```text
Frame
  ↓
Object Detection
  ↓
Player 1
Player 2
Player 3
...
```

Object detection typically gives us information such as:

- **Class label**
- **Bounding box**
- **Confidence score**

For example:

```text
Player
Bounding Box
Confidence = 0.92
```

So detection tells us:

> What exists in the image, and where it's located.

---

### Why Does Detection Matter for Tracking?

If an object isn't detected correctly, tracking runs into trouble too. For example, if a car isn't detected in a frame, the tracking algorithm may not be able to correctly continue its path. So:

```text
Good Detection
      ↓
Better Tracking
```

Detection is one of the most important components of any tracking system.

---

## 2. Association — Linking Objects Together

After detection, we need to link objects across different frames. This step is called:

> **Association**

Suppose we have a ball in frame 1:

```text
Frame 1

⚽
```

In the next frame, the ball has moved:

```text
Frame 2

      ⚽
```

and in the frame after that:

```text
Frame 3

            ⚽
```

We need to recognize that:

```text
⚽ Frame 1
      ↓
⚽ Frame 2
      ↓
⚽ Frame 3
```

are all **the same ball**. So association is responsible for linking the detections from different frames together.

---

## An Example With Multiple Objects

Now let's make the problem a bit harder. Suppose the image has several players:

```text
Frame 1

🧍       🧍       🧍
```

In the next frame:

```text
Frame 2

    🧍       🧍       🧍
```

All three players have moved. The system needs to correctly determine:

```text
Player A → Player A
Player B → Player B
Player C → Player C
```

and must not mistakenly conclude:

```text
Player A → Player B
```

So association is critical for preserving each object's identity.

---

## Prediction — Predicting Position

One idea used in tracking is **prediction**. Suppose an object has followed this path over the previous few frames:

```text
Frame 1    Frame 2    Frame 3    Frame 4

  ●  →       ●  →       ●  →       ●
```

We can use the object's prior motion to estimate roughly where it's likely to be in the next frame. For example:

```text
Current Position
       ↓
Motion Information
       ↓
Prediction
       ↓
Expected Region
```

In other words, the system can say:

> Based on its previous motion, the object is likely to be in this region.

It then compares new detections against this expected position.

---

## Matching — Finding the Best Correspondence

After prediction, we need to check which previous object a new detection corresponds to. For example:

```text
Previous Object
      ↓
Prediction
      ↓
Expected Position
      ↓
New Detections
      ↓
Matching
```

If a new detection matches an object's expected position well, we can conclude it's likely the same object as before.

---

## Using an Object's Visual Appearance

For association and matching, position alone isn't everything — an object's visual appearance can help too.

For example, a car might have characteristics such as:

- Color
- Size
- Shape
- Overall appearance
- Other visual features

Suppose a red car is moving through the scene:

```text
Frame 1       Frame 2       Frame 3

  🚗             🚗             🚗
 Red            Red            Red
```

If another car with a different color and appearance enters the frame, these visual characteristics can help the algorithm avoid confusing the two objects. Of course, exactly how these features are used depends on the specific tracking algorithm.

---

## What Is an ID?

One of the most important concepts in object tracking is the **ID**. Every object in a tracking system is assigned a:

> **Unique ID**

For example:

```text
Player → ID 1
Player → ID 2
Player → ID 3
Car    → ID 4
```

If player `ID 1` moves in the next frame, it should still keep the same:

```text
ID 1
```

In other words:

```text
Frame 1        Frame 2        Frame 3        Frame 4

Player         Player         Player         Player
ID 1           ID 1           ID 1           ID 1
```

The object's position changes, but its identity doesn't.

---

## A Simple Example With Cars

Suppose we have three cars in a video:

```text
Car A → ID 1
Car B → ID 2
Car C → ID 3
```

Car `ID 1` moves:

```text
Frame 1
ID 1 → 🚗

Frame 2
     ID 1 → 🚗

Frame 3
          ID 1 → 🚗

Frame 4
               ID 1 → 🚗
```

As long as it's still the same car, its ID doesn't change. But if a new car enters the frame:

```text
New Car → ID 4
```

it gets assigned a brand new ID.

---

## Tracking's Central Question

If we want to define tracking in its simplest form, a tracking algorithm needs to continuously answer this question:

> **Is the object I'm seeing in the new frame the same object I had in the previous frame?**

If the answer is **yes**:

```text
Same Object
    ↓
Keep Previous ID
```

If the answer is **no**:

```text
New Object
    ↓
Assign New ID
```

---

## The Overall Object Tracking Process

The entire process can be summarized as follows:

```text
                Video
                  ↓
              Frame t
                  ↓
             Detection
                  ↓
        Detected Objects
                  ↓
             Prediction
                  ↓
             Association
                  ↓
              Matching
                  ↓
          Same Object?
            ↙       ↘
          Yes        No
           ↓          ↓
     Keep Same ID   New ID
           ↓          ↓
              Next Frame
                   ↓
                  ...
```

---

## Tracking at a Glance

Summarizing every step very simply:

### Step 1 — Detection

Finding objects within a frame.

```text
Frame
 ↓
Detection
 ↓
Objects
```

### Step 2 — Association

Linking newly detected objects to previously tracked objects.

```text
Previous Objects
       +
New Detections
       ↓
Association
```

### Step 3 — Prediction

Predicting an object's likely position based on prior information.

```text
Previous Motion
       ↓
Prediction
       ↓
Expected Position
```

### Step 4 — Matching

Determining which previous object a new detection belongs to.

```text
Prediction + Detection
          ↓
       Matching
```

### Step 5 — ID Assignment

If the object is the same as before, its previous ID is kept; otherwise, a new ID is assigned.

```text
Same Object → Same ID

New Object → New ID
```

---

## Detection vs. Tracking

These two concepts are closely related, but they aren't the same.

### Object Detection

Its core question:

> **What objects exist in this frame, and where are they?**

For example:

```text
Frame
 ↓
YOLO
 ↓
Car
Person
Car
```

### Object Tracking

Its core question:

> **Across different frames, is this object the same one as before, or a new one?**

For example:

```text
Frame 1 → Car → ID 1
Frame 2 → Car → ID 1
Frame 3 → Car → ID 1
Frame 4 → Car → ID 1
```

So detection is mainly focused on **finding objects**, while tracking is focused on **preserving identity and following an object over time**.

---

## Applications of Object Tracking

Object tracking is used in many computer vision projects, including:

- 🚗 Vehicle tracking
- 👤 People tracking
- ⚽ Player tracking
- 🏀 Ball tracking
- 🚦 Traffic analysis
- 🎥 Video analytics
- 🏭 Surveillance and monitoring
- 🤖 Robotics systems
- 🚘 Intelligent vehicle systems
- 📹 Video surveillance systems

---

## Summary

**Object tracking** means following one or more objects throughout a video.

In a tracking system, objects are first **detected** in each frame. Then, **association** needs to be established between detections across different frames, in order to determine which previous object each new detection corresponds to. This can rely on information such as:

- The object's position
- Its motion
- Its size
- Its color
- Its visual features
- Other extractable characteristics

Next, **prediction** is used to estimate the object's likely position, and **matching** is used to determine which previous object a new detection belongs to. Finally, every object is assigned a **unique ID**.

So the overall concept of tracking can be summarized as:

```text
Detection
    ↓
Prediction
    ↓
Association
    ↓
Matching
    ↓
ID Assignment
    ↓
Object Tracking
```

And the most important question throughout this entire process is:

> **Is the object in the current frame the same object we had in the previous frame?**

If yes, the same **previous ID** is kept; if it's a new object, a **new ID** is assigned to it.

From here, we can move on to studying various **object tracking algorithms** and how each one performs prediction, association, and matching.
