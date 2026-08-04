# Sepsis Clinical Analytics Project V1

A patient-level clinical data analysis of ICU time-series data investigating clinical characteristics associated with sepsis and their ability to discriminate between patients with and without sepsis.

Built using Python, Pandas, Scikit-learn, Seaborn, Matplotlib, and real-world ICU data.

---

# Overview

This project analyzes intensive care unit (ICU) time-series data from the **PhysioNet/Computing in Cardiology Challenge 2019**.

The analysis transforms longitudinal ICU measurements into interpretable patient-level clinical features and combines:

- Exploratory data analysis
- Missing-data assessment
- Clinical feature engineering
- Statistical hypothesis testing
- Effect-size analysis
- Multivariable logistic regression
- Evaluation under substantial class imbalance

The project is intended as an exploratory clinical analytics study rather than a validated early-warning or clinical prediction system.

---

# Clinical Question

**Which routinely collected clinical variables differ between ICU patients who develop sepsis and those who do not, and how well do these features discriminate between the two groups?**

### Primary Question

Which physiological and laboratory characteristics are associated with sepsis status in ICU patients?

### Secondary Exploratory Question

How well can a small set of clinically relevant patient-level features discriminate between patients with and without sepsis?

---

# Objectives

- Identify patients with and without sepsis
- Assess missingness at measurement and patient level
- Investigate outcome-dependent missingness
- Transform longitudinal ICU measurements into clinically meaningful patient-level features
- Compare clinical characteristics between sepsis and non-sepsis patients
- Quantify group differences using hypothesis tests and effect sizes
- Develop an interpretable logistic regression baseline model
- Account for substantial class imbalance
- Evaluate model discrimination using ROC and precision-recall analysis

---

# Dataset

The dataset contains ICU time-series measurements from the **PhysioNet/Computing in Cardiology Challenge 2019**.

Selected variables include:

### Vital Signs

- Heart rate (HR)
- Mean arterial pressure (MAP)
- Respiratory rate (Resp)
- Oxygen saturation (O2Sat)

### Laboratory Measurements

- Lactate
- White blood cell count (WBC)
- Creatinine
- Platelets

### Demographics

- Age
- Gender

### Outcome

- SepsisLabel

Each row in the original dataset represents an hourly observation during a patient's ICU stay.

The complete dataset is publicly available via Kaggle. To keep this repository lightweight, only reduced/sample data are stored in the repository where applicable.

---

# Methodology

## 1. Data Acquisition

- ICU dataset downloaded using `kagglehub`
- Individual patient CSV files combined into a single longitudinal dataset
- Patient identifiers generated to enable patient-level aggregation

---

## 2. Data Quality and Missingness Assessment

Missingness was evaluated at two levels.

### Measurement-Level Missingness

Missingness in the original longitudinal dataset was assessed to determine how frequently individual clinical measurements were available across ICU time points.

### Patient-Level Missingness

Following aggregation, missingness was reassessed to determine whether at least one measurement was available for each patient.

Most variables used in the primary model showed relatively low patient-level missingness.

Lactate was a notable exception:

- Approximately **69% of patients had no available lactate measurement**
- Lactate availability differed substantially by sepsis status
- Missingness was therefore considered potentially informative rather than purely random

Because of this, lactate was excluded from the primary multivariable model and reserved for secondary analysis.

---

## 3. Data Validation

Clinical variables were screened using:

- Descriptive statistics
- Distributional quantiles
- Review of extreme values

Several variables contained highly extreme observations. However, these remained clinically plausible in a critically ill population.

Therefore, observations were not excluded solely on the basis of statistical extremity.

---

## 4. Patient-Level Feature Engineering

Longitudinal ICU measurements were transformed into patient-level features.

Feature aggregation was selected according to clinical relevance.

Examples include:

- `HR_mean` — mean heart rate
- `MAP_min` — minimum mean arterial pressure
- `Resp_mean` — mean respiratory rate
- `Lactate_max` — maximum lactate
- `Creatinine_max` — maximum creatinine
- `Platelets_min` — minimum platelet count
- `Age_first` — patient age

