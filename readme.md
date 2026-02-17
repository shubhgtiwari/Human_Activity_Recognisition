# Human Activity Recognition Using Smartphone Sensors

## Overview

A comprehensive machine learning benchmark study that trains and compares 8 classification models on the UCI Human Activity Recognition dataset. The entire pipeline — from data loading and preprocessing through model training and evaluation — is implemented in R using the `caret` framework.

## Dataset

- **Source:** [UCI Human Activity Recognition Using Smartphones](https://archive.ics.uci.edu/ml/datasets/human+activity+recognition+using+smartphones)
- **Collection:** Samsung Galaxy S II worn on the waist by 30 volunteers
- **Sensors:** Tri-axial accelerometer and gyroscope at 50 Hz
- **Activities:** 6 classes — Walking, Walking Upstairs, Walking Downstairs, Sitting, Standing, Laying
- **Original Features:** 561 time-domain and frequency-domain variables

## Preprocessing Pipeline

The preprocessing applies a sequence of transformations to reduce dimensionality and normalize the feature space:

| Step | Description | Features |
|------|-------------|----------|
| 1. Load and merge | Combine train and test sets with labels | 561 |
| 2. Deduplicate columns | Remove duplicate feature names | 478 |
| 3. Box-Cox transform | Stabilize variance (shift values to positive range first) | 478 |
| 4. Spatial Sign transform | Project features onto a unit hypersphere | 478 |
| 5. Remove `Subject` column | Not a significant predictor | 477 |
| 6. Correlation filtering | Remove features with pairwise correlation > 0.9 | **185** |
| 7. PCA | Principal Component Analysis on reduced feature set | Components |

This reduces the feature space by 67% (561 to 185) while preserving the most informative features.

## Models

All 8 models are trained with **3x3 repeated cross-validation** using the **Kappa** metric on a stratified 80/20 train-test split:

| Model | R Method | Description |
|-------|----------|-------------|
| Linear Discriminant Analysis | `lda` | Linear boundary classifier |
| Partial Least Squares DA | `pls` | Dimensionality-reducing discriminant analysis |
| Penalized Logistic (Elastic Net) | `glmnet` | L1/L2 regularization with alpha grid (0, 0.2, 0.4, 0.6, 0.8, 1.0) and lambda range (0.01-0.1) |
| Neural Network | `nnet` | Single hidden layer, tuneLength=5 |
| Flexible Discriminant Analysis | `fda` | Non-linear extension of LDA, tuneLength=10 |
| Support Vector Machine (RBF) | `svmRadial` | Radial Basis Function kernel, tuneLength=10 |
| K-Nearest Neighbors | `knn` | Distance-based classifier, tuneLength=10 |
| Naive Bayes | `nb` | Probabilistic classifier |

## Evaluation

- Confusion matrices generated for all 8 models
- Train and test Kappa scores compared in summary table
- Variable importance analysis (Top 20 features from glmnet and Neural Network models)
- All models evaluated on the same held-out test set for fair comparison

## Tech Stack

- **Language:** R
- **ML Framework:** caret (Classification and Regression Training)
- **Libraries:** dplyr (data manipulation), corrplot (correlation visualization), earth (FDA), glmnet (elastic net), nnet (neural networks), e1071 (SVM, Naive Bayes)

## Project Structure

```
Human_Activity_Recognisition/
├── rcode.r                    # Complete analysis pipeline (349 lines)
├── UCI HAR Dataset/           # Raw data directory
│   ├── train/                 # Training data (X_train.txt, y_train.txt, subject_train.txt)
│   ├── test/                  # Test data (X_test.txt, y_test.txt, subject_test.txt)
│   ├── features.txt           # 561 feature names
│   └── activity_labels.txt    # 6 activity class labels
└── README.md
```

## Getting Started

```r
# Set working directory to project root
# Ensure UCI HAR Dataset folder is present

source("rcode.r")

# The script will:
# 1. Load and preprocess the data
# 2. Train all 8 models with cross-validation
# 3. Print confusion matrices and Kappa scores
# 4. Display variable importance plots
```

## License

Apache 2.0
