# Evaluation Metrics in Object Detection

> **How do we know whether our Object Detection model is performing well?**

To answer this question, we need **Evaluation Metrics**.

In Machine Learning and Deep Learning, different tasks require different evaluation metrics.

For example:

* **Regression** → MAE, MSE, RMSE
* **Classification** → Accuracy, Precision, Recall, F1-Score
* **Object Detection** → Precision, Recall, IoU, AP, mAP, and other metrics

Object Detection is more challenging to evaluate because the model must perform two tasks:

1. **Classification** → determine the object's class.
2. **Localization** → determine the object's location using a Bounding Box.

Therefore, we need to evaluate both the **class prediction** and the **quality of the predicted Bounding Box**.

---

# 1. Evaluation in Classification

Before discussing Object Detection, it is useful to briefly review how model predictions are evaluated in Classification.

Suppose we have three classes:

```text
Class 1
Class 2
Class 3
```

For a given image, the model may produce:

```text
Class 1 → 0.4
Class 2 → 0.3
Class 3 → 0.3
```

These values represent the model's scores or probabilities for the different classes, depending on the model and output formulation.

If these values are probabilities produced by a Softmax layer, their sum is:

```text
0.4 + 0.3 + 0.3 = 1.0
```

The class with the highest score is typically selected as the predicted class:

```text
Prediction = Class 1
Score = 0.4
```

However, a score of `0.4` indicates relatively low confidence compared with a prediction such as `0.95`.

---

# 2. Confidence Score

In Object Detection, each predicted detection is typically associated with a **Confidence Score** or detection score.

For example:

```text
Detection 1 → 0.95
Detection 2 → 0.72
Detection 3 → 0.31
```

A higher score generally indicates that the model considers the detection more likely to be valid.

However, the exact meaning and calculation of a confidence score depend on the detection architecture.

Therefore, it is better to think of confidence as:

> **A score that represents the model's confidence in a predicted detection.**

---

# 3. Confidence Threshold

We can define a **Confidence Threshold** to decide which predictions should be kept during inference.

For example:

```text
Confidence Threshold = 0.5
```

Then:

```text
0.95 → Accept
0.72 → Accept
0.31 → Reject
```

In general:

```text
Confidence ≥ Threshold → Keep
Confidence < Threshold → Discard
```

The confidence threshold is typically applied during **Inference / Post-processing**, rather than being part of the model's training objective itself.

---

# 4. Evaluation in Object Detection

Object Detection is more complicated than Classification because a detection must be correct in terms of both:

```text
Classification
      +
Localization
```

For example, suppose the model predicts:

```text
Class = Car
```

If the predicted Bounding Box is far away from the actual car, the detection should not be considered fully correct.

Therefore, we need a way to measure how well the predicted Bounding Box matches the Ground Truth Bounding Box.

This is where **IoU** becomes important.

---

# 5. What Is IoU?

**IoU** stands for:

> **Intersection over Union**

IoU measures the overlap between:

* **Ground Truth Bounding Box**
* **Predicted Bounding Box**

The formula is:

[
IoU =
\frac{\text{Area of Intersection}}
{\text{Area of Union}}
]

or:

```text
IoU = Intersection / Union
```

where:

* **Intersection** → the area shared by the two boxes
* **Union** → the total area covered by either box

---

# 6. Intersection and Union

## Intersection

The **Intersection** is the region where the Ground Truth and Predicted Bounding Boxes overlap.

```text
        Ground Truth
     ┌───────────────┐
     │               │
     │    ┌──────────┼──────┐
     │    │Intersection     │
     └────┼──────────┘      │
          │                 │
          └─────────────────┘
             Prediction
```

The intersection represents the common area between the two boxes.

---

## Union

The **Union** represents the total area covered by the Ground Truth and Predicted Bounding Boxes.

Therefore:

```text
IoU = Intersection Area / Union Area
```

The more the two boxes overlap, the higher the IoU.

---

# 7. IoU Range

IoU always falls between `0` and `1`:

[
0 \leq IoU \leq 1
]

### IoU = 0

The two bounding boxes have no overlap.

```text
Ground Truth          Prediction

┌─────────┐            ┌─────────┐
│         │            │         │
│         │            │         │
└─────────┘            └─────────┘
```

Therefore:

```text
Intersection = 0
IoU = 0
```

---

### IoU = 1

The two bounding boxes are identical.

```text
┌─────────────────┐
│                 │
│ Ground Truth =  │
│ Prediction      │
│                 │
└─────────────────┘
```

In this case:

```text
Intersection = Union
IoU = 1
```

---