The patient-level outcome was defined as:

`SepsisPatient`

where:

- `1` = sepsis occurred during the observed ICU stay
- `0` = no sepsis label occurred

---

# Exploratory Data Analysis

## Cohort Analysis

The final patient-level cohort contained approximately **40,000 ICU patients**.

Sepsis occurred in approximately **7.3%** of patients, resulting in substantial class imbalance.

---

## Univariate Analysis

Clinical variable distributions were examined to assess:

- Distribution shape
- Skewness
- Extreme observations
- Potential differences in measurement characteristics

Several laboratory variables, particularly lactate and creatinine, showed strongly right-skewed distributions.

---

## Clinical Characteristics by Sepsis Status

Patient-level clinical features were compared between patients with and without sepsis.

Descriptive patterns included:

- Higher mean heart rate in patients with sepsis
- Lower minimum MAP
- Higher maximum lactate
- Higher maximum creatinine
- Lower minimum platelet counts
- Substantial overlap in age distributions

Lactate comparisons require particular caution because lactate measurements were substantially more frequently available among patients with sepsis.

---

# Statistical Analysis

Group differences were evaluated using:

- Welch's two-sample t-test
- Mann–Whitney U test for selected skewed variables
- Cohen's d for standardized effect-size estimation

A significance level of **α = 0.05** was used.

## Welch's Two-Sample t-test

| Feature | Statistic | p-value | Cohen's d | Interpretation |
|---|---:|---:|---:|---|
| HR_mean | 18.58 | <0.001 | +0.39 | Higher in sepsis |
| MAP_min | -16.15 | <0.001 | -0.31 | Lower in sepsis |
| Lactate_max | 4.88 | <0.001 | +0.13 | Higher in sepsis |
| Creatinine_max | 9.73 | <0.001 | +0.20 | Higher in sepsis |
| Platelets_min | -6.45 | <0.001 | -0.14 | Lower in sepsis |
| Age_first | 1.42 | 0.157 | +0.03 | No meaningful difference |

Despite highly significant p-values for several variables, standardized effect sizes were generally small.

Heart rate and minimum MAP showed the largest standardized group differences.

This illustrates the importance of considering **effect size in addition to statistical significance**, particularly in large datasets.

---

## Mann–Whitney U Analysis

Selected skewed laboratory variables also showed differences in their distributions:

| Feature | Statistic | p-value |
|---|---:|---:|
| Lactate_max | 10,451,437.5 | <0.001 |
| Creatinine_max | 58,380,432.5 | <0.001 |
| Platelets_min | 41,761,867.0 | <0.001 |

These results complement the mean-based Welch tests but should not be interpreted as testing exactly the same hypothesis.

---

# Correlation Analysis

Pairwise correlations were assessed before multivariable modeling.

No strong correlations were observed among the selected primary clinical features:

- All absolute correlation coefficients were <0.30
- The strongest correlation was between mean heart rate and respiratory rate (`r ≈ 0.26`)

This provided no evidence of relevant multicollinearity based on pairwise correlations.

---

# Primary Logistic Regression Model

An interpretable logistic regression model was developed using routinely available patient-level features:

- `HR_mean`
- `MAP_min`
- `Resp_mean`
- `Creatinine_max`
- `Platelets_min`
- `Age_first`

Lactate was deliberately excluded from the primary model because of its high and outcome-dependent missingness.

---

## Preprocessing

A Scikit-learn pipeline was used to prevent data leakage.

The pipeline included:

1. Median imputation for missing predictor values
2. Standardization using `StandardScaler`
3. Logistic regression

The dataset was split into:

- **70% training data**
- **30% test data**

Stratified sampling preserved the approximately 7.3% sepsis prevalence in both subsets.

---

# Class Imbalance

An unweighted baseline logistic regression model achieved approximately **93% accuracy** but failed to identify any sepsis patients at the default classification threshold.

This demonstrated why accuracy alone is misleading in a strongly imbalanced clinical dataset.

