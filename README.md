# 🏥 Healthcare Analytics — DSO545 Final Project

> Applying statistical and machine learning techniques to synthetic EHR data for patient outcome analysis and risk stratification.

---

## 📌 Project Overview

This project analyzes a **synthetic Electronic Health Record (EHR) dataset** covering 100,000 adult patients in the United States (2018–2024). The goal is to extract actionable healthcare and business insights using a combination of statistical and unsupervised machine learning methods.

The dataset was calibrated against real-world sources including CDC NHANES, CMS hospitalization benchmarks, and WHO ICD-10 classifications.

---

## 📂 Repository Structure

```
DSO545-Final-Project/
│
├── data/
│   ├── patients.csv                  # 100,000 patient demographics & diagnostics
│   └── patient_outcome.csv           # 6,600 hospitalization outcome records
│
├── notebooks/
│   └── final_report_Murphy.ipynb     # Full analysis notebook (Google Colab)
│
├── report/
│   └── final_project_Murphy.pdf      # Rendered PDF report
│
├── README.md
└── .gitignore
```

---

## 🔍 Analysis Pipeline

### 1. Data Overview & Screening
- Inspected structure, types, and missingness across both datasets
- Applied IQR-based outlier detection; retained clinically valid edge cases
- Confirmed `days_to_readmission` NaNs are structurally valid (non-readmitted patients)

### 2. Feature Engineering
| Variable | Method | Justification |
|---|---|---|
| `gender` | Label Encoding | Binary, no ordinal meaning |
| `alcohol_use` | Label Encoding | Ordinal: none → heavy |
| `exercise_level` | Label Encoding | Ordinal: sedentary → vigorous |
| `smoking_status` | One-Hot Encoding | Nominal, 3 categories |
| `insurance_type` | One-Hot Encoding | Nominal, 5 categories |
| `discharge_disposition` | One-Hot Encoding | Nominal, 6 categories |
| `admission_month/year` | Extracted from date | Temporal feature engineering |

### 3. A/B Testing

**Test 1 — Insurance Status vs. Total Charges**
- Test: Two-sided Welch's t-test
- Result: **Fail to reject H₀** (p = 0.8343) — no significant difference overall
- Confound: ICU patients show a borderline signal (p = 0.0530)

**Test 2 — Gender vs. 30-Day Readmission Rate**
- Test: Two-sample proportion z-test
- Result: **Fail to reject H₀** (p = 0.5390) — gender does not predict readmission
- Confound: COPD status does not confound the result

### 4. Bootstrap Simulation

| Metric | Observed | 95% CI |
|---|---|---|
| Mean Length of Stay | 5.25 days | (5.14, 5.36) |
| 30-Day Readmission Rate | 14.30% | (13.45%, 15.15%) |

> ⚠️ The upper CI of 15.15% approaches the CMS penalty threshold (~15%), flagging operational risk.

### 5. Principal Component Analysis (PCA)

- Applied to 28 patient features (demographics, vitals, lifestyle, diagnostics)
- **5 components retained** (28.3% variance explained)
- Kaiser's Criterion: 12 components have eigenvalue > 1; 5 chosen for interpretability

| Component | Interpretation |
|---|---|
| PC1 | Chronic Disease Burden (Charlson Index, CKD, Age) |
| PC2 | Age-Related Cardiovascular Risk (Age, Systolic BP, Hypertension) |
| PC3 | Smoking History (Never / Former smokers) |
| PC4 | Active Smoking Behavior (Current / Former smokers) |
| PC5 | Lifestyle & Metabolic Risk (T2D, Alcohol, Exercise, Osteoarthritis) |

### 6. K-Means Clustering

Clustering was performed on the 5 PCA components. Optimal k selected by silhouette score.

| Cluster | Size | Readmission Rate | Avg Charges | Avg LOS | Charlson Index |
|---|---|---|---|---|---|
| **0** — High Cost | 1,659 | 13.68% | $18,592 | 5.31 days | 1.31 |
| **1** — Moderate | 3,676 | 14.25% | $18,134 | 5.21 days | 1.35 |
| **2** — High Risk ⚠️ | 1,265 | **15.26%** | $17,306 | 5.29 days | **1.39** |

> Cluster 2 patients have the highest readmission rate and comorbidity burden — prime candidates for post-discharge intervention programs.

---

## 💡 Key Business Insights

- A **2% readmission reduction** in Cluster 2 alone could prevent ~25 readmissions and save **$430,000+** in index admission costs
- The hospital's readmission rate upper CI (**15.15%**) brushes the CMS penalty threshold — proactive monitoring is advised
- Insurance status alone does not drive cost differences; **ICU admission** is the more critical factor
- Gender is **not** a meaningful predictor of readmission risk in this dataset

---

## 🛠️ Tech Stack

![Python](https://img.shields.io/badge/Python-3.12-blue?logo=python)
![Pandas](https://img.shields.io/badge/Pandas-2.2-lightgrey?logo=pandas)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-1.6-orange?logo=scikit-learn)
![SciPy](https://img.shields.io/badge/SciPy-1.16-blueviolet?logo=scipy)
![Seaborn](https://img.shields.io/badge/Seaborn-latest-teal)
![Google Colab](https://img.shields.io/badge/Google%20Colab-notebook-F9AB00?logo=googlecolab)

**Libraries used:** `numpy`, `pandas`, `matplotlib`, `seaborn`, `scipy`, `statsmodels`, `sklearn`, `factor_analyzer`

---

## ▶️ How to Run

1. Open `notebooks/final_report_Murphy.ipynb` in [Google Colab](https://colab.research.google.com/)
2. Upload `patients.csv` and `patient_outcome.csv` to your Google Drive under the path referenced in the notebook
3. Run all cells in order
4. Random seed is set to `551` for reproducibility

---

## 📋 Course Information

| | |
|---|---|
| **Course** | DSO545 — Spring 2026 |
| **Topics** | A/B Testing · Bootstrap · PCA · Clustering |
| **Dataset** | Synthetic EHR (calibrated against CDC, CMS, WHO ICD-10) |
| **Random Seed** | 551 |

---

*Synthetic data only — no real patient information is used or disclosed.*