# 8. Interpreting IoU

In general:

```text
IoU → 1  → Strong overlap
IoU → 0  → Little or no overlap
```

For example:

```text
IoU = 0.90 → Very strong overlap
IoU = 0.70 → Strong overlap
IoU = 0.50 → Moderate overlap
IoU = 0.20 → Weak overlap
IoU = 0.00 → No overlap
```

However, whether a particular IoU value is considered a successful detection depends on the **IoU Threshold** used during evaluation.

---

# 9. IoU Threshold

We can define an IoU threshold to determine whether a predicted bounding box sufficiently overlaps the Ground Truth.

For example:

```text
IoU Threshold = 0.5
```

Then, conceptually:

```text
IoU ≥ 0.5 → Sufficient overlap
IoU < 0.5 → Insufficient overlap
```

The IoU threshold plays an important role in determining whether a detection is counted as a **True Positive** or **False Positive**.

---

# 10. True Positive in Object Detection

A predicted detection can be considered a **True Positive (TP)** when it satisfies the evaluation criteria.

In a simplified setting, two important conditions are:

1. The predicted class matches the Ground Truth class.
2. The predicted Bounding Box has sufficient overlap with the Ground Truth.

Conceptually:

```text
IoU ≥ IoU Threshold
        AND
Predicted Class = Ground Truth Class
```

For example:

```text
IoU = 0.80
IoU Threshold = 0.50
Class = Correct
```

Therefore:

```text
True Positive
```

The exact matching procedure can be more sophisticated in real evaluation pipelines, especially when multiple predictions and multiple Ground Truth objects are present.

---

# 11. False Positive in Object Detection

A **False Positive (FP)** occurs when a prediction does not correspond to a valid Ground Truth detection according to the evaluation rules.

This can happen in several ways.

## Case 1 — Insufficient Bounding Box Overlap

Suppose:

```text
IoU = 0.20
IoU Threshold = 0.50
```

Even if the predicted class is correct, the localization is insufficient.

Therefore, the detection may be counted as a False Positive.

---

## Case 2 — Incorrect Class

Suppose:

```text
Ground Truth → Balloon
Prediction    → Car
```

and the bounding box overlaps strongly:

```text
IoU = 0.90
```

The localization is good, but the class is incorrect.

Therefore, the prediction is not a correct detection and may be counted as a False Positive.

---

# 12. False Negative in Object Detection

A **False Negative (FN)** occurs when a Ground Truth object is not successfully detected.

For example, suppose an image contains:

```text
10 Ground Truth Objects
```

but the model successfully detects only:

```text
4 Objects
```

The remaining objects that were not detected contribute to False Negatives.

Therefore:

```text
FN = Missed Ground Truth Objects
```

---

# 13. Precision

Using TP and FP, we can define **Precision**.

Precision answers:

> **Of all the detections made by the model, how many were correct?**

The formula is:

[
Precision =
\frac{TP}{TP + FP}
]

Conceptually:

```text
Precision =
Correct Detections
───────────────────
All Accepted Detections
```

---

# 14. Precision Example

Suppose:

```text
TP = 4
FP = 1
```

Then:

[
Precision = \frac{4}{4+1}
]

```text
Precision = 0.80
```

Therefore:

```text
Precision = 80%
```

This means that 80% of the detections counted by the evaluation process were correct.

---

# 15. Why Precision Alone Is Not Enough

Precision does not directly account for False Negatives.

A model could make only a few predictions, but if most of those predictions are correct, it could achieve high Precision.

For example:

```text
Precision = 95%
```

This sounds excellent.

But suppose the model detects only a small portion of the objects that actually exist.

In that case, the model may still perform poorly.

Therefore, we also need **Recall**.

---

# 16. Recall

Recall answers:

> **Of all the objects that actually exist, how many did the model successfully detect?**

The formula is:

[
Recall =
\frac{TP}{TP + FN}
]

Conceptually:

```text
Recall =
Correct Detections
──────────────────────
All Ground Truth Objects
```

---

# 17. Recall Example

Suppose:

```text
TP = 4
FN = 5
```

Then:

[
Recall = \frac{4}{4+5}
]

```text
Recall = 4 / 9
```

Therefore:

```text
Recall ≈ 44.4%
```

This means the model successfully detected approximately 44.4% of the Ground Truth objects.

---

# 18. Precision vs. Recall

The difference can be summarized with two questions.

### Precision

> **Of the objects I detected, how many were correct?**

```text
Precision → Focuses on False Positives
```

### Recall

> **Of the objects that actually existed, how many did I detect?**

```text
Recall → Focuses on False Negatives
```

