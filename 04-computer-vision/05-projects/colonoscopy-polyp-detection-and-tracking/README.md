# Real-Time Polyp Detection & Auto-Frame-Capture for Colonoscopy Video (YOLO)

A YOLO-based computer vision assistant for colonoscopy: it detects polyps in real time as the doctor moves through the video feed, tracks them frame-to-frame, and **automatically saves every frame where a polyp was detected** (with a timestamp filename) so the doctor can review findings after the procedure without re-watching the whole video.

<p align="center">
  <img src="docs/demo.gif" width="700" alt="Demo: real-time polyp detection and frame capture"/>
</p>


## Motivation

During a colonoscopy, a doctor watches a live video feed and has to spot polyps in real time — small, fast-moving, easy to miss. This project acts as a second pair of eyes:

- Runs polyp detection on every frame of the live/recorded feed.
- Whenever a polyp is detected, saves that frame to disk with a timestamp (`{seconds}.{centiseconds}.jpg`), plus a full annotated output video.
- The doctor can review the saved frames/video afterward as a report, instead of needing to catch everything live.

## Pipeline

```
video / live feed
      │
      ▼
YOLO11 detector + tracker (model.track, persist=True)
      │
      ├── if a polyp box is detected in the frame:
      │        → save annotated frame to report/  (named by video timestamp)
      │
      └── always: write annotated frame to report/tracked_output_video.mp4
```


## Dataset

- Polyp detection dataset (Roboflow, YOLO format): [BSC polyp dataset](https://universe.roboflow.com/bsc-9d5aw/bsc-train_psc1toc5/dataset/7)

Not included in this repo — download it from the link above and place it as:

```
dataset/
├── data.yaml
├── train/
├── valid/
└── test/
```

## Model & Training

- **Model**: YOLO11n (nano — fast enough for near real-time use on modest hardware; a larger variant can be used for higher accuracy if latency allows).
- **Result**: mAP ≈ 83% on the validation set with the settings below (larger models can push this higher).

### Augmentation choices — and why

Augmentation here was deliberately picked for the *medical imaging domain*, not the generic defaults:

| Augmentation | Value | Reasoning |
|---|---|---|
| `hsv_h` (hue) | `0.0` | Tissue/polyp color is diagnostically meaningful (e.g. reddish tissue, whitish lesions) — shifting hue could hide or fake a diagnostic signal. Disabled. |
| `hsv_s`, `hsv_v` (saturation/value) | `0.3` | Lighting/exposure varies a lot with camera angle and scope movement — safe to vary. |
| `flipud`, `fliplr` | `0.5` | Colonoscopy footage has no fixed "up/down" or "left/right" orientation, so flipping doesn't break the meaning of the image. |
| `degrees` | `0.15` | Small rotations tolerate scope movement without distorting anatomy. |
| `translate`, `scale`, `shear`, `perspective`, `mosaic`, `mixup`, `copy_paste` | `0.0` | Disabled — kept the frame geometry close to the real endoscopic view. |

> **Note on domain expertise**: color/geometry augmentation choices for medical imagery should ideally be validated with a domain expert (e.g. a physician) — this project's choices are a reasonable starting point, not a clinical guarantee.



## Usage

1. Download the dataset from the Roboflow link above and place it under `dataset/`.
2. Run `01_train_yolo_polyp_detection.ipynb` → produces `runs/detect/train/weights/best.pt`.
3. Run `02_track_and_save_frames.ipynb`, pointing `VIDEO_PATH` to a colonoscopy video → produces:
   - `report/tracked_output_video.mp4` — full annotated video
   - `report/{timestamp}.jpg` — one saved frame per detection event, for later review

## Key Design Notes

- **`persist=True` in `model.track()`**: tells YOLO's tracker that consecutive frames belong to the same video, so it can maintain object IDs across frames instead of treating each frame as an independent detection.
- **`conf=0.8`**: a fairly high confidence threshold, intentionally conservative to reduce false-positive frame saves in a medical review context — tune based on desired precision/recall trade-off.
- **Timestamp-based filenames**: frames are named by their position in the video (`seconds.centiseconds.jpg`), so a reviewer can jump straight to that moment in the source video.

## Limitations / Future Work

- Detection-based, not diagnosis-based — this tool flags regions of interest, it does not classify polyp type or malignancy.
- No deduplication of frames when the same polyp is tracked over many consecutive frames — the report may contain many near-duplicate frames of one detection event; grouping by tracker ID could reduce this.
- Augmentation/threshold choices should be reviewed with a medical domain expert before any clinical use.
- This is a research/educational project and **is not a certified medical device**.

## Acknowledgments

Built as part of a computer vision course project applying real-time detection and tracking to a medical imaging use case (colonoscopy).

