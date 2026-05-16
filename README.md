# YOLOv8-Object-Tracking-System

# VisDrone Object Detection with YOLOv8

Human and vehicle detection on drone imagery using YOLOv8, trained on the VisDrone2019 dataset.

## Overview

This project fine-tunes a YOLOv8 model to detect humans and vehicles in aerial drone images.
The pipeline includes annotation cleaning, contrast enhancement, image tiling,
augmentation, and a human/car counting system at inference time.

## Dataset

VisDrone Dataset — a large-scale benchmark
collected by drones across various locations, environments, and crowd densities.

| Split | Images |
|-------|--------|
| Train | 6,471  |
| Val   | 548    |
| Test  | 1,610  |

**10 object classes:**

| ID | Class           | Annotation Count |
|----|-----------------|-----------------|
| 0  | pedestrian      | 79,337          |
| 1  | people          | 27,059          |
| 2  | bicycle         | 10,480          |
| 3  | car             | 144,867         |
| 4  | van             | 24,956          |
| 5  | truck           | 12,875          |
| 6  | tricycle        | 4,812           |
| 7  | awning-tricycle | 3,246           |
| 8  | bus             | 5,926           |
| 9  | motor           | 29,647          |

## Dataset Characteristics & Challenges

- **Scale variation** — objects appear from a few pixels (distant) to clearly structured (close-up).
- **Lighting inconsistency** — images taken at different times of day produce varying shadows.
- **Background inconsistency** — concrete floors, greenery, indoor courts (reddish backgrounds).
- **Blur** — some images are blurry, obscuring object shapes and fine details.
- **Viewing angle variation** — nadir (straight down) to oblique angles.
- **Class imbalance** — `car` dominates; `awning-tricycle` and `tricycle` are highly imbalanced.

## Preprocessing & Augmentation

### Annotation Cleaning
- Every label must have exactly 5 attributes: `class x y width height`
- Labels failing this check are discarded along with their images
- Bounding boxes smaller than a minimum pixel threshold after scaling to 640×640 are removed

### CLAHE (Contrast Limited Adaptive Histogram Equalization)
Applied on the L-channel of LAB colorspace to handle the heavy shadow and low-light
conditions common in drone footage. Performs local histogram equalization tile-by-tile
to evenly distribute pixel values without blowing out bright regions.

### Augmentation
Introduces diversity to prevent the model from memorising patterns:
- Gaussian noise
- Random 90° rotation
- Brightness and contrast jitter
- HSV hue/saturation/value tweaking

### SAHI-style Tiling
Addresses the core challenge of small objects being destroyed by downscaling:
- Full images are sliced into overlapping 640×640 tiles
- Each tile is passed to YOLO at full resolution
- Bounding boxes are remapped into tile-local coordinates
- A visibility threshold determines whether a box belongs to a tile —
  boxes with less than the threshold fraction of their area inside the tile are discarded

## Model

| Setting       | Value      |
|---------------|------------|
| Architecture  | YOLOv8s    |
| Pretrained on | COCO       |
| Image size    | 640×640    |
| Epochs        | 50         |
| Batch size    | 16         |
| Optimizer     | AdamW      |
| Learning rate | 1e-3       |

## Results

Inference run on 1,610 test images after 50 epochs:

| Metric    | Score  |
|-----------|--------|
| Precision | 0.4338 |
| Recall    | 0.3108 |
| mAP50     | 0.2804 |
| mAP50-95  | 0.1584 |

### Detection Counts on Test Set

| Metric           | Value    |
|------------------|----------|
| Images processed | 1,610    |
| Total humans     | 7,755    |
| Total cars       | 23,802   |
| Total time       | 105.73s  |

## Discussion

The model performs reasonably well given only 50 epochs of fine-tuning.
It successfully detects humans even under low lighting conditions as seen
in the output images.
