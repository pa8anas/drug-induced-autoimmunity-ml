# Drug-Induced Autoimmunity Prediction

Machine learning and cheminformatics framework for predicting **Drug-Induced Autoimmunity (DIA)** from molecular descriptors and molecular structure.

## Overview

Drug-induced autoimmunity is a challenging drug-safety problem because positive cases are relatively rare and standard accuracy can be misleading. This project develops and compares machine learning pipelines that combine classical molecular descriptors with graph-based and fingerprint-based representations, while explicitly addressing class imbalance and probability reliability.

The main workflow includes:

- RDKit molecular descriptor preprocessing
- Morgan / ECFP fingerprints
- Tanimoto-similarity graph construction
- Graph smoothing and Laplacian Eigenmaps
- Feature fusion
- SMOTE class balancing
- Murcko scaffold-aware cross-validation
- Hyperparameter optimization with `RandomizedSearchCV`
- Isotonic probability calibration
- Adaptive F-beta threshold optimization
- Dual-threshold classification with abstention
- Multi-seed probability averaging

## Dataset

The project uses the **Drug Induced Autoimmunity Prediction** dataset from the UCI Machine Learning Repository.

- Total molecules: **597**
- RDKit molecular descriptors: **195**
- Training set: **477** molecules
  - 118 DIA-positive
  - 359 DIA-negative
- Test set: **120** molecules
  - 30 DIA-positive
  - 90 DIA-negative
- Positive-class prevalence: approximately **25%**

Each sample contains:

- `SMILES` — molecular structure representation
- RDKit descriptors — physicochemical, topological and structural features
- `Label` — binary target (`1` = DIA-positive, `0` = DIA-negative)

Dataset source: https://archive.ics.uci.edu/dataset/1104/drug_induced_autoimmunity_prediction

## Feature Engineering

### Molecular descriptors

The preprocessing pipeline removes:

- zero-variance features
- duplicate descriptor columns
- highly correlated descriptors (`|r| > 0.95`)

### ECFP fingerprints

Morgan fingerprints are generated with RDKit and used both for graph construction and as additional model features. Informative fingerprint bits are selected with chi-square feature selection.

### Tanimoto similarity graph

A molecular similarity graph is constructed from ECFP fingerprints using Tanimoto/Jaccard similarity and k-nearest neighbours.

The graph representation supports:

- weighted edges
- optional mutual-kNN filtering
- self-loops
- normalized adjacency
- graph smoothing of RDKit descriptors
- Laplacian Eigenmaps spectral embeddings

The final feature representation can concatenate:

1. original RDKit descriptors
2. graph-smoothed descriptors
3. spectral graph embeddings
4. selected ECFP bits

## Models

Three primary classifiers are evaluated:

- **XGBoost**
- **Random Forest**
- **Logistic Regression**

The notebook also supports experimentation with alternative models and hyperparameter settings.

## Class Imbalance and Validation

Because the dataset is imbalanced, the project uses **SMOTE** to oversample the minority class.

Validation is performed using **Murcko scaffold-aware cross-validation** when scaffold groups are available. This reduces overly optimistic validation results caused by structurally similar molecules appearing in both training and validation folds.

## Calibration and Decision Strategy

Raw classifier probabilities are calibrated with **Isotonic Regression** using out-of-fold predictions.

Instead of relying on a fixed threshold of `0.5`, the decision threshold is optimized from the precision-recall curve using an **F-beta score** under a minimum-precision constraint.

An optional **dual-threshold** strategy defines:

- confident negative predictions
- confident positive predictions
- an uncertainty / abstention region

For stochastic estimators, multiple random seeds are trained and their predicted probabilities are averaged.

## Results

Seed-averaged test-set results reported in the accompanying study are:

| Model | Accuracy | Precision | Recall | F1-score | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| XGBoost | 0.8917 | 0.8696 | 0.6667 | 0.7547 | 0.9274 |
| Random Forest | **0.8917** | 0.8148 | **0.7333** | **0.7719** | **0.9326** |
| Logistic Regression | 0.8583 | **0.8824** | 0.5000 | 0.6383 | 0.8770 |

The **Random Forest** achieved the strongest overall balance between precision and recall, while **XGBoost** produced higher precision and **Logistic Regression** served as an interpretable baseline.

## Repository Contents

```text
.
├── DIA_trainingset_RDKit_descriptors.csv
├── DIA_testset_RDKit_descriptors.csv
├── Drug_Induced_Autoimmunity_Prediction.ipynb
├── RDKit_ChemDes.xlsx
├── report.pdf
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

## Installation

Clone the repository and install the dependencies:

```bash
git clone https://github.com/pa8anas/drug-induced-autoimmunity-ml.git
cd drug-induced-autoimmunity-ml
pip install -r requirements.txt
```

The notebook was designed to run conveniently in **Google Colab**. GPU acceleration is optional and mainly benefits XGBoost-based experiments.

## Running the Project

Open:

```text
Drug_Induced_Autoimmunity_Prediction.ipynb
```

and run the notebook cells in order. If using Google Colab, upload or mount the dataset files in the expected location and update dataset paths if necessary.

## Evaluation Metrics

The project reports:

- Accuracy
- Precision
- Recall
- F1-score
- ROC-AUC
- Average Precision / PR-AUC
- Confusion matrices
- ROC curves
- Precision-Recall curves

For an imbalanced drug-safety problem, particular attention is given to **Recall, F1-score and Average Precision**, rather than accuracy alone.

## Technologies

- Python
- pandas
- NumPy
- scikit-learn
- imbalanced-learn
- XGBoost
- RDKit
- SciPy
- Matplotlib
- Seaborn
- Google Colab

## Report

The full academic report is included as `report.pdf` and documents the dataset, exploratory analysis, methodology, model comparison and conclusions.

## License

This repository is released under the license included in the `LICENSE` file.

## Authors

Project developed as part of the MSc coursework in **Big Data & Analytics**, University of Piraeus.