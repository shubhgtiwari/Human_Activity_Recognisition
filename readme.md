#  Human Activity Recognition Using Smartphone Sensors

A comprehensive classification study comparing **8 machine learning models** on the UCI HAR dataset. Built in **R** using the `caret` framework with rigorous preprocessing and cross-validation.

## Dataset

- **Source:** [UCI Human Activity Recognition Using Smartphones](https://archive.ics.uci.edu/ml/datasets/human+activity+recognition+using+smartphones)
- **Sensors:** Accelerometer + Gyroscope (Samsung Galaxy S II)
- **Activities:** 6 classes (Walking, Walking Upstairs, Walking Downstairs, Sitting, Standing, Laying)
- **Original Features:** 561 time and frequency domain variables

## Preprocessing Pipeline

| Step | Features |
|------|----------|
| Raw | 561 |
| After deduplication | 478 |
| Box-Cox Transformation | 478 |
| Spatial Sign Transformation | 478 |
| Correlation Filtering (cutoff=0.9) | **185** |
| PCA | Components used for modeling |

## Models Benchmarked

All models trained with **3×3 repeated cross-validation** on the **Kappa** metric:

| # | Model | Method |
|---|-------|--------|
| 1 | Linear Discriminant Analysis | `lda` |
| 2 | Partial Least Squares DA | `pls` |
| 3 | Penalized Logistic (Elastic Net) | `glmnet` |
| 4 | Neural Network | `nnet` |
| 5 | Flexible Discriminant Analysis | `fda` |
| 6 | Support Vector Machine (RBF) | `svmRadial` |
| 7 | K-Nearest Neighbors | `knn` |
| 8 | Naive Bayes | `nb` |

### Evaluation
- Confusion matrices for all models
- Kappa scores (Train + Test)
- Variable importance analysis (Top 20 features)
- Stratified 80/20 train-test split

## Tech Stack

`R` · `caret` · `dplyr` · `earth` · `corrplot` · `glmnet` · `nnet` · `e1071`

## Usage

```r
# Ensure the UCI HAR dataset is in the working directory
source("rcode.r")
```

## 📄 License

Apache 2.0
