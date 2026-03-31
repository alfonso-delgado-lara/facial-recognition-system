# Facial Recognizer in R: PCA, Fisher Discriminant Analysis and kNN with Rejection

## Overview
This project develops a facial recognition system in **R** from raw image data. Each image is converted into a high-dimensional RGB feature vector, and different recognition pipelines are implemented and compared:

1. **PCA + kNN**
2. **Raw pixel-space kNN (without dimensionality reduction)**
3. **PCA + Fisher Discriminant Analysis + kNN with rejection**

The work goes beyond simply applying libraries: key parts of the pipeline are implemented manually, including dimensionality reduction logic, custom kNN classification, distance normalization, rejection rules, and model selection through cross-validation.

---

## Problem
The objective is to recognize identities from face images while also allowing the system to **reject unknown faces** when the input does not confidently match any trained identity.

This turns the task into more than a standard closed-set classification problem: the system must both **classify known individuals** and **avoid forced misclassification** when a face is too far from the training distribution.

---

## Dataset and Data Representation
The training set contains **150 images** corresponding to **25 different people**, with **6 images per person**. Each image is loaded with the `magick` package, converted to **RGB**, and normalized to the **[0,1]** range. After flattening the three channels, each image becomes a single feature vector, producing a dataset of size **150 × 108,000**. :contentReference[oaicite:2]{index=2}

This extremely high-dimensional setting is one of the core challenges of the project and motivates the use of dimensionality reduction.

---

## Methodology

### 1. Image preprocessing
- Images are loaded from disk and converted to RGB.
- Pixel intensities are extracted channel by channel.
- The R, G and B channels are flattened and concatenated.
- Values are normalized by dividing by 255.

### 2. PCA-based recognizer
A custom PCA function is used to reduce dimensionality before classification. This helps handle the huge number of variables and makes downstream computation feasible. A kNN classifier is then built on the retained principal components. The number of components is selected through a **variance-explained ratio**.

### 3. kNN classifier implemented with multiple metrics
The recognition stage is not treated as a black box. The project explicitly implements a kNN classifier that supports:
- **Euclidean distance**
- **Manhattan distance**
- **Cosine distance**

Distances are computed directly in the projected space, and predictions are obtained through neighbor voting.

### 4. Rejection mechanism
A particularly strong part of the project is the addition of an **open-set rejection rule**. Instead of always assigning a known identity, distances are first normalized into the **[0,1]** range using empirical minimum and maximum values from the training set. If the closest normalized distance exceeds a threshold, the face is rejected as **unknown**. This same logic is used in the Fisher-based model as well. :contentReference[oaicite:3]{index=3}

### 5. Raw-pixel recognizer without PCA
To make the comparison meaningful, the project also rebuilds the full recognition pipeline directly in raw pixel space. This is not just PCA with all components retained: the functions are redefined because applying PCA, even with all eigenvectors, changes the geometry of the space for non-Euclidean metrics. This is a very nice methodological detail.

### 6. Fisher Discriminant Analysis under small-sample conditions
The most original part of the project is the **PCA + FDA** pipeline. Because the number of features is far larger than the number of observations, the classical within-class scatter matrix is singular. To solve this, the project first applies PCA and keeps **N − C = 150 − 25 = 125** components, then performs Fisher Discriminant Analysis in that reduced space. The final projection is built by chaining both transformations into a single optimal projection matrix. :contentReference[oaicite:4]{index=4}

This is not a trivial implementation: it explicitly addresses the **small sample size problem**, regularizes the within-class scatter matrix for numerical stability, and derives a discriminant representation tailored to identity separation.

---

## Hyperparameter optimization
The Fisher-kNN system is tuned through **grid search with 5-fold cross-validation**, exploring:
- variance ratio
- number of neighbors `k`
- distance metric

During this phase, the rejection threshold is temporarily disabled to focus on pure classification performance. Once the best configuration is found, threshold calibration is handled separately. :contentReference[oaicite:5]{index=5}

---

## Threshold calibration
A dedicated threshold analysis compares:
- **intra-class distances** (same person)
- **inter-class distances** (different people)

Distances are normalized and their density distributions are studied to select a threshold that creates a sensible safety margin between genuine and impostor matches. This is one of the most practically relevant parts of the project, because it transforms the classifier into a more realistic recognition system rather than a forced closed-set predictor. :contentReference[oaicite:6]{index=6}


The project compares the three approaches on **external images** outside the training subset. An especially interesting conclusion is that the **PCA-based model performs best overall**, followed by the raw recognizer, while the **PCA + FDA model underperforms in this specific setting**, likely because FDA is supervised and more sensitive to noise when the dataset is small. That is a thoughtful and honest conclusion, and it shows good statistical judgment rather than just reporting a more complex model as “better”. 

---

## Tech stack
- **R**
- `magick`
- `gtools`
- `ggplot2`
- `knitr`
- `kableExtra`

---

## Authors:
- Alfonso Delgado Lara
- Diego Rivera Suárez

This project received a calification of 9.2/10 at the Statistical Learning subject for the MSc in Big Data at Universidad Carlos III de Madrid.
