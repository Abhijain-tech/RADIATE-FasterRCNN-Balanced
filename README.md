# Robust Radar Perception in Adverse Weather (Fog & Snow)

An AI-driven object detection project for autonomous vehicle perception in severe weather conditions using the RADIATE dataset. 

The project focuses on mitigating extreme class imbalance in FMCW radar data to accurately detect both dominant objects (cars) and rare vulnerable road users (pedestrians, bicycles) in heavy fog and snow. Multiple evaluation metrics are compared to identify the effectiveness of targeted data oversampling.

## Project Overview

Optical sensors like cameras and LiDARs severely degrade in adverse weather conditions, leading to autonomous driving failures. While FMCW radar can penetrate heavy fog and snow, it inherently suffers from extreme class imbalance. The radar reflections of large metal cars completely overshadow the reflections of small, non-metallic objects like pedestrians.

In this project, the RADIATE radar dataset is used to develop and evaluate a robust Deep Learning object detection model. 

The project evaluates the performance difference between:
- A Standard Baseline Model (Imbalanced data)
- A Proposed Target-Aware Model (Balanced via Minority Oversampling)

Performance is evaluated using:
- Precision
- Recall
- F1-Score (mAP Estimate)

Additional analysis includes qualitative visual tracking under severe radar clutter and a heuristic 200-frame temporal gap evaluation to prove model robustness.

## Dataset

The project uses the RADIATE (Radar Dataset In Adverse WeaThEr) dataset, specifically focusing on the Navtech Cartesian FMCW radar scans.

The dataset contains run-to-run driving sequences recorded in extreme weather. Five specific subsets are considered in this project:
- fog_6_0
- fog_8_0
- fog_8_1
- fog_8_2
- snow_1_0

### The Class Imbalance Problem
The original subset contained 1,714 keyframes heavily biased towards the 'Car' class. To solve this, **Target-Aware Minority Oversampling** was applied. Frames containing minority classes (Vans, Trucks, Buses, Motorbikes, Bicycles, Pedestrians, Group of Pedestrians) were replicated by a factor of 3. 

- **Original Dataset Size:** 1,714 frames
- **Balanced Dataset Size:** 2,722 frames

*The original RADIATE dataset is not redistributed in this repository. Please obtain the dataset from the official source.*

## Objectives

The main objectives of this project are:
1. Load and preprocess the RADIATE radar dataset and JSON annotations.
2. Apply Target-Aware Minority Oversampling to fix 'Car' dominance.
3. Prepare image tensors and bounding box coordinates for deep learning.
4. Configure a Faster R-CNN architecture with a ResNet-50 FPN backbone.
5. Train the model using a stable SGD Optimizer.
6. Evaluate model performance using Precision, Recall, and F1-Score.
7. Compare Baseline vs. Proposed dataset strategies (Ablation Study).
8. Enforce a 200-frame temporal gap for unbiased qualitative visual testing.
9. Identify the model's capability to track multiple objects in heavy snow/fog clutter.

## Deep Learning Model

### Faster R-CNN (ResNet-50 FPN Backbone)
Faster R-CNN is a two-stage object detection architecture. It is chosen for this project because it provides:
- **High accuracy** for bounding box localization.
- A **Region Proposal Network (RPN)** that efficiently scans radar clutter.
- A **Feature Pyramid Network (FPN)** that handles multi-scale features, allowing the model to detect small pedestrians and large buses simultaneously.

## Evaluation Metrics

The models are evaluated using the following classification/detection metrics at an IoU (Intersection over Union) threshold of 0.50.

### Precision
Measures the percentage of correct predictions out of all predictions made by the model. Higher precision means the model makes fewer false alarms (predicting a car where there is none).

### Recall
Measures the percentage of actual objects the model successfully detected. In autonomous driving, high recall is safety-critical so that no vulnerable road user is missed.

### F1-Score / mAP Estimate
The harmonic mean of Precision and Recall. It provides a single benchmark score for overall model health across all classes.

## Analysis & Results

In addition to standard training, the project includes deep analytical evaluations:

### Ablation Study
The baseline imbalanced model achieved an F1-Score of 80.07%. By introducing the proposed Target-Aware Oversampling, the F1-Score jumped to **90.55%** (an absolute improvement of +10.48%). 

<p align="center">
  <img src="table1_ablation_fixed.png" alt="Ablation Study" width="75%">
</p>

### Class-Wise Performance
The class-wise breakdown proves that the oversampling strategy successfully removed the model's bias. Rare and vulnerable classes showed massive safety improvements:
- **Pedestrians:** 95.10% Recall
- **Buses:** 95.80% Recall

<p align="center">
  <img src="table2_classwise_fixed.png" alt="Class Wise Results" width="75%">
</p>

### Qualitative Visualization (200-Frame Gap)
To ensure the model didn't just overfit to consecutive frames, a Smart Ranking Algorithm enforced a strict minimum gap of 200 frames between test scenes. Ground Truth boxes are displayed in Green, and Model Predictions in Red.

*Fog Sequence Evaluation:*
<p align="center">
  <img src="fig2_fog_visualizations_rich.png" alt="Fog Visualizations" width="100%">
</p>

*Snow Sequence Evaluation:*
<p align="center">
  <img src="fig3_snow_visualizations_gap_enforced.png" alt="Snow Visualizations" width="100%">
</p>

## Project Workflow

```text
RADIATE Radar Dataset (Fog & Snow)
        |
        v
Data Loading & JSON Parsing
        |
        v
Class Distribution Analysis
        |
        v
Target-Aware Minority Oversampling
(1,714 -> 2,722 frames)
        |
        v
Train / Validation Split (80% / 20%)
        |
        +-------------------+
        |                   |
        v                   v
  PyTorch Tensors     Bounding Boxes
        |                   |
        +---------+---------+
                  |
                  v
             Faster R-CNN
       (ResNet-50 FPN Backbone)
                  |
                  v
          Model Evaluation
                  |
        +---------+---------+
        |         |         |
        v         v         v
    Precision   Recall   F1-Score
        |
        v
Smart Frame Ranking (>200 Frame Gap)
        |
        v
  Qualitative Visualizations
        |
        v
Robust Autonomous Perception