The final primary model therefore used:

`class_weight="balanced"`

to increase the contribution of the minority sepsis class during model fitting.

---

# Model Performance

At the default classification threshold, the balanced logistic regression model achieved approximately:

| Metric | Result |
|---|---:|
| Sepsis Recall | **0.62** |
| Sepsis Precision | **0.12** |
| Sepsis F1-score | **0.20** |
| Accuracy | **0.63** |
| ROC-AUC | **0.67** |
| Average Precision | **0.14** |

Class weighting substantially increased sensitivity compared with the unweighted model but resulted in a large number of false-positive classifications.

This illustrates an important clinical trade-off:

> Increasing sensitivity for sepsis detection can substantially increase false-positive alerts.

---

# Multivariable Associations

Because predictors were standardized before logistic regression, coefficient magnitudes can be compared more directly.

| Feature | Standardized Coefficient | Direction |
|---|---:|---|
| HR_mean | +0.325 | Higher values associated with sepsis |
| MAP_min | -0.313 | Lower values associated with sepsis |
| Creatinine_max | +0.180 | Higher values associated with sepsis |
| Resp_mean | +0.174 | Higher values associated with sepsis |
| Platelets_min | -0.126 | Lower values associated with sepsis |
| Age_first | +0.033 | Minimal association |

Heart rate and minimum MAP showed the strongest multivariable associations with sepsis status.

The direction and relative magnitude of these associations were broadly consistent with the preceding univariate analyses.

---

# Clinical Interpretation

Patients with sepsis showed a pattern of greater physiological disturbance, including:

- Higher heart rate
- Higher respiratory rate
- Lower minimum mean arterial pressure
- Higher maximum creatinine
- Lower minimum platelet count
- Higher maximum lactate among patients with available measurements

These findings are clinically consistent with cardiovascular stress, hemodynamic instability, and organ dysfunction associated with severe systemic illness.

However, statistically significant and clinically plausible associations did **not** translate into strong individual-level discrimination.

The primary model achieved only modest discrimination (`ROC-AUC = 0.67`) and limited precision for the minority sepsis class (`Average Precision = 0.14`).

This demonstrates an important distinction between:

**group-level association**

and

**individual-level predictive performance**.

---

# Key Findings

1. Several routinely collected clinical characteristics differed between patients with and without sepsis.

2. Mean heart rate and minimum MAP showed the largest standardized group differences and strongest multivariable associations.

3. Statistical significance was common because of the large cohort, while effect sizes remained mostly small.

4. Lactate was clinically informative but highly incomplete and showed substantial outcome-dependent missingness.

5. Severe class imbalance made overall accuracy misleading.

6. Class weighting substantially improved sepsis sensitivity but produced many false-positive classifications.

7. The selected patient-level features provided modest discrimination but were insufficient for high-performance sepsis classification.

---

# Secondary Analysis: Lactate and Informative Missingness

Lactate was investigated separately because of its high degree of missingness and its clinical relevance in the assessment of critically ill patients.

## Lactate Availability

Approximately **69% of patients had no recorded lactate measurement**.

Importantly, lactate availability differed substantially by sepsis status:

| Group | Lactate Not Measured | Lactate Measured |
|---|---:|---:|
| Non-Sepsis | 71.5% | 28.5% |
| Sepsis | 38.0% | 62.0% |

This outcome-dependent missingness suggested that lactate measurements were unlikely to be missing completely at random.

To investigate whether the lactate value itself or the availability of a lactate measurement contributed additional discriminatory information, three logistic regression models were compared using the same train-test split and preprocessing strategy.

## Model Comparison

| Model | ROC-AUC | Average Precision | Sepsis Precision | Sepsis Recall | Sepsis F1 |
|---|---:|---:|---:|---:|---:|
| Primary model | 0.666 | 0.141 | 0.117 | 0.619 | 0.197 |
| Primary + Lactate_max | 0.672 | 0.147 | 0.124 | 0.626 | 0.207 |
| Primary + Lactate_max + Lactate_measured | **0.737** | **0.185** | **0.142** | **0.652** | **0.233** |

