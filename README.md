# Sepsis Clinical Analytics Project V1

A patient-level exploratory data analysis of ICU time-series data to investigate clinical characteristics associated with Sepsis compared with Non-Sepsis patients.

Built using Python, Pandas, Seaborn, and real-world clinical ICU data.

---

# Overview

This project analyzes intensive care unit (ICU) time-series data to explore physiological and laboratory differences between patients with and without sepsis.

The main goal is to build a **clean, interpretable, patient-level clinical analytics pipeline** that transforms complex ICU measurements into clinically meaningful features.

---

# Clinical Question

**Which physiological and laboratory characteristics differ between ICU patients with and without sepsis?**

---

# Objectives

- Identify patients with and without sepsis
- Assess data quality and missingness patterns
- Transform ICU time-series measurements into patient-level features
- Explore clinical characteristics associated with sepsis status
- Visualize relevant physiological and laboratory differences

---

# Dataset

The dataset contains ICU time-series measurements from the **PhysioNet/Computing in Cardiology Challenge 2019**.

Included variables:

- Vital signs:
  - Heart rate (HR)
  - Mean arterial pressure (MAP)
  - Respiratory rate (Resp)
  - Oxygen saturation (O2Sat)
  - Temperature (Temp)

- Laboratory measurements:
  - Lactate
  - White blood cell count (WBC)
  - Creatinine
  - Platelets

- Demographic characteristics:
  - Age
  - Gender

- Sepsis label:
  - Binary outcome indicating whether sepsis occurred during ICU stay

Each row represents one hourly measurement during a patient's ICU stay.

The original dataset is publicly available via Kaggle. To keep this repository lightweight, a reduced sample dataset is included for demonstration and testing purposes. The complete dataset can be downloaded through the provided notebook.

---

# Methodology

## 1. Data Acquisition

- ICU dataset downloaded using `kagglehub`
- Raw time-series data imported from CSV

---

## 2. Data Quality Assessment

- Missing value analysis performed
- Variables with >90% missing values removed
- Clinically relevant variables retained
- Plausibility checks performed using descriptive statistics

---

## 3. Patient-Level Feature Engineering

Hourly ICU measurements were transformed into patient-level features.

Aggregation included:

- Mean values
- Maximum values
- Minimum values

Examples:

- Mean heart rate
- Minimum MAP
- Maximum lactate
- Maximum creatinine

The final outcome variable was defined as:

`SepsisPatient`

where:

- 1 = patient developed sepsis
- 0 = patient did not develop sepsis

---

# 4. Exploratory Data Analysis (EDA)

The analysis followed a structured clinical EDA workflow:

## Cohort Analysis

- Distribution of Sepsis vs Non-Sepsis patients

## Univariate Analysis

- Assessment of clinical variable distributions
- Evaluation of skewness and potential outliers

## Clinical Characteristics by Sepsis Status

- Comparison of physiological and laboratory parameters between groups
- Boxplots comparing:
  - Heart rate
  - Mean arterial pressure
  - Lactate
  - Creatinine
  - Platelets

## Clinical Summary

- Group-level comparison of key clinical parameters

---

# 5. Statistical Analysis

Group differences were assessed using hypothesis testing.

### Welch's Two-Sample t-test

Continuous variables with approximately symmetric distributions were compared using Welch's two-sample t-test, which does not assume equal variances between groups.

Variables analyzed:

- Heart rate (HR)
- Mean arterial pressure (MAP)

### Mann–Whitney U test

Laboratory variables with clearly skewed distributions were additionally analyzed using the non-parametric Mann–Whitney U test.

Variables analyzed:

- Maximum lactate
- Maximum creatinine
- Mean platelet count

### Statistical Significance

A significance level of **α = 0.05** was used.

Most clinical variables showed statistically significant differences between Sepsis and Non-Sepsis patients. Platelet counts were not significant using Welch's t-test but reached significance using the Mann–Whitney U test, suggesting differences in the underlying distributions rather than in the group means.

### Continuous Variables

| Feature | Test | Statistic | p-value | Interpretation |
|---|---|---:|---:|---|
| HR_mean | Welch t-test | 18.58 | <0.001 | Significant difference: septic patients showed higher mean heart rates |
| MAP_mean | Welch t-test | -9.49 | <0.001 | Significant difference: septic patients showed lower mean arterial pressure |
| Lactate_max | Welch t-test | 4.88 | <0.001 | Significant difference: septic patients had higher maximum lactate levels |
| Creatinine_max | Welch t-test | 9.73 | <0.001 | Significant difference: septic patients had higher maximum creatinine levels |
| Platelets_mean | Welch t-test | -1.70 | 0.088 | No statistically significant difference detected |
| Age_first | Welch t-test | 1.42 | 0.157 | No statistically significant difference detected |

---

### Non-normally Distributed Variables

| Feature | Test | Statistic | p-value | Interpretation |
|---|---|---:|---:|---|
| Lactate_max | Mann–Whitney U | 10,451,437.50 | <0.001 | Significant difference confirmed: septic patients showed higher lactate values |
| Creatinine_max | Mann–Whitney U | 58,380,432.50 | <0.001 | Significant difference confirmed: septic patients showed higher creatinine values |
| Platelets_mean | Mann–Whitney U | 44,743,645.00 | <0.001 | Significant difference detected despite Welch test result |

---

---

# Clinical Interpretation

The observed differences reflect established clinical concepts:

- Increased heart rate may represent compensatory cardiovascular stress during systemic inflammation.
- Reduced MAP can occur due to vasodilation and impaired vascular tone in severe infection.
- Elevated lactate may indicate impaired tissue oxygen utilization and hypoperfusion.
- Increased creatinine may reflect impaired renal function associated with critical illness.

---

# Key Findings

Patients with sepsis showed different clinical profiles compared with non-sepsis patients.

Observed patterns included:

- Higher heart rate values
- Lower mean arterial pressure values
- Increased lactate levels
- Higher creatinine levels

These findings are consistent with known physiological changes associated with systemic infection, impaired perfusion, and organ dysfunction.

---

# Limitations

- Temporal information was simplified through patient-level aggregation.
- Aggregation may hide important changes occurring before or after sepsis onset.
- Missing data handling was simplified and may introduce bias.
- This project focuses on exploratory analysis only; no predictive model was developed in V1.

---

# Clinical Relevance

This project demonstrates how patient-level aggregation of ICU time-series data can reveal clinically meaningful patterns.

The analysis provides an interpretable foundation for future work including predictive modeling, temporal feature engineering, and clinical risk assessment.

---

# Visualizations

Included visualizations:

- Patient cohort distribution
- Clinical variable distributions
- Boxplots comparing Sepsis vs Non-Sepsis patients
- Clinical summary heatmap

---

# Tools Used

- Python
- Pandas
- NumPy
- Seaborn
- Matplotlib
- KaggleHub

---

# Future Work (V2)

Potential extensions:

- Time-to-sepsis prediction
- Machine learning classification models
- Temporal feature engineering
- Early warning risk scoring
- Model explainability (e.g., feature importance, SHAP)
