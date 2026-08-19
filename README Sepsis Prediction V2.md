# Early Sepsis Prediction from Hourly ICU Data

Machine-learning project exploring early sepsis prediction from longitudinal ICU data.

The project compares interpretable baseline models with a Random Forest while addressing common challenges in clinical machine learning:

- Repeated observations per patient
- Missing clinical measurements
- Strong class imbalance
- Temporal feature engineering
- Patient-level data leakage
- Clinically relevant threshold selection

> **Clinical disclaimer:** This is an educational AI healthcare portfolio project and not a clinically validated decision-support system.

## Project Objective

Can current clinical measurements, measurement availability and recent clinical trajectories be used to predict the provided hourly `SepsisLabel` in ICU patients?

Unlike a patient-level aggregation approach, this project retains the original hourly observations. The model can therefore generate updated predictions throughout a patient's ICU stay.

## Dataset

The project uses the *Prediction of Sepsis* dataset available through Kaggle and based on the PhysioNet/Computing in Cardiology Challenge 2019.

- 40,336 ICU patients
- 1,552,210 hourly observations
- Vital signs, laboratory measurements and demographic variables
- Binary hourly target: `SepsisLabel`

Each row represents one patient-hour. The provided target already represents an early-prediction label and was therefore not shifted again.

## Notebook

The complete analysis is available in [Sepsis_Prediction_V2-4.ipynb](./Sepsis_Prediction_V2-4.ipynb).

## Project Workflow

1. Data integrity and temporal label validation
2. Patient-level development/test split
3. Analysis of hourly missingness
4. Time-aware missing-value handling
5. Measurement-indicator creation
6. Six-hour temporal feature engineering
7. Patient-grouped cross-validation
8. Model comparison
9. Development-only threshold selection
10. Held-out test evaluation
11. Random Forest feature importance

## Development and Test Design

The split was performed at the patient level and stratified by whether a patient ever had a positive `SepsisLabel`.

| Cohort | Patients | Patient-hours |
|---|---:|---:|
| Development | 32,268 | 1,240,596 |
| Test | 8,068 | 311,614 |

All observations belonging to the same patient remained in one cohort. There was no overlap of patient IDs between development and test data.

## Missing-Value Strategy

Clinical measurements were not available during every patient-hour. Laboratory variables were particularly sparse because they were measured irregularly.

The following strategy was used:

1. Sort observations by `Patient_ID` and `ICULOS`.
2. Create a binary `*_measured_now` indicator before filling missing values.
3. Carry the last available measurement forward within the same patient for no more than six hours.
4. Leave values missing if no recent measurement was available.
5. Impute remaining missing values using the median inside each model pipeline.

Because median imputation was part of the pipeline, imputation values were learned only from the corresponding training data.

## Temporal Features

Six-hour temporal features were created for frequently measured vital signs:

- Heart rate
- Mean arterial pressure
- Respiratory rate
- Temperature
- Oxygen saturation

For every variable, the model included:

- Rolling mean
- Rolling minimum
- Rolling maximum
- Rolling standard deviation
- Change compared with six hours earlier

Only current and previous observations were used. Future measurements were not included. The final feature matrix contained 96 predictors.

## Leakage Prevention

The project uses several safeguards against data leakage:

- Development/test splitting at the patient level
- Cross-validation grouped by `Patient_ID`
- No use of `Patient_ID` as a predictor
- Forward filling within patients only
- Past-looking rolling features
- Measurement indicators created before forward filling
- Median imputation inside model pipelines
- Threshold selection using development predictions only
- No test-set probabilities used for model fitting

## Cross-Validation

Models were evaluated using five-fold `StratifiedGroupKFold` cross-validation. This ensured that all patient-hours belonging to one patient remained in the same fold. Out-of-fold probabilities were generated using `cross_val_predict`.

## Models

The following models were compared:

- Dummy Classifier
- Balanced Logistic Regression
- Unweighted Logistic Regression
- Logistic Regression with measurement indicators
- Logistic Regression with temporal features
- Shallow Decision Tree
- Random Forest

Logistic Regression was used as an interpretable baseline. The Random Forest was selected as the final model because it achieved the strongest discrimination.

## Development Results