Adding `Lactate_max` alone resulted in only a small improvement in model discrimination.

In contrast, additionally including a binary indicator representing whether lactate had been measured (`Lactate_measured`) produced a substantially larger improvement across ROC-AUC, Average Precision, precision, recall, and F1-score.

## Model Coefficients

In the full secondary model, the standardized logistic regression coefficients included:

| Feature | Standardized Coefficient |
|---|---:|
| Lactate_measured | **+0.549** |
| HR_mean | +0.258 |
| MAP_min | -0.202 |
| Creatinine_max | +0.177 |
| Resp_mean | +0.172 |
| Platelets_min | -0.046 |
| Age_first | +0.043 |
| Lactate_max | +0.033 |

`Lactate_measured` showed the largest standardized coefficient, whereas the coefficient for the actual maximum lactate value was comparatively small.

## Interpretation

These findings suggest that **lactate measurement availability contained substantially more discriminatory information than the lactate value itself** in this patient-level analysis.

This is consistent with the concept of **informative missingness** in real-world clinical data.

Clinical measurements are not necessarily obtained randomly. Lactate testing may be more likely in patients with greater disease severity, suspected hypoperfusion, or clinical concern for sepsis. Consequently, measurement availability may encode information about clinical decision-making and diagnostic workflow in addition to underlying patient physiology.

The improved performance after adding `Lactate_measured` should therefore **not be interpreted as evidence that the lactate biomarker itself accounts for the observed improvement in discrimination**.

Instead, this analysis demonstrates an important challenge in clinical machine learning:

> **Missingness itself can become predictive when clinical measurement behavior is related to patient condition and clinician decision-making.**

This distinction is particularly important when developing models intended for deployment across different hospitals or clinical workflows, where measurement practices may differ.



# Limitations

- Longitudinal ICU measurements were reduced to patient-level summary statistics.
- Temporal relationships between clinical measurements and sepsis onset were therefore lost.
- Features such as maximum lactate or minimum MAP may include measurements obtained after sepsis onset.
- The current model should **not** be interpreted as an early-sepsis prediction model.
- Lactate showed substantial and outcome-dependent missingness.
- Median imputation simplifies potentially complex missing-data mechanisms.
- Class weighting improves minority-class sensitivity but does not solve the limited discriminatory information contained in the selected features.
- No external validation was performed.
- The analysis is exploratory and is not intended for clinical decision-making.

---

# Clinical Relevance

This project demonstrates several challenges commonly encountered when working with real-world clinical data:

- Longitudinal feature engineering
- Extensive and informative missingness
- Extreme but clinically plausible observations
- Class imbalance
- The distinction between statistical significance and clinical relevance
- The distinction between association and predictive discrimination
- Sensitivity–false-positive trade-offs in clinical classification

The project provides an interpretable foundation for more advanced temporal modeling.

---

# Visualizations

Included visualizations include:

- Patient cohort distribution
- Clinical variable distributions
- Sepsis vs Non-Sepsis boxplots
- Missingness analyses
- Correlation heatmap
- Confusion matrix
- ROC curve
- Precision-recall curve

---

# Tools Used

- Python
- Pandas
- NumPy
- SciPy
- Scikit-learn
- Seaborn
- Matplotlib
- KaggleHub

---

# Future Work – V2

The most important extension is to move from patient-level association analysis toward **temporally valid early-sepsis prediction**.

Potential extensions include:

- Define a clinically meaningful prediction window before sepsis onset
- Restrict predictors to information available before the prediction time
- Engineer temporal features and trends
- Investigate more advanced missing-data strategies
- Evaluate lactate availability and informative missingness
- Compare logistic regression with tree-based models
- Optimize classification thresholds according to clinical objectives
- Evaluate calibration
- Perform cross-validation
- Assess model explainability
- Perform external validation where suitable data are available

A future V2 should explicitly distinguish between **describing patients who develop sepsis** and **predicting sepsis before it occurs**.
