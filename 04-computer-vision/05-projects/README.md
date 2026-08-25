


## 🔬 Project 1 — Colonoscopy Polyp Detection & Tracking

The colonoscopy project demonstrates a practical medical-imaging application of object detection and tracking.

### Pipeline

```text
Colonoscopy Video / Live Feed
            │
            ▼
     YOLO11 Detector
            │
            ▼
      Object Tracker
            │
            ├───────────────┐
            │               │
      Polyp Detected?       │
            │               │
           Yes              │
            │               │
            ▼               │
   Save Annotated Frame     │
   with Timestamp           │
            │               │
            └───────────────┘
                    │
                    ▼
          Annotated Output Video
```

The system automatically saves frames in which a polyp is detected, allowing the generated frames to be reviewed afterward.

### Main Concepts

* YOLO11 object detection
* Real-time inference
* Object tracking
* `persist=True`
* Confidence thresholding
* Timestamp-based frame capture
* Annotated video generation

The project also explores domain-specific data augmentation choices for medical imagery.

> **Medical Disclaimer:** The project is intended for research and educational purposes only. It is not a certified medical device and should not be used for clinical diagnosis or treatment decisions.

---

## 🚗 Project 2 — Road Lane Segmentation & Vehicle Tracking

The autonomous-driving project demonstrates how multiple perception models can operate together on driving video.

### Pipeline

```text
                    ┌──────────────────────┐
Driving Video ─────►│ DeepLabV3+ / ResNet50│
                    └──────────┬───────────┘
                               │
                               ▼
                         Lane Segmentation
                               │
                               │
                    ┌──────────▼───────────┐
Driving Video ─────►│       YOLO11         │
                    └──────────┬───────────┘
                               │
                               ▼
                       Vehicle Detection
                               │
                               ▼
                         BoT-SORT / ReID
                               │
                               ▼
                       Persistent IDs
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
        Lane Mask Overlay                Vehicle Boxes
              │                                 │
              └────────────────┬────────────────┘
                               ▼
                       Combined Output Video
```

The project demonstrates the integration of:

* Semantic segmentation
* Object detection
* Multi-object tracking
* Re-identification
* Real-time video processing