| Model | ROC-AUC | Average Precision |
|---|---:|---:|
| Dummy Classifier | 0.492 | 0.018 |
| Balanced LR - values | 0.736 | 0.070 |
| Unweighted LR - values | 0.730 | 0.071 |
| Unweighted LR - values + indicators | 0.755 | 0.077 |
| Unweighted LR - complete temporal features | 0.762 | 0.080 |
| Shallow Decision Tree | 0.740 | 0.083 |
| Random Forest | **0.802** | **0.099** |

The Random Forest outperformed the simpler baseline models. Average Precision remained much lower than ROC-AUC because positive patient-hours represented only approximately 1.8% of the data.

## Threshold Selection

The default probability threshold of `0.5` was not assumed to be clinically appropriate.

Because missing a potential sepsis event may have serious consequences, sensitivity was prioritised. The highest threshold achieving at least 80% hourly sensitivity was selected using out-of-fold Random Forest probabilities from the development cohort.

This threshold was subsequently applied to the held-out test cohort without further adjustment.

Threshold-dependent results were interpreted using:

- Sensitivity
- Specificity
- Precision
- Confusion matrix
- Patient-level timely sensitivity
- Lead time
- Percentage of non-sepsis patients receiving an alert

## Held-Out Test Performance

Threshold-independent performance on the held-out test cohort was:

| Metric | Test result |
|---|---:|
| ROC-AUC | 0.809 |
| Average Precision | 0.103 |

The test results were similar to the grouped cross-validation results, suggesting stable discrimination within the available dataset.

The high-sensitivity operating point nevertheless produced low precision and a substantial false-alert burden. The model is therefore not suitable for clinical deployment in its current form.

## Model Interpretation

Random Forest impurity-based feature importance identified `ICULOS` as the most influential individual feature.

Other influential predictors included:

- Temperature
- Heart rate
- Respiratory rate
- FiO2
- BUN
- Creatinine
- PaCO2
- Temporal vital-sign features

The dominance of `ICULOS` may indicate that the model learned a strong time-in-ICU effect. Although ICU length of stay was available at prediction time and was not direct temporal leakage, its importance raises concerns about transportability.

Feature importance:

- Does not establish causality
- Does not show the direction of an association
- May be divided among correlated features
- May reflect clinical workflow as well as physiology

## Main Limitations

- Retrospective dataset
- No external hospital validation
- Strong class imbalance
- Low precision at a high-sensitivity threshold
- High false-alert burden
- Possible dependence on time in the ICU
- Measurement indicators may encode local clinical workflows
- Some patients were already label-positive at their first available observation
- No prospective evaluation of clinical outcomes or workflow effects

Patients whose first available observation already had a positive `SepsisLabel` were retained. For these patients, no earlier negative observation was available, which limits interpretation of prospective lead time.

## Conclusion

The project demonstrates that current measurements, measurement availability and recent clinical trajectories contain information associated with the provided early sepsis label.

The Random Forest showed stable discrimination and outperformed the baseline models. However, its low precision and substantial false-alert burden illustrate an important distinction between statistical performance and clinical usefulness.

> The model is a reproducible exploratory early-warning prototype, not a clinically validated prediction system.

## Repository Structure

```text
.
├── README.md
└── Sepsis_Prediction_V2-4.ipynb
```

## Installation

Install the required packages:

```bash
pip install pandas numpy matplotlib seaborn scikit-learn kagglehub jupyter
```

## Running the Project

1. Clone the repository.
2. Open the Jupyter notebook.
3. Restart the kernel.
4. Run all cells in order.

```bash
git clone <repository-url>
cd <repository-name>
jupyter notebook Sepsis_Prediction_V2-4.ipynb
```

The notebook downloads the dataset using:

```python
kagglehub.dataset_download(
    "salikhussaini49/prediction-of-sepsis"
)
```

## Technologies

- Python
- pandas
- NumPy
- Matplotlib
- seaborn
- scikit-learn
- Jupyter Notebook
- KaggleHub

## Potential Next Steps

- External validation on an independent hospital dataset
- Probability calibration
- Subgroup performance analysis
- Comparison with established clinical scores
- Evaluation of alternative alert rules
- Prospective clinical workflow assessment

## Disclaimer

This project was created for education and portfolio demonstration. The model has not been clinically validated and must not be used for diagnosis, treatment decisions or patient care.
