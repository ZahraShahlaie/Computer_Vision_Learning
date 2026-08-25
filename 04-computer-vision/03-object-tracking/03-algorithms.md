# Object Tracking Algorithms

In the previous piece, we covered the concept of **object tracking** and saw that its goal is to follow one or more objects throughout a video while preserving their identity across frames.

In general, the tracking process can be viewed as:

```text
Video
  ↓
Frames
  ↓
Object Detection
  ↓
Association
  ↓
Matching
  ↓
ID Assignment
  ↓
Object Tracking
```

In this piece, we'll look at how various tracking algorithms establish this connection between frames, and how they differ from one another.

---

## Reviewing the Tracking Process

Suppose we have a video, and several objects appear in each frame. First, **object detection** needs to run on every frame.

The detector typically gives us the following information for each object:

- **Class label**
- **Bounding box**
- **Confidence score**

For example:

```text
Person
Bounding Box
Confidence = 0.92
```

After detection, the core question becomes:

> Which object in the previous frame does the object in the current frame correspond to?

If it's the same object as before, it should keep its **previous ID**. If it's a new object, it should be assigned a **new ID**.

---

## What Are Active Tracks?

One important term in tracking is:

> **Active tracks**

Active tracks refer to the objects that the tracking system is currently following. Suppose we have three cars in a frame:

```text
Car A → ID 1
Car B → ID 2
Car C → ID 3
```

These objects are the system's active tracks at that moment. In simple terms:

```text
Active Tracks
      ↓
Objects currently being tracked
      ↓
Along with their ID and tracking information
```

The system needs to manage and update these active tracks across subsequent frames.

---

## Association in Tracking

After detection, we need to connect new detections to existing tracks. This process is called:

> **Association**

For example:

```text
Frame 1
Car → ID 1

        ↓

Frame 2
Car → ?
```

The system needs to determine whether the car in the second frame is the same `ID 1`, or a new car. If it matches:

```text
Same Object
    ↓
Keep ID 1
```

If it doesn't match:

```text
New Object
    ↓
New ID
```

---

## Object Loss

One important problem in tracking occurs when an object disappears from the system's view for a while. This is called:

> **Object loss**

For example, an object might:

- Get occluded behind another object.
- Enter a region where detection performs poorly.
- Fail to be detected correctly due to fast motion.
- Temporarily go undetected due to noise or low image quality.

Suppose:

```text
Frame 1 → Object ID 1
Frame 2 → Object ID 1
Frame 3 → Object Lost
Frame 4 → Object Lost
Frame 5 → Object Lost
Frame 6 → Object Detected
```

Here we face an important question:

> Is the object in frame 6 the same object as before?

If the algorithm can't recover this connection, it may mistakenly create a new ID.

---

## Prediction

To handle tracking more robustly, some algorithms rely on the concept of:

> **Prediction**

Suppose an object has followed this path in the previous frames:

```text
Frame 1 → ●
Frame 2 →   ●
Frame 3 →     ●
Frame 4 →       ●
```

Using its previous motion, we can predict its likely position in the next frame:

```text
Current Position
       ↓
Motion Information
       ↓
Prediction
       ↓
Expected Position
```

The new detection is then compared against this predicted position.

---

## Matching Using IoU

One way to link a new detection to a previous track is by comparing bounding boxes. One key metric here is:

> **IoU — Intersection over Union**

Simply put:

```text
Previous Bounding Box
          +
Current Detection
          ↓
         IoU
          ↓
   Compare with Threshold
```

If the IoU value exceeds a set threshold, we can conclude that the two bounding boxes likely belong to the same object. For example:

```text
IoU > Threshold
      ↓
Match
      ↓
Same ID
```

and:

```text
IoU < Threshold
      ↓
No Match
      ↓
Possible New Object
```

Of course, how this metric is used varies between algorithms.

---

## SORT

One important tracking algorithm is:

> **SORT — Simple Online and Realtime Tracking**

SORT's core idea is to perform tracking with a focus on **speed and simplicity**. Its overall structure can be seen as:

```text
Detection
    ↓
Prediction
    ↓
Association
    ↓
Matching
    ↓
Track Update
    ↓
ID
```

For association, SORT relies mainly on **an object's spatial and motion information**.

---

## The Role of the Kalman Filter in SORT

SORT uses a **Kalman filter** for prediction. We don't need to go into the mathematical details of the Kalman filter here — the important point is that it helps predict the object's likely position in the next frame. Simply put:

```text
Previous State
      ↓
Kalman Filter
      ↓
Predicted State
      ↓
Compare with Detection
```

In other words, using the object's previous state, the algorithm generates an expected position for it.

---

## Association in SORT

After prediction, the new detections are compared against existing tracks. For example:

```text
Predicted Box
      +
Detected Box
      ↓
IoU
      ↓
Threshold
      ↓
Match / No Match
```

If it's a good match:

```text
Same Object
     ↓
Keep Previous ID
```

If no match is found:

```text
New Object
     ↓
New ID
```

---

## The Limitation of SORT

SORT's key advantage is its high speed, but it also has an important limitation:

> **SORT mainly focuses on spatial and motion information — it doesn't take an object's visual appearance into account.**

This can become a problem in certain situations. For example, consider two similar objects:

```text
Object A        Object B

   🚗              🚗
```

If these two objects get very close to each other, relying purely on spatial information can cause the algorithm to confuse them.

---

## The ID Switch Problem

Another problem that can occur is called **ID switch**. Suppose:

```text
Frame 1 → Car → ID 1
Frame 2 → Car → ID 1
Frame 3 → Car → ID 1
Frame 4 → Car → Lost
Frame 5 → Car → Lost
Frame 6 → Car → ID 5
```

Even though it's the same car as before, the system might give it a new ID after losing track of it. This means:

> An **ID switch** has occurred.

---

## Re-Identification

To better solve this problem, the concept of:

> **Re-identification, or re-ID**

becomes important. Suppose an object goes undetected for several frames:

```text
Frame 1 → ID 1
Frame 2 → ID 1
Frame 3 → Lost
Frame 4 → Lost
Frame 5 → Lost
Frame 6 → ?
```

If the object reappears, the system needs to check whether it's the same object as before or a new one. For this, in addition to position, an object's **visual appearance features** can be used, such as:

- Color
- Appearance
- Shape
- Visual characteristics

These features can help re-identify an object even after it's been lost for a short period.

---

## DeepSORT

To address SORT's limitations, the:

> **DeepSORT**

algorithm was introduced. DeepSORT's core idea is to use an object's **visual appearance features**, in addition to spatial information. So:

```text
SORT
    ↓
Motion / Location
```

whereas:

```text
DeepSORT
    ↓
Motion / Location
        +
Appearance Features
```

In DeepSORT, a neural network is used to extract a **feature vector** for each object.

---

## What Is an Appearance Feature?

Suppose we have two objects in an image:

```text
Person A        Person B

  👤              👤
 Red             Blue
```

Their positions might be very close to each other at a given moment, but their visual appearance is different. So if the algorithm only checks position, mistakes become more likely. But if it also considers appearance:

```text
Location
   +
Appearance
   ↓
Better Association
```

it can form a much more accurate connection.

---

## Feature Vectors

The neural network used in DeepSORT can generate a **feature vector** for each object. Simply put:

```text
Object Image
     ↓
Deep Neural Network
     ↓
Feature Vector
```

These feature vectors, belonging to different objects, are then compared against each other. If two objects are visually similar, the distance between their feature vectors tends to be smaller. So association can rely on two types of information:

```text
Motion Information
        +
Appearance Information
        ↓
Association
```

---

## Tentative and Confirmed States

During tracking, a new detection isn't necessarily accepted immediately as a fully valid track. A new object might first enter a trial state, called:

> **Tentative**

If the object is reliably seen across several consecutive frames, its track can transition to:

> **Confirmed**

Simply put:

```text
New Detection
      ↓
Tentative
      ↓
Seen in Multiple Frames
      ↓
Confirmed
      ↓
Active Track
```

This helps prevent weak or incorrect detections from quickly becoming valid, trusted tracks.

---

## Association in DeepSORT

In DeepSORT, several types of information can be considered during association. In general:

```text
Motion / Location
        +
Appearance Feature
        ↓
Association
        ↓
Matching
```

So, unlike SORT, which focuses mainly on position and motion, DeepSORT also brings appearance information into the matching process.

---

## SORT vs. DeepSORT

| Feature | SORT | DeepSORT |
| --- | --- | --- |
| Detection | Yes | Yes |
| Prediction | Yes | Yes |
| Motion Information | ✅ | ✅ |
| Location Information | ✅ | ✅ |
| Appearance Feature | ❌ | ✅ |
| Re-ID | Limited | Better |
| Speed | Very high | Lower than SORT |
| Complexity | Lower | Higher |
| ID Switch Reduction | Limited | Better |

In general:

> **SORT is well suited for fast, simple tracking, while DeepSORT improves tracking in more complex scenarios by adding appearance features.**

---

## ByteTrack

Another important algorithm that has gained a lot of attention is:

> **ByteTrack**

ByteTrack's key idea is that, unlike approaches that only consider high-confidence detections for tracking, it also makes deliberate use of low-confidence detections.

---

## High-Confidence and Low-Confidence Detections

Suppose the detector produces different confidence scores for different objects:

```text
Detection A → 0.92
Detection B → 0.87
Detection C → 0.61
Detection D → 0.31
```

Detections can be split into two groups:

### High-confidence detections

Detections with a high confidence score.

```text
High Confidence
       ↓
More Reliable
```

### Low-confidence detections

Detections with a lower confidence score.

```text
Low Confidence
       ↓
Less Reliable
```

At first glance, we might think low-confidence detections should simply be discarded entirely. But ByteTrack takes a different approach.

---

## The Core Idea of ByteTrack

One of tracking's problems is that a real object might get detected with low confidence in a given frame, for various reasons, such as:

- Occlusion
- Cluttered scenes
- Changes in the object's pose
- Poor frame quality
- Motion blur
- The object being hidden behind another object