Therefore:

```text
Precision → FP is important
Recall    → FN is important
```

---

# 19. A Complete Precision and Recall Example

Suppose an image contains:

```text
10 Ground Truth Objects
```

and the model produces:

```text
TP = 4
FP = 1
FN = 6
```

### Precision

[
Precision = \frac{4}{4+1}
]

```text
Precision = 0.80 = 80%
```

### Recall

[
Recall = \frac{4}{4+6}
]

```text
Recall = 0.40 = 40%
```

The model has relatively high Precision but low Recall.

This means:

> When the model makes a detection, it is often correct, but it misses many of the objects that actually exist.

---

# 20. The Role of Thresholds

Thresholds play an important role in Object Detection evaluation.

Two important thresholds are:

* **Confidence Threshold**
* **IoU Threshold**

These thresholds have different purposes.

---

## Confidence Threshold

The Confidence Threshold determines whether a prediction should be kept based on the model's confidence.

```text
Confidence ≥ Threshold
        ↓
     Keep
```

---

## IoU Threshold

The IoU Threshold determines whether the predicted Bounding Box overlaps sufficiently with the Ground Truth.

```text
IoU ≥ Threshold
        ↓
Sufficient Localization
```

Therefore:

```text
Confidence Threshold
        ↓
Should this prediction be kept?

IoU Threshold
        ↓
Is the localization sufficiently accurate?
```

These two concepts should not be confused.

---

# 21. Effect of Confidence Threshold

Suppose the model produces:

```text
0.20
0.40
0.60
0.80
0.90
```

If:

```text
Confidence Threshold = 0.50
```

then:

```text
0.20 → Reject
0.40 → Reject
0.60 → Accept
0.80 → Accept
0.90 → Accept
```

Now suppose we increase the threshold:

```text
Confidence Threshold = 0.80
```

Then:

```text
0.20 → Reject
0.40 → Reject
0.60 → Reject
0.80 → Accept
0.90 → Accept
```

The model becomes more selective.

In general, increasing the confidence threshold can lead to:

```text
Threshold ↑
     ↓
Fewer predictions are accepted
     ↓
FP may decrease
     ↓
Precision may increase
     ↓
Some valid detections may be removed
     ↓
FN may increase
     ↓
Recall may decrease
```

The exact behavior depends on the model and dataset, but this illustrates the general Precision-Recall trade-off.

---

# 22. Precision-Recall Trade-off

Precision and Recall often have an inverse relationship as the confidence threshold changes.

If we use a high threshold:

```text
Only high-confidence predictions are kept
```

This can reduce incorrect detections:

```text
FP ↓
Precision ↑
```

but some valid objects may be missed:

```text
FN ↑
Recall ↓
```

On the other hand, if we lower the threshold:

```text
More predictions are accepted
```

the model may detect more real objects:

```text
Recall ↑
```

but it may also produce more incorrect detections:

```text
FP ↑
Precision ↓
```

This is known as the:

> **Precision-Recall Trade-off**

---

# 23. Precision-Recall Curve

If we continuously change the confidence threshold, the corresponding Precision and Recall values also change.

For example:

| Confidence Threshold | Precision | Recall |
| -------------------: | --------: | -----: |
|                  0.1 |      0.50 |   0.95 |
|                  0.3 |      0.65 |   0.85 |
|                  0.5 |      0.75 |   0.70 |
|                  0.7 |      0.85 |   0.55 |
|                  0.9 |      0.95 |   0.30 |

These points can be plotted to form a:

> **Precision-Recall Curve**

The curve provides a more complete view of the model's behavior across different confidence thresholds.

---

# 24. Why Precision and Recall Alone Are Not Enough

Suppose one model achieves:

```text
Precision = 95%
Recall = 30%
```

The model is very selective and most of its accepted detections are correct, but it misses many objects.

Another model may achieve:

```text
Precision = 60%
Recall = 95%
```

This model detects most of the objects, but it also produces more incorrect detections.

Therefore, comparing models using only one operating point of Precision or Recall may not provide a complete picture.

This leads us to more comprehensive metrics such as:

> **Average Precision (AP)**

and:

> **mean Average Precision (mAP)**

---

# 25. Average Precision (AP)

**Average Precision (AP)** summarizes the Precision-Recall performance of a detector for a particular class under a specified evaluation protocol.

Conceptually:

```text
Confidence Thresholds
        ↓
Precision / Recall
        ↓
Precision-Recall Curve
        ↓
Average Precision (AP)
```

AP therefore provides a way to summarize performance across different confidence thresholds rather than evaluating the detector at only one threshold.

