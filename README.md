# Predicting Time to Treatment for Metastatic Breast Cancer WiDS Datathon++ 2024 University Challenge

 **2nd place** WiDS Datathon++ 2024 University Challenge (Kaggle)

## Problem Overview

This project tackles the [WiDS Datathon++ 2024 University Challenge](https://www.kaggle.com/competitions/widsdatathon2024-university/overview), which asks participants to predict how many days it takes a patient diagnosed with metastatic triple-negative breast cancer (TNBC) to receive their first treatment. The target, `treatment_pd`, is the gap between metastatic diagnosis and first treatment which a proxy for disparities in healthcare access. TNBC is among the most aggressive forms of breast cancer, making timely treatment especially critical, and the dataset (sourced from Health Verity and enriched with zip-code-level socioeconomic data) allows for exploring how demographic, diagnostic, and treatment related factors relate to that delay.

## Data

The dataset is provided by the competition sponsor (Gilead Sciences) via Kaggle: [competition data page](https://www.kaggle.com/competitions/widsdatathon2024-university/data). It includes ~39K patient records with demographic attributes (age, race, insurance type, zip-code-level socioeconomic indicators), diagnostic details (ICD10/ICD9 codes and descriptions), and treatment information. Raw data files are not included in this repo, kindly see the competition page to access them directly.

## Exploratory Data Analysis

Before modeling, we explored the dataset across three feature groups **demographic** (age, race, geography, insurance type), **diagnostic** (cancer codes, diagnosis descriptions), and **treatment** (first treatment type and timing). Key findings, visualized in an accompanying dashboard (`dashboard.png`):

- The median treatment delay dropped sharply for patients diagnosed in 2016 onward compared to 2015, then declined gradually.
- Insurance/payer type showed a clear relationship with speed of care. Medicare Advantage patients had the longest median delays, Commercial the shortest.
- Older patients (60+) faced longer delays than younger age groups, with delay increasing roughly monotonically by age bracket.
- Geographic division showed some variation in median treatment delay, though less pronounced than the insurance or age effects.
- First treatment type was one of the strongest differentiators e.g., Nonsteroidal Anti-Inflammatory and Antimycobacterial first treatments were associated with much longer delays than Antineoplastics.

*A companion Power BI dashboard exploring these patterns in depth is available at [WiDS_2024_Breast_Cancer_Dashboard](https://github.com/Aditirk8/WiDS_2024_Breast_Cancer_Dashboard).*

## Variables Used in the Model

**Target**

| Variable | Description |
|---|---|
| `treatment_pd` | Difference (in days) between metastatic first treatment date and breast cancer diagnosis date |

**Predictors**

| Variable | Description |
|---|---|
| `breast_cancer_diagnosis_code` | ICD10 or ICD9 diagnosis code for the breast cancer diagnosis |
| `breast_cancer_diagnosis_desc` | Text description corresponding to the ICD10/ICD9 diagnosis code source for the CountVectorizer + PCA features |
| `breast_cancer_diagnosis_year` | Calendar year of first breast cancer diagnosis |
| `metastatic_cancer_diagnosis_code` | ICD10 diagnosis code for the metastatic cancer |
| `metastatic_first_treatment` | Generic drug name of the first treatment administered after metastatic diagnosis |

**Derived features**

| Feature | Description |
|---|---|
| `PC1`–`PC4` | Four principal components summarizing a bag-of-words representation of `breast_cancer_diagnosis_desc`, used to capture textual diagnosis detail without the dimensionality of raw CountVectorizer output |

All three categorical fields (`breast_cancer_diagnosis_code`, `metastatic_cancer_diagnosis_code`, `metastatic_first_treatment`) were passed natively to CatBoost, with missing values imputed as `'missing'` rather than encoded manually.

Demographic features (age, race, insurance type, geography, etc.) were explored during EDA but **not included in the final Approach 1 model** this repo covers the diagnostic/treatment-based approach that placed 2nd; a second approach incorporating demographics further was also developed but isn't covered here.

## Approach & Modeling

**Data Preparation**

Categorical features (`breast_cancer_diagnosis_code`, `metastatic_cancer_diagnosis_code`, `metastatic_first_treatment`) were passed directly into CatBoost rather than manually encoded, taking advantage of CatBoost's native handling of high-cardinality categoricals. Missing values in these fields were imputed with the placeholder `'missing'` rather than dropped, preserving row count and letting the model treat "missing" as its own informative category.

**Feature Engineering**

To capture the textual detail in `breast_cancer_diagnosis_desc` beyond its raw diagnosis code, we applied `CountVectorizer` to generate a bag-of-words representation. Since this produced a large, sparse feature space, we standardized it and reduced it to 4 principal components (`PC1`–`PC4`) via PCA enough to retain the signal in diagnosis descriptions without letting text features dominate the model or introduce excessive dimensionality relative to the dataset size.

**Feature Selection**

We first fit an initial CatBoost model using all available variables. Using this model's built-in feature importance output, we then identified the most influential features and used them to build the final, reduced feature set described above rather than selecting fields purely on domain intuition.

**Model Training**

The final feature set (diagnostic codes, treatment type, diagnosis year, and PCA components) was split 80/20 into train and test sets. A `CatBoostRegressor` was trained on the training pool, with the categorical columns explicitly flagged via `cat_features`, and evaluated against the held-out test pool using RMSE.

CatBoost was chosen specifically for this approach because it removes the need for manual target/one-hot encoding on fields like `metastatic_first_treatment` and diagnosis codes, which have high cardinality a property we'd flagged during EDA.

## Results

The final CatBoostRegressor achieved an RMSE of **138.45** on our internal 80/20 holdout split.

On Kaggle's private leaderboard, the model scored an RMSE of **140.16**, placing **2nd overall** in the WiDS Datathon++ 2024 University Challenge.

## Related

- [WiDS_2024_Breast_Cancer_Dashboard](https://github.com/Aditirk8/WiDS_2024_Breast_Cancer_Dashboard) is a companion Power BI dashboard exploring socioeconomic, geographic, and clinical factors behind treatment delays for this same challenge.

## Acknowledgements

Thank you to WiDS Worldwide and Gilead Sciences for organizing the WiDS Datathon++ 2024 University Challenge and providing the dataset. This project was a collaborative effort with my teammate Vadiraj Moktali.