In these situations, the detector might still find the object, but with low confidence. If we discard this detection entirely, we risk losing the track for that object. ByteTrack tries to use these weaker detections to help continue existing tracks.

---

## A Simplified ByteTrack Process

Conceptually:

```text
All Detections
      ↓
Split by Confidence
      ↓
High-Confidence Detections
      +
Low-Confidence Detections
```

First, existing tracks are associated with high-confidence detections. Then, low-confidence detections are used to help match tracks that are still unmatched. Simply put:

```text
High Confidence
       ↓
First Association
       ↓
Unmatched Tracks
       ↓
Low Confidence
       ↓
Second Association
       ↓
Track Update
```

This idea helps ensure that objects with weaker detections aren't dropped from tracking unnecessarily.

---

## Why Does ByteTrack Matter?

Suppose a car moves through several frames:

```text
Frame 1 → Confidence 0.92
Frame 2 → Confidence 0.88
Frame 3 → Confidence 0.42
Frame 4 → Confidence 0.35
Frame 5 → Confidence 0.91
```

If we only accepted high-confidence detections, we might lose the car in frames 3 and 4. But ByteTrack can use the low-confidence detections to maintain the track's connection. As a result:

```text
Frame 1 → ID 1
Frame 2 → ID 1
Frame 3 → ID 1
Frame 4 → ID 1
Frame 5 → ID 1
```

and the track is much less likely to break.

---

## When Should You Use Each Algorithm?

Choosing a tracking algorithm depends on your project's requirements.

### SORT

If:

- Speed matters a great deal.
- The system needs to be lightweight.
- The scene isn't very complex.
- Detection quality is good.

SORT can be a good option.

```text
Speed Priority
      ↓
SORT
```

### DeepSORT

If:

- Objects frequently overlap or interact with each other.
- Appearance features matter.
- ID switches are a significant problem.
- Re-identification matters.

DeepSORT can be a better fit.

```text
Appearance
     +
Re-ID
     ↓
DeepSORT
```

### ByteTrack

If:

- The detector produces detections with varying confidence levels.
- Objects are sometimes detected with low confidence.
- High speed matters.
- You want to use weaker detections to preserve tracks.

ByteTrack is an excellent option.

```text
High Confidence
       +
Low Confidence
       ↓
ByteTrack
```

---

## The Role of the Detector in Tracking

An important point: tracking quality doesn't depend only on the tracking algorithm itself — **the detector plays a critical role too.**

If detection is weak:

```text
Poor Detection
      ↓
Poor Association
      ↓
Poor Tracking
```

But if detection is good:

```text
Good Detection
      ↓
Better Association
      ↓
Better Tracking
```

So in any tracking system, the quality of the detector needs to be taken seriously.

---

## Common Problems in Tracking

A tracking algorithm can run into several common issues:

### 1. Occlusion

An object gets hidden behind another object.

```text
Object A
   ↓
Object B
   ↓
A is hidden
```

### 2. Motion Blur

Fast motion reduces frame quality.

### 3. Detection Failure

The detector fails to find the object in certain frames.

### 4. Similar Objects

Several similar objects appear in the same scene.

### 5. ID Switch

The system assigns one object's ID to a different object.

### 6. Object Loss

An object disappears from the system's view for a period of time.

These problems are the main reasons behind the development of various tracking algorithms.

---

## Summarizing the Evolution of Tracking Algorithms

The evolution of these ideas can be seen simply as:

```text
SORT
 │
 │  Motion / Location
 ↓
DeepSORT
 │
 │  + Appearance Features
 │  + Re-ID
 ↓
ByteTrack
 │
 │  Better use of high- and
 │  low-confidence detections
 ↓
Robust Tracking
```

Each of these algorithms has tried to better address a specific part of the tracking problem.

---

## Final Summary

In an object tracking system, the detector first runs on the video's frames and identifies objects along with their:

- Label
- Bounding box
- Confidence score

The tracking algorithm then needs to link the different detections together. In this process, concepts such as:

- **Active tracks**
- **Association**
- **Prediction**
- **Matching**
- **IoU**
- **Object loss**
- **ID assignment**
- **ID switch**
- **Re-identification**

all come into play.

**SORT**, by focusing on speed and spatial information, provides a simple, fast approach to tracking.

**DeepSORT**, by adding appearance features and re-ID, tries to better handle ID switches and distinguish similar objects.

**ByteTrack**, by deliberately using low-confidence detections, tries to preserve tracks for objects that are only weakly detected.

Ultimately, the overall concept can be summarized as:

```text
                 VIDEO
                   ↓
              DETECTION
                   ↓
        ┌──────────┴──────────┐
        ↓                     ↓
   Bounding Box          Confidence
        ↓                     ↓
        └──────────┬──────────┘
                   ↓
              TRACKING
                   ↓
        Prediction + Association
                   ↓
                Matching
                   ↓
              ID Assignment
                   ↓
            Active Tracks
                   ↓
          Next Video Frame
```

The ultimate goal is for an object — despite motion, changes in position, occlusion, weak detections, or changes in scene conditions — to be followed with **the same ID** throughout the video, as much as possible.