The exact calculation of AP depends on the evaluation protocol.

---

# 26. mean Average Precision (mAP)

When there are multiple object classes, we can calculate AP separately for each class.

For example:

```text
Airplane → AP
Car      → AP
Person   → AP
Motorcycle → AP
```

Then we can calculate the mean:

[
mAP =
\frac{\sum AP_{classes}}
{\text{Number of Classes}}
]

Conceptually:

```text
        AP
         │
         ├── Airplane
         ├── Car
         ├── Person
         └── Motorcycle
              ↓
          Average
              ↓
             mAP
```

Thus:

> **AP evaluates detection performance for a class, while mAP summarizes AP across multiple classes.**

---

# 27. The Importance of the IoU Threshold in AP and mAP

AP and mAP are not meaningful without specifying the evaluation protocol, including the IoU criterion used to determine whether a prediction matches a Ground Truth object.

For example, we may evaluate at:

```text
IoU = 0.50
```

which is commonly referred to as:

```text
AP@0.50
```

or:

```text
mAP@0.50
```

We can also evaluate across multiple IoU thresholds.

For example:

```text
0.50
0.55
0.60
...
0.95
```

This leads to metrics such as:

```text
mAP@0.50:0.95
```

which are commonly used in modern Object Detection benchmarks.

---

# 28. The Complete Evaluation Pipeline

The relationship between the main concepts can be summarized as:

```text
Model Predictions
        ↓
Confidence Score
        ↓
Confidence Threshold
        ↓
Keep / Discard Predictions
        ↓
Match Predictions with Ground Truth
        ↓
IoU
        ↓
IoU Threshold
        ↓
TP / FP / FN
        ↓
Precision / Recall
        ↓
Precision-Recall Curve
        ↓
Average Precision (AP)
        ↓
mean Average Precision (mAP)
```

This pipeline shows how the different evaluation concepts are connected.

---

# 29. Important Distinctions

There are several concepts that should not be confused.

### Confidence Score

> How confident is the model in a predicted detection?

### Confidence Threshold

> Which predictions should be kept based on their confidence?

### IoU

> How much do the Predicted and Ground Truth Bounding Boxes overlap?

### IoU Threshold

> How much overlap is required for a prediction to count as a valid match?

### Precision

> How many accepted detections are correct?

### Recall

> How many Ground Truth objects were successfully detected?

### AP

> How well does the detector perform across different confidence thresholds for a particular class?

### mAP

> How well does the detector perform across classes, according to the specified AP evaluation protocol?

---

# 30. Summary

The main evaluation concepts in Object Detection are:

### 1. Confidence Score

Represents the model's confidence in a predicted detection.

### 2. Confidence Threshold

Determines which predictions should be kept during inference or evaluation.

### 3. IoU

Measures the overlap between a Predicted Bounding Box and a Ground Truth Bounding Box.

[
IoU =
\frac{Intersection}{Union}
]

### 4. IoU Threshold

Defines how much overlap is required for a predicted box to sufficiently match a Ground Truth box.

### 5. True Positive

A valid detection that satisfies the class and localization criteria.

### 6. False Positive

A predicted detection that does not correspond to a valid Ground Truth match.

### 7. False Negative

A Ground Truth object that was not successfully detected.

### 8. Precision

[
Precision =
\frac{TP}{TP+FP}
]

Measures how many accepted detections are correct.

### 9. Recall

[
Recall =
\frac{TP}{TP+FN}
]

Measures how many Ground Truth objects were successfully detected.

### 10. Precision-Recall Curve

Shows how Precision and Recall change as the confidence threshold changes.

### 11. Average Precision (AP)

Summarizes the Precision-Recall performance for a class under a specified evaluation protocol.

### 12. mean Average Precision (mAP)

Aggregates AP across multiple classes and, depending on the benchmark, potentially across multiple IoU thresholds.

---

# 31. Learning Path

The evaluation concepts can be learned in the following order:

```text
Object Detection
       ↓
Confidence Score
       ↓
Confidence Threshold
       ↓
Bounding Box Matching
       ↓
IoU
       ↓
IoU Threshold
       ↓
True Positive / False Positive / False Negative
       ↓
Precision / Recall
       ↓
Precision-Recall Curve
       ↓
Average Precision (AP)
       ↓
mean Average Precision (mAP)
       ↓
mAP@0.50
       ↓
mAP@0.50:0.95
```

The key idea is:

> **Object Detection evaluation must measure both classification correctness and localization quality. IoU evaluates bounding-box overlap, while Precision, Recall, AP, and mAP provide increasingly comprehensive measures of detection performance.**
