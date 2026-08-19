# Breast Cancer ER Status: Gene Expression Analysis & Machine Learning

## Overview

This project explores gene expression patterns associated with estrogen receptor (ER) status in breast cancer and evaluates whether gene expression profiles can be used to predict ER-positive and ER-negative samples.

The analysis combines exploratory data analysis, statistical testing and supervised machine learning in a two-stage workflow:

1. **Exploratory and differential expression analysis**
2. **Leakage-controlled machine learning and model interpretation**

The main objective is not only to build a predictive model, but to connect statistical analysis with machine learning and biological interpretation.

---

## Research Question

**Can gene expression profiles distinguish ER-positive from ER-negative breast cancer samples, and which genes contribute most strongly to this classification?**

---

## Dataset

The project uses breast cancer gene expression and clinical data.

After matching the expression and clinical datasets, **519 samples** with available ER-status information were retained.

The expression dataset contained:

- **519 samples**
- **17,268 gene features**
- **401 ER-positive samples**
- **118 ER-negative samples**

ER status was encoded as a binary target:

- `1` → ER-positive
- `0` → ER-negative

---

# 01 — Exploratory Data Analysis

The first notebook investigates the global structure of the gene expression data and identifies genes associated with ER status.

## Principal Component Analysis

PCA was used to reduce the dimensionality of the expression data and visualize the main sources of variation.

The first principal components showed a clear separation between ER-positive and ER-negative samples, indicating that ER status is associated with a substantial transcriptional signal in the dataset.

The genes contributing most strongly to the separation were then examined to identify biologically meaningful expression patterns.

## Differential Expression Analysis

Gene expression was compared between ER-positive and ER-negative samples using the **Mann–Whitney U test**.

Because thousands of genes were tested simultaneously, p-values were corrected using the **Benjamini–Hochberg procedure** to control the false discovery rate.

The criteria used for downstream gene analysis were the following:

- `FDR < 0.05`
- `|mean expression difference| ≥ 1`

This identified **950 candidate genes** in the exploratory analysis. However, this process was repeated with the training subset in order to select the genes used for machine learning.

## Volcano Plot

The volcano plot provides a global visualization of the relationship between statistical significance and effect size.

Several genes showed particularly strong differences between ER-positive and ER-negative samples, including genes such as **ESR1, GATA3, AGR3 and CA12**.

---

# 02 — Machine Learning

The second notebook evaluates whether the expression patterns identified during exploration can be used to predict ER status in previously unseen samples.

## Train-Test Split

The 519 matched samples were divided into:

- **415 training samples**
- **104 test samples**

The split was stratified to preserve the ER-status distribution.

Importantly, the test set was kept completely independent until final model evaluation.

## Feature Selection

To avoid data leakage, feature selection was repeated **exclusively within the training set** rather than directly reusing the 950 genes identified in the exploratory analysis.

Genes were selected using:

- Mann–Whitney U test
- Benjamini–Hochberg FDR correction
- `FDR < 0.05`
- `|mean expression difference| ≥ 1`

This resulted in **1,009 selected genes** for machine learning.

## Preprocessing

Missing values were imputed using gene-specific medians learned from the training set.

For Logistic Regression, gene expression was subsequently standardized using parameters learned exclusively from the training data.

The same learned parameters were then applied to the test set, preventing information leakage.

---

# Model Performance

Two complementary classification models were evaluated:

- **Logistic Regression**
- **Random Forest**

Both models were evaluated on the same independent test set.

| Model | Accuracy | Precision | Recall | F1-score | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| **Logistic Regression** | **0.923** | **0.962** | **0.938** | **0.949** | **0.900** |
| Random Forest | 0.894 | 0.926 | 0.938 | 0.932 | 0.885 |

## Best-performing Model

**Logistic Regression achieved the best overall performance**, with:

- **92.3% accuracy**
- **0.949 F1-score**
- **0.900 ROC-AUC**

The Random Forest model achieved a ROC-AUC of **0.885**.

The results suggest that a relatively simple linear model was sufficient to capture much of the predictive signal present in the selected gene expression features.

---

# Model Interpretation

The Logistic Regression coefficients were examined to identify genes contributing most strongly to ER-status prediction.

Positive coefficients indicate genes that push the prediction towards ER-positive status, while negative coefficients push the prediction towards ER-negative status.

The genes with the strongest coefficients included:

### ER-positive direction

- **SBSN**
- **FBXL16**
- **AMDHD1**
- **NFE2L3**
- **KCTD6**
- **TFF1**
- **PHGDH**

### ER-negative direction

- **ZIC1**
- **GALNT13**
- **TMEM40**
- **PAX6**
- **MAP2**
- **VLDLR**
- **PGR**
- **SOX11**

The genes identified by the predictive model do not necessarily correspond to those showing the largest individual expression differences.

This is expected because differential expression and Logistic Regression answer different questions:

- **Differential expression** identifies genes with strong individual differences between ER-positive and ER-negative samples.
- **Logistic Regression** evaluates the contribution of genes jointly within a multivariate predictive model.

Because gene expression features can be correlated, a gene with a large individual difference does not necessarily provide the greatest additional predictive value once other genes are included.

Among the 40 genes with the strongest Logistic Regression coefficients, **PGR and SOX11** were also among the 40 genes with the largest absolute expression differences.

---

# Key Findings

## 1. ER status is associated with a strong transcriptional signal

PCA revealed clear separation between ER-positive and ER-negative samples, indicating substantial differences in their global gene expression profiles.

## 2. Differential expression identified numerous ER-associated genes

Statistical testing identified a large number of genes with significant and substantial expression differences between the two groups.

## 3. Gene expression can predict ER status

Using only features selected from the training data, Logistic Regression achieved a **ROC-AUC of 0.900** on the independent test set.

## 4. Model complexity did not improve performance

Random Forest performed slightly worse than Logistic Regression, suggesting that the main predictive signal could be captured effectively by a relatively simple linear model.

## 5. Statistical and predictive analyses provide complementary information

The genes with the largest individual expression differences were not necessarily the genes with the strongest contribution to the multivariate predictive model.

---

# Limitations

This project is intended as a computational analysis and proof of concept rather than a clinical prediction model.

Important limitations include:

- The analysis uses a single dataset and an independent external validation cohort was not available.
- The sample size is relatively small compared with the number of gene features.
- Feature selection was performed using the available training set and may therefore be dataset-specific.
- Model performance should be validated on independent datasets before considering clinical applications.
- Logistic Regression coefficients indicate predictive contribution within this model and should not be interpreted as evidence of biological causality.

---

# Technologies

The analysis was performed in Python using:

- **Python**
- **NumPy**
- **pandas**
- **SciPy**
- **statsmodels**
- **scikit-learn**
- **Matplotlib**
- **Seaborn**
- **Jupyter Notebook**

---

# Conclusions

This project demonstrates a complete workflow for high-dimensional biological data analysis, from exploratory analysis and statistical testing to supervised machine learning and model interpretation.

The combination of PCA, differential expression analysis and predictive modelling showed that ER-positive and ER-negative breast cancer samples exhibit distinct gene expression patterns that can be used for accurate classification.

The final results also illustrate an important distinction between **statistical association and predictive importance**, highlighting why combining classical statistical analysis with machine learning can provide a more comprehensive view of high-dimensional biological datasets.
