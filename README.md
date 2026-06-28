# Dental Caries Segmentation -- Model Evaluation

A quantitative evaluation framework for comparing U-Net variant segmentation models on dental X-ray caries detection, built as part of a research project currently under peer review.

---

## Role

I was responsible for the evaluation pipeline only. Model training, dataset curation, clinical methodology, and the Streamlit webapp deployment were handled by other team members. My contribution covers everything in this repository: the island-counting detection metric, per-model precision/recall/F1 aggregation, and visualisation of predictions against ground truth masks.

---

## What it does

Segmentation models output pixel-level probability maps. Standard pixel-level metrics (Intersection over Union/IoU, Dice coefficient) measure mask overlap but do not directly answer the clinical question: **how many caries lesions did the model find, and how many did it miss?**

This evaluation reframes the problem at the lesion level. Each connected region of predicted pixels is treated as a single detection, and each connected region in the ground truth is a single lesion. A detection counts as a true positive only if it overlaps with a ground truth lesion by at least one pixel.

![Example comparison](images/example.png)

---

## Method

**Island counting via DFS.** Connected components in binary mask arrays are counted using a depth-first search adapted from a standard 2D island-counting algorithm. 8-connectivity is used (including diagonal neighbours). Regions smaller than 3 pixels are discarded as noise.

**Per-image metrics.** For each of the 50 test images and each of the 14 model variants, the pipeline computes:

- GT (Ground Truth) -- number of ground truth lesions
- TP (True Positive) -- ground truth lesions that have at least one overlapping predicted pixel
- FP (False Positive) -- predicted regions with no overlap with any ground truth lesion
- FN (False Negative) -- ground truth lesions with no overlapping prediction (FN = GT - TP)

**Aggregation.** Per-image counts are summed across the 50-image test set to produce model-level precision, recall, and F1.

---

## Results

| Model | TP | FP | FN | Precision | Recall | F1 |
|---|---|---|---|---|---|---|
| OG U-Net | 101 | 31 | 49 | 0.765 | 0.673 | 0.716 |
| VGG16 U-Net | 105 | 52 | 45 | 0.669 | 0.700 | 0.684 |
| EfficientNet-B5 U-Net | 127 | 61 | 23 | 0.676 | 0.847 | 0.752 |
| ResNet U-Net | 102 | 41 | 48 | 0.713 | 0.680 | 0.696 |
| MobileNet U-Net | 107 | 49 | 43 | 0.686 | 0.713 | 0.699 |
| EfficientNet-B5_1x | 127 | 61 | 23 | 0.676 | 0.847 | 0.752 |
| EfficientNet-B5_1.5x | 124 | 47 | 26 | 0.725 | 0.827 | 0.773 |
| EfficientNet-B5_2x | 123 | 48 | 27 | 0.719 | 0.820 | 0.766 |
| EfficientNet-B5_2.5x | 125 | 43 | 25 | 0.744 | 0.833 | 0.786 |
| EfficientNet-B5_3x | 121 | 31 | 29 | 0.796 | 0.807 | **0.801** |
| EfficientNet-B5_3.5x | 118 | 29 | 32 | **0.803** | 0.787 | 0.795 |
| EfficientNet-B5_150 | 105 | 49 | 45 | 0.682 | 0.700 | 0.691 |
| EfficientNet-B5_250 | 112 | 51 | 38 | 0.687 | 0.747 | 0.716 |
| EfficientNet-B5_350 | 119 | 41 | 31 | 0.744 | 0.793 | 0.768 |

All models were evaluated against the same 50-image test set (150 ground truth lesions total). EfficientNet-B5 with 3x augmentation achieved the best F1 (0.801), while keeping precision and recall around 80%.

---

## Note on data availability

The dental X-ray images and trained model weights are omitted from this repository. The images are patient data subject to institutional data access restrictions. The model weights are part of the research contribution and will not be released in this repository. The evaluation code runs on pre-exported `.npy` prediction arrays and ground truth masks, which are also omitted for the same reason.

---

## Stack

- NumPy -- mask array operations and DFS implementation
- pandas -- per-image result compilation and CSV export
- Matplotlib -- ground truth / prediction / combined overlay visualisation
- scikit-learn -- confusion matrix display
- Google Colab -- execution environment