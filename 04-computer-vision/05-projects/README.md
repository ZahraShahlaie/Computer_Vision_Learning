# Real-Time Road Lane Segmentation & Vehicle Detection/Tracking for Autonomous Driving

An end-to-end computer vision pipeline for autonomous driving perception, combining **real-time semantic segmentation** (road lanes) with **object detection and multi-object tracking** (vehicles), applied frame-by-frame to driving video.

<p align="center">
  <img src="docs/demo.gif" width="700" alt="Demo: lane segmentation + vehicle tracking"/>
</p>


## Overview

Self-driving perception systems need to understand *where the road is* and *what other vehicles are doing* at the same time. This project builds both capabilities as separate models and then fuses them into one real-time video pipeline:

1. **Lane segmentation** — a DeepLabV3+ model (ResNet50 backbone) trained to produce a binary road-lane mask for every frame.
2. **Vehicle detection** — a YOLO11 model trained to detect cars, trucks, and pedestrians.
3. **Multi-object tracking** — BoT-SORT (with ReID) assigns persistent IDs to detected vehicles across frames.
4. **Combined pipeline** — both models run together on the same video, overlaying the lane mask and the tracked vehicle boxes in a single output.

## Pipeline

```
                 ┌─────────────────────┐
   video frame → │  DeepLabV3+ (ResNet50)│ → binary lane mask
                 └─────────────────────┘
                 ┌─────────────────────┐
   video frame → │      YOLO11 detector  │ → vehicle boxes + classes
                 └─────────────────────┘
                              ↓
                 ┌─────────────────────┐
                 │   BoT-SORT tracker    │ → tracked IDs per vehicle
                 └─────────────────────┘
                              ↓
                    overlay mask + boxes + IDs → output video
```

## Repository Structure

```
.
├── notebooks/
│   ├── 01_lane_segmentation_deeplabv3.ipynb   # DeepLabV3+ training: data loading, mask sanity checks,
│   │                                           # geometric + photometric augmentation, class balancing, training
│   ├── 02_convert_dataset_to_yolo.ipynb       # Converts raw (xmin/ymin/xmax/ymax + CSV) annotations
│   │                                           # into YOLO txt format + data.yaml
│   ├── 03_yolo_vehicle_detection.ipynb        # YOLO11 training on the converted dataset
│   ├── 04_tracking_botsort.ipynb              # BoT-SORT tracking on top of YOLO detections
│   ├── 05_segmentation_tracking_botsort.ipynb # Tracking objects derived directly from the segmentation mask
│   └── 06_combined_pipeline.ipynb             # Full pipeline: segmentation + detection + tracking together
├── docs/
│   └── demo.gif / screenshots
├── requirements.txt
├── .gitignore
└── README.md
```



## Models

| Task | Model | Backbone | Notes |
|---|---|---|---|
| Lane segmentation | DeepLabV3+ | ResNet50 (ImageNet, frozen) | Binary mask, ASPP module, combined Dice + CE loss, median-frequency class balancing |
| Vehicle detection | YOLO11n | — | Trained on converted `car / truck / pedestrian` labels |
| Tracking | BoT-SORT | OSNet (ReID: `osnet_x1_0_msmt17`) | Works both on YOLO boxes and on contours extracted from the segmentation mask |

## Dataset

- Lane segmentation frames + masks: see `dataset link 1` and `dataset link 2` *(paste your Kaggle links here)*
- Vehicle detection annotations: [Udacity self-driving-car annotations](https://github.com/udacity/self-driving-car/tree/master/annotations)

Datasets are **not included** in this repo (too large for Git). Download them using the links above and place them as:

```
dataset/
├── frames/            # lane segmentation input images
├── lane-masks/        # lane segmentation binary masks
├── object-detection-crowdai/   # raw images for vehicle detection
└── labels_crowdai.csv          # raw xmin/ymin/xmax/ymax annotations
```


## Usage

1. Download the datasets and place them under `dataset/` as shown above.
2. Run `01_lane_segmentation_deeplabv3.ipynb` to train the segmentation model → produces `bestmodel.keras`.
3. Run `02_convert_dataset_to_yolo.ipynb` to build `yolo_dataset/` in YOLO format.
4. Run `03_yolo_vehicle_detection.ipynb` to train YOLO → produces `runs/detect/train/weights/best.pt`.
5. Run `06_combined_pipeline.ipynb`, pointing `VIDEO_PATH` to your driving video, to get a combined segmentation + tracking output.

## Key Design Notes

- **Augmentation consistency**: geometric augmentations (flip, rotation) are applied jointly to image *and* mask by stacking them into one tensor before the `RandomFlip`/`RandomRotation` layers — otherwise the mask and image drift out of alignment. Photometric augmentations (brightness/contrast) are applied to the image only, since pixel-value changes don't need a matching mask change.
- **Class imbalance**: lane pixels are a small minority of each frame, so median-frequency balancing is used to weight the loss.
- **Two tracking entry points**: vehicles can be tracked either from YOLO detections (`04_tracking_botsort.ipynb`) or directly from segmentation-mask contours (`05_segmentation_tracking_botsort.ipynb`) — useful for comparing a detection-based vs. a segmentation-based pipeline.

## Limitations / Future Work

- Currently two separate models (segmentation + detection) run independently rather than a single multi-task network — merging them would reduce inference cost.
- No lane-departure or trajectory-planning logic yet; this repo focuses on perception only.
- Could add traffic-sign / traffic-light detection as a third task.

## Acknowledgments

Built as part of a computer vision course project on autonomous-driving perception (real-time segmentation + detection + tracking).

## License

*(Add a license, e.g. MIT, if you want others to freely reuse the code)*
