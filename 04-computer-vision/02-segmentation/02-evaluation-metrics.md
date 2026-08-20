# Evaluation Metrics for Segmentation

After training a segmentation model, we need to evaluate how well it segmented the regions of an image.

In segmentation, instead of predicting a single label for the whole image, the model predicts a class for every pixel. So evaluation metrics are also defined by comparing the predicted pixels against the ground-truth mask.

The most important evaluation metrics in segmentation are:

- Pixel Accuracy
- Mean Pixel Accuracy (mPA)
- Intersection over Union (IoU)
- Mean Intersection over Union (mIoU)
- Dice Coefficient

---

## 1. Pixel Accuracy

### The Concept of Pixel Accuracy

Pixel accuracy checks how many pixels, out of all the pixels in the image, were correctly predicted by the model.

In simple terms:

> This metric asks: what percentage of the image's pixels were classified correctly?

Formula for Pixel Accuracy:

$$ PA=\frac{Number\ of\ Correct\ Pixels}{Total\ Number\ of\ Pixels}$$

For example, if an image has 10,000 pixels and the model correctly predicts 9,500 of them:

$$
PA=\frac{9500}{10000}=0.95
$$

So the pixel accuracy would be 95%.

### The Problem With Pixel Accuracy

The main issue with this metric shows up when the number of pixels per class is imbalanced.

For example, in an image:

- The background class might make up 95% of the image.
- The object of interest might only make up 5% of the image.

If the model predicts the entire image as background:

- All background pixels are classified correctly.
- The actual object is completely missed.

But the pixel accuracy would still be:

$$
PA=\frac{95}{100}=95\%
$$

even though the model has captured no information about the actual object at all.

So pixel accuracy on its own isn't a reliable metric for evaluating segmentation, and it's usually only used as a supplementary measure.

---

## 2. Mean Pixel Accuracy (mPA)

### The Concept of Mean Pixel Accuracy

To address the problem with pixel accuracy, mean pixel accuracy was introduced.

In this metric, accuracy is first calculated separately for each class, and then those values are averaged.

In other words:

> First, we check how accurately the model detected each individual class, then we average the performance across all classes.

Formula for Mean Pixel Accuracy:

$$
mPA=\frac{1}{N}\sum_{i=1}^{N}PA_i
$$

Where:

- $N$ is the number of classes.
- $PA_i$ is the pixel accuracy for class $i$.

---

### Example: Mean Pixel Accuracy

Suppose an image has 100 pixels and contains two classes:

- Background
- Tire

The actual pixel counts:

- Background pixels: 90
- Tire pixels: 10

The model's predictions:

- All 90 background pixels were classified correctly.
- Out of the 10 tire pixels, only 5 were classified correctly.

#### Background class accuracy

$$
PA_{background}=\frac{90}{90}=100\%
$$

#### Tire class accuracy

$$
PA_{tire}=\frac{5}{10}=50\%
$$

The mean pixel accuracy is then:

$$
mPA=\frac{100+50}{2}=75\%
$$

If we had only used ordinary pixel accuracy:

$$
PA=\frac{90+5}{100}=95\%
$$

the model would appear to be performing very well. But mean pixel accuracy reveals that:

- The background class was detected well.
- The tire class was only correctly detected 50% of the time.

So mPA reduces the effect of class imbalance.

### Pixel Accuracy vs. Mean Pixel Accuracy

| Metric | How It's Calculated | Weakness |
|---|---|---|
| Pixel Accuracy | Total correct pixels divided by total pixels | Large classes dominate the score |
| Mean Pixel Accuracy | Accuracy per class, then averaged | Reduces the influence of large classes |

### The Limitation of Mean Pixel Accuracy

Although mPA is an improvement over pixel accuracy, it still has a shortcoming.

This metric only checks how many pixels of each class were correctly predicted. But it doesn't check:

- Whether the region is located in the correct place.
- How similar the predicted mask actually is to the ground-truth mask in shape and position.

For this reason, metrics like IoU and mIoU are generally considered more important in modern segmentation.

---

## 3. Intersection over Union (IoU)

IoU measures the overlap between the ground-truth mask and the predicted mask.

The core idea:

> How much of the predicted region overlaps with the actual region?

Formula for IoU:

$$
IoU=\frac{A\cap B}{A\cup B}
$$

Where:

- Region $A$ is the ground-truth mask.
- Region $B$ is the predicted mask.
- $A\cap B$ is the overlapping region between the two masks.
- $A\cup B$ is the total combined area of both regions.

IoU ranges between 0 and 1:

- A value of 1 means the prediction perfectly matches the ground-truth mask.
- A value of 0 means there is no overlap at all.

---

## 4. Mean Intersection over Union (mIoU)

In multi-class problems, an IoU value is calculated for each class, and then averaged across all classes.

Formula:

$$
mIoU=\frac{1}{N}\sum_{i=1}^{N}IoU_i
$$

Where:

- $N$ is the number of classes.
- $IoU_i$ is the IoU value for class $i$.

mIoU is one of the most important standard metrics in segmentation, since — beyond just counting correct pixels — it also captures the degree of overlap and the object's position.

---

## 5. Dice Coefficient

The Dice coefficient also measures the similarity between two masks.

Formula for Dice:

$$
Dice=\frac{2|A\cap B|}{|A|+|B|}
$$

Where:

- $|A\cap B|$ is the number of shared pixels between the two masks.
- $|A|$ is the number of pixels in the ground-truth mask.
- $|B|$ is the number of pixels in the predicted mask.

### Example: Dice

Suppose:

- The ground-truth mask has 100 pixels.
- The predicted mask has 80 pixels.
- The number of shared pixels is 60.

So:

$$
Dice=\frac{2\times60}{100+80}
$$

$$
Dice=\frac{120}{180}=0.67
$$

This means the two masks are 67% similar.

---

## IoU vs. Dice

Both metrics range between 0 and 1.

The main differences:

- IoU is more geometrically intuitive, representing the ratio of overlap to union.
- Dice is more sensitive to increases in the overlap area, and typically reacts to changes more sharply.

---

## Comparing Segmentation Metrics

| Metric               | Concept                                     | Use Case                                   |
| --------------------- | -------------------------------------------- | -------------------------------------------- |
| Pixel Accuracy         | Percentage of correctly predicted pixels     | A simple metric, but sensitive to class imbalance |
| Mean Pixel Accuracy    | Accuracy calculated separately per class     | Reduces the influence of large classes         |
| IoU                     | Overlap between ground-truth and predicted masks | Precise evaluation of a single class            |
| mIoU                    | Average IoU across all classes                | One of the primary segmentation metrics         |
| Dice                    | More sensitive to the overlap region          | Widely used in medical image segmentation       |

---

## Summary

In segmentation, pixel accuracy isn't always reliable, since it's heavily influenced by dominant classes like background.

Mean pixel accuracy reduces this problem by evaluating each class separately, but it still doesn't account for the location and shape of the object.

For this reason, metrics like:

- mIoU
- Dice score

are generally used as the primary metrics for evaluating segmentation models, since — beyond just checking whether pixels are correct — they also measure how much the predicted mask overlaps with and resembles the ground-truth mask.
