
# DATA1030 Automotive Safety Project — Xinfinity

This repository contains our DATA1030 Applied Data Science midterm and final project, investigating automotive safety complaints and their relationship with formal safety defect investigations conducted by the National Highway Traffic Safety Administration (NHTSA).

The project begins with exploratory data analysis (EDA) and extends into a supervised machine learning task: predicting future NHTSA investigation activity using historical complaint patterns.

## 1. Project Overview

### Research Question

**Can historical vehicle safety complaint patterns predict whether a vehicle model-year group will be subject to a newly opened NHTSA investigation or review action within the next 12 months?**

Our objective is to examine whether complaint frequency, severity, reporting behavior, and component-related patterns provide useful predictive signals for future regulatory investigation activity.

### Project Objectives

1. Explore historical automotive safety complaint patterns across selected vehicle models.
2. Integrate NHTSA complaint and defect investigation datasets.
3. Construct a time-based modeling dataset using historical complaint information.
4. Create a binary target indicating whether a new NHTSA investigation action opens within the following 12 months.
5. Compare machine learning algorithms using time-aware cross-validation.
6. Evaluate predictive performance and interpret important historical safety indicators.

### Prediction Task

- **Task type:** Binary classification
- **Prediction unit:** Vehicle model-year group at a quarterly snapshot date
- **Target:** `future_investigation`
- **Prediction horizon:** 12 months
- **Primary model-selection metric:** Average Precision
- **Validation strategy:** Purged Expanding Window Cross-Validation

---

## 2. Data Sources

### 2.1 NHTSA Vehicle Complaints

Source: https://www.nhtsa.gov/nhtsa-datasets-and-apis

API endpoint:

https://api.nhtsa.gov/complaints/complaintsByVehicle

The complaints dataset provides information about consumer-reported vehicle safety problems, including:

- Vehicle make, model, and model year
- Complaint filing date
- Reported incident date
- Affected vehicle components
- Crash and fire indicators
- Reported injuries and deaths
- Complaint descriptions

### Midterm Dataset

The initial midterm analysis focused on:

- 10 vehicle model families
- Model years 2018–2020
- 30 make–model–year combinations

| Item | Value |
|---|---|
| API retrieval date (UTC) | 2026-10-14 21:49:03 |
| Total complaints | 9,461 |
| Vehicle groups | 30 |
| Earliest filing date | 2017-08-04 |
| Latest filing date | 2026-10-12 |

This dataset was used for exploratory data analysis and initial shared preprocessing.

### Final Project Complaint Dataset

For the final project, the analysis was expanded to model years 2015–2023 across the same 10 selected vehicle model families.

| Item | Value |
|---|---:|
| Vehicle model families | 10 |
| Model years | 2015–2023 |
| Vehicle model-year groups | 90 |
| Unique complaints | 39,905 |

The expanded dataset supports historical feature engineering across quarterly prediction snapshots.

### 2.2 NHTSA Defect Investigations

Source: https://www.nhtsa.gov/nhtsa-datasets-and-apis

The investigation dataset is stored locally at:

`data/raw/Investigation/FLAT_INV.zip`

Investigation records contain:

- Action number
- Vehicle make and model
- Model year
- Investigation opening date
- Investigation closing date, when available
- Investigation subject and summary

The project includes investigation action-number prefixes:

| Prefix | Action Type |
|---|---|
| PE | Preliminary Evaluation |
| EA | Engineering Analysis |
| RQ | Recall Query |
| AQ | Audit Query |
| DP | Defect Petition |

After vehicle matching and deduplication:

| Item | Count |
|---|---:|
| Matched Action–Vehicle Events | 104 |
| Unique Action Numbers | 33 |
| Vehicle Model-Year Groups with Matched Actions | 54 |

Each Action–Vehicle Event is identified by vehicle group and action number.

---

## 3. Data Preprocessing

### 3.1 Shared Midterm Preprocessing

Notebook:

`src/preprocessing/00_shared_preprocessing.ipynb`

The shared preprocessing workflow includes:

1. Collecting and organizing NHTSA complaint data.
2. Standardizing vehicle make and model names.
3. Converting complaint and incident dates into datetime values.
4. Standardizing vehicle component information.
5. Identifying duplicate complaint records.
6. Flagging implausible incident dates.
7. Calculating reporting delays.
8. Exporting the cleaned complaint-level dataset.

Output:

`data/processed/complaints_clean.csv`

### 3.2 Final Project Data Preparation

Notebook:

`src/preprocessing/01_final_data_preparation.ipynb`

The final preprocessing workflow includes:

- Expanding model-year coverage to 2015–2023.
- Integrating NHTSA defect investigation records.
- Standardizing vehicle model-family names.
- Matching investigations to vehicle model-year groups.
- Deduplicating complaint records.
- Constructing historical complaint features.
- Creating the future investigation target.
- Exporting the final modeling dataset.

The expanded complaint dataset is saved locally as:

`data/processed/complaints_final.csv`

### 3.3 Reporting Delay

Reporting delay is defined as:

**Reporting Delay = Complaint Filing Date − Reported Incident Date**

The delay is measured in days.

Incident dates earlier than two years before the vehicle model year are flagged as potentially implausible. Reporting delays associated with flagged dates are treated as missing.

Final complaint dataset validation:

| Item | Count |
|---|---:|
| Total complaints | 39,905 |
| Missing reporting delays | 87 |
| Negative reporting delays | 0 |
| Reporting delays exceeding five years | 209 |

Unusually long reporting delays are retained because they may represent genuinely delayed complaint submissions.

---

## 4. Modeling Dataset Construction

### 4.1 Unit of Analysis

Each observation represents one vehicle model-year group at a quarterly snapshot date.

The primary sample identifiers are:

- `make`
- `model_family`
- `model_year`
- `snapshot_date`

Each combination is unique.

Quarterly snapshots were constructed from January 2016 through October 2025, subject to the model-year eligibility rule.

### 4.2 Final Modeling Dataset

| Item | Value |
|---|---:|
| Vehicle model families | 10 |
| Vehicle model-year groups | 90 |
| Total modeling samples | 2,480 |
| Numeric predictive features | 19 |
| Target columns | 1 |
| Sample identifier columns | 4 |
| Total columns | 24 |
| Positive samples | 357 |
| Negative samples | 2,123 |
| Overall positive rate | 14.40% |

Output:

`data/processed/modeling_dataset.csv`

### 4.3 Target Variable

**Target:** `future_investigation`

For each vehicle group at snapshot date `t`:

- `1`: At least one qualifying NHTSA action opens within the next 12 months.
- `0`: No qualifying NHTSA action opens within the prediction window.

The prediction interval is:

`[snapshot_date, snapshot_date + 12 months)`

The target includes qualifying PE, EA, RQ, AQ, and DP actions.

Each distinct action number is considered a new action when opened.

An action may be associated with multiple vehicle model-year groups and multiple quarterly prediction snapshots.

### 4.4 Target Validation

Target labels were independently recomputed from the reconstructed investigation event table.

| Validation Check | Result |
|---|---:|
| Total samples checked | 2,480 |
| Original positive labels | 357 |
| Recomputed positive labels | 357 |
| Mismatched labels | 0 |

Of the 33 matched action numbers, 32 were associated with at least one positive prediction window.

The remaining action, `EA26007`, opened on November 13, 2026, after the final available prediction window ended.

---

## 5. Historical Feature Engineering

The final modeling dataset contains 19 numerical features based on complaints submitted before each snapshot date.

Historical features use complaint filing dates strictly earlier than the corresponding prediction snapshot.

### 5.1 Complaint Frequency and Trends

| Feature | Description |
|---|---|
| `complaints_last_12m` | Complaints filed during the previous 12 months |
| `complaints_last_6m` | Complaints filed during the previous 6 months |
| `complaints_last_3m` | Complaints filed during the previous 3 months |
| `complaint_trend_3m` | Recent 3-month complaint rate relative to the preceding 9-month rate |

### 5.2 Safety Severity

| Feature | Description |
|---|---|
| `crash_count_12m` | Historical complaints reporting a crash |
| `fire_count_12m` | Historical complaints reporting a fire |
| `injuries_12m` | Total reported injuries |
| `deaths_12m` | Total reported deaths |
| `crash_rate_12m` | Proportion of complaints reporting a crash |
| `fire_rate_12m` | Proportion of complaints reporting a fire |

### 5.3 Component-Related Complaints

| Feature | Description |
|---|---|
| `engine_complaints_12m` | Complaints involving the engine |
| `brake_complaints_12m` | Complaints involving brakes |
| `electrical_complaints_12m` | Complaints involving the electrical system |
| `powertrain_complaints_12m` | Complaints involving the powertrain |
| `airbag_complaints_12m` | Complaints involving airbags |
| `component_diversity_12m` | Number of distinct reported component categories |

### 5.4 Reporting Behavior

| Feature | Description |
|---|---|
| `avg_reporting_delay_12m` | Mean historical reporting delay |
| `median_reporting_delay_12m` | Median historical reporting delay |
| `valid_delay_count_12m` | Number of complaints with valid reporting delays |

### 5.5 Missing Values

Missing values are preserved when a feature cannot be calculated reliably.

Examples include:

- Complaint rates when historical complaint counts are zero.
- Complaint trends when the preceding-period complaint rate is zero.
- Reporting delay statistics when no valid delays exist.

Missing-value handling and scaling must be fitted using training data only, within each cross-validation fold.

---

## 6. Train, Validation, and Test Strategy

### 6.1 Time-Based Splitting

The target predicts future investigation activity over a 12-month horizon.

Random splitting could introduce temporal leakage because neighboring quarterly snapshots have overlapping prediction windows.

Therefore, the project uses a purged chronological splitting strategy.

### 6.2 Final Train / Test Split

| Split | Samples | Positives | Positive Rate |
|---|---:|---:|---:|
| Train Pool | 1,490 | 297 | 19.93% |
| Temporal Gap | 270 | 31 | 11.48% |
| Test | 720 | 29 | 4.03% |

- **Train Pool:** Snapshots through January 2023.
- **Temporal Gap:** April, July, and October 2023.
- **Test:** January 2024 through October 2025.

The temporal gap prevents training label windows from overlapping the beginning of the final test period.

Gap observations are excluded from training and final evaluation.

Saved datasets:

- `data/processed/train.csv`
- `data/processed/test.csv`

### 6.3 Purged Expanding Window Cross-Validation

Three expanding-window folds are used for model selection within the training pool.

Training observations whose 12-month label windows extend beyond the beginning of the validation period are excluded.

| Fold | Train Samples | Train Positives | Validation Samples | Validation Positives |
|---|---:|---:|---:|---:|
| 1 | 410 | 51 | 240 | 29 |
| 2 | 620 | 63 | 280 | 60 |
| 3 | 870 | 111 | 320 | 111 |

The validation years are 2020, 2021, and 2022.

Fold assignments are saved in:

`data/processed/cv_fold_assignments.csv`

The same folds should be used for all model comparisons.

### 6.4 Temporal Leakage Controls

The project uses:

- Historical complaint features constructed before each snapshot date.
- Forward-looking 12-month investigation labels.
- Chronological train/test separation.
- Purging of overlapping training label windows.
- Training-fold-only preprocessing.
- A held-out final test dataset.

The saved Train/Test files and CV assignments were independently reloaded and validated.

---

## 7. Machine Learning Methodology

*Model development and evaluation are the next stage of the final project.*

### 7.1 Candidate Algorithms

The planned comparison includes at least four classification algorithms:

1. **Regularized Logistic Regression:** Linear baseline classifier.
2. **Random Forest:** Ensemble model for nonlinear relationships.
3. **XGBoost:** Gradient-boosted decision trees.
4. **RBF Support Vector Machine:** Nonlinear margin-based classifier.

Hyperparameters will be selected using the predefined chronological validation folds.

### 7.2 Preprocessing

Initial experiments use the 19 numerical historical complaint features.

Vehicle identifiers and snapshot dates are retained for sample identification and temporal splitting but are excluded from the initial numerical baseline.

Preprocessing must follow these rules:

- Apply feature scaling where appropriate.
- Use a model-appropriate missing-value strategy.
- Fit preprocessing transformations only on each training fold.
- Do not use validation or test observations to fit preprocessing.
- Use reproducible random seeds.

### 7.3 Evaluation

The primary model-selection metric is **Average Precision (AP)**.

Additional evaluation metrics may include:

- Precision
- Recall
- F1 Score
- ROC-AUC
- Confusion Matrix

Cross-validation performance will be summarized using scores across validation periods.

Multiple random seeds will be used when model randomness is relevant.

The final test set will be evaluated only after model selection and hyperparameter tuning.

### 7.4 Model Interpretation

Planned interpretability analyses include:

- Global feature importance
- Permutation importance
- Model-specific importance measures
- Local prediction explanations
- SHAP analysis where applicable

---

## 8. Repository Structure

```text
DATA1030-Midterm-Xinfinity/
├── data/
│   ├── raw/
│   ├── processed/
│   │   ├── modeling_dataset.csv
│   │   ├── train.csv
│   │   ├── test.csv
│   │   ├── cv_fold_assignments.csv
│   │   ├── broad_events.csv
│   │   └── model_config.json
│   ├── modeling/
│   └── data_contract.md
│
├── src/
│   ├── preprocessing/
│   │   ├── 00_shared_preprocessing.ipynb
│   │   ├── 01_final_data_preparation.ipynb
│   │   └── 02_modeling_and_splitting.ipynb
│   └── analysis/
│       ├── 01_EDA.ipynb
│       ├── 02_EDA.ipynb
│       └── 03_EDA.ipynb
│
├── figures/
├── results/
├── report/
├── environment.yml
├── .gitignore
├── LICENSE
└── README.md
```

Additional modeling code, evaluation results, and presentation materials will be added as the project progresses.

---

## 9. Environment Setup

The current environment configuration specifies:

- Python 3.14.6
- NumPy 2.4.6
- pandas 3.0.3
- Matplotlib 3.11.0
- seaborn 0.13.2
- scikit-learn 1.9.0
- Jupyter Notebook

Additional modeling dependencies will be documented as the modeling stage progresses.

Create and activate the Conda environment:

```bash
conda env create -f environment.yml
conda activate data1030
```

Launch Jupyter Notebook:

```bash
jupyter notebook
```

Package compatibility and environment installation should be verified on the target system.

---

## 10. Reproducibility and How to Run

### 10.1 Midterm Analysis

Run the shared preprocessing notebook:

`src/preprocessing/00_shared_preprocessing.ipynb`

Then run the EDA notebooks:

- `src/analysis/01_EDA.ipynb`
- `src/analysis/02_EDA.ipynb`
- `src/analysis/03_EDA.ipynb`

### 10.2 Final Modeling Dataset

Final data preparation:

`src/preprocessing/01_final_data_preparation.ipynb`

Time-based splitting and validation:

`src/preprocessing/02_modeling_and_splitting.ipynb`

The preparation workflow generates historical complaint features, investigation event matches, and target labels.

The splitting workflow creates the training pool, final test set, and expanding-window CV assignments.

Large original and intermediate datasets are excluded from normal Git tracking.

Full raw-data reconstruction requires the corresponding NHTSA source files and preprocessing inputs.

Notebook file-path assumptions must be verified before running the entire workflow on another machine.

### 10.3 Modeling Handoff Files

| File | Purpose |
|---|---|
| `train.csv` | Training pool |
| `test.csv` | Held-out final test dataset |
| `cv_fold_assignments.csv` | Train/Validation fold assignments |
| `model_config.json` | Features, target, metrics, seeds, and CV configuration |
| `modeling_dataset.csv` | Full modeling dataset |
| `broad_events.csv` | Investigation event reference table |

Modeling code should use the saved configuration and cross-validation folds rather than generating new random splits.

The test set should remain unused during model selection.

---

## 11. Data Quality and Validation

The final modeling dataset passed the following checks:

| Check | Result |
|---|---:|
| Modeling samples | 2,480 |
| Duplicate sample identifiers | 0 |
| Target label mismatches | 0 |
| Invalid complaint window ordering | 0 |
| Count feature violations | 0 |
| Crash/fire rates outside [0, 1] | 0 |
| Negative numerical feature values | 0 |
| Infinite numerical feature values | 0 |

Some numerical features retain missing values because their underlying statistics are undefined or unavailable.

### Investigation Event Dependence

Although the modeling dataset contains 357 positive snapshots, multiple snapshots can correspond to the same underlying NHTSA action.

The final test set contains:

- 720 samples
- 29 positive samples
- 4 unique investigation action numbers associated with positive snapshots

No positive-label action numbers are shared between the training pool and final test set.

The small number of distinct test-period actions limits the certainty of conclusions about generalization to new investigation events.

---

## 12. Project Limitations

### Limited Independent Investigation Events

Multiple positive snapshots may be associated with the same investigation action.

Consequently, sample-level observations are not necessarily statistically independent.

### Class Imbalance

The overall positive rate is 14.40%, compared with 4.03% in the final test set.

Classification metrics should be interpreted relative to class prevalence and relevant baselines.

### Temporal Distribution Shift

Complaint and investigation patterns may change over time, potentially affecting generalization.

### Vehicle Name Matching

Vehicle family matching requires standardized names and documented alias mappings.

Incomplete or incorrect mappings may affect action associations.

### Historical Data Availability

NHTSA source data may be updated or corrected after the original filing dates.

Reconstructed historical snapshots use complaint filing dates, but the dataset may not preserve all historical record revisions.

Investigation coverage is also limited by the source extract.

### Dataset Scope

The analysis covers 10 selected vehicle model families and model years 2015–2023.

Results should not automatically be generalized to all vehicles.

---

## 13. Team Collaboration

This is a three-member collaborative project.

### Member A — Data Preparation and Splitting

- Integrate complaint and investigation datasets.
- Standardize vehicle model families.
- Construct historical features and targets.
- Implement chronological splitting.
- Generate cross-validation assignments.
- Validate and document the modeling datasets.

### Member B — Model Development

- Implement classification algorithms.
- Build preprocessing pipelines.
- Perform hyperparameter tuning.
- Apply chronological cross-validation.
- Compare validation performance.
- Save models and evaluation outputs.

### Member C — Evaluation and Interpretation

- Evaluate final model performance.
- Analyze classification errors.
- Examine global feature importance.
- Produce local prediction explanations.
- Prepare interpretation figures and results.

All team members contribute to the final presentation and integration of the project.

### GitHub Workflow

- Work on separate branches.
- Avoid overwriting shared files.
- Submit Pull Requests for shared changes.
- Review changes before merging into `main`.
- Maintain consistent preprocessing and target definitions.
- Update documentation when shared modeling conventions change.

See `data/data_contract.md` for shared data conventions.

---

## 14. Project Progress

### Midterm

- [x] Repository initialization
- [x] Initial complaint data collection
- [x] Shared preprocessing
- [x] Initial cleaned dataset
- [x] Exploratory Data Analysis
- [x] Midterm report

### Final — Data Preparation

- [x] Expanded complaint dataset
- [x] Investigation data integration
- [x] Vehicle model-family matching
- [x] Historical feature engineering
- [x] Future 12-month target construction
- [x] Target label validation
- [x] Feature consistency validation
- [x] Purged chronological splitting
- [x] Three-fold expanding-window CV
- [x] Modeling dataset export
- [x] Modeling configuration export
- [x] Independent file loading and CV validation

### Final — Modeling and Reporting

- [ ] Preprocessing pipelines
- [ ] Machine learning model training
- [ ] Hyperparameter tuning
- [ ] Cross-validation model comparison
- [ ] Final test evaluation
- [ ] Global feature importance
- [ ] Local prediction explanations
- [ ] Final figures and results
- [ ] Final report and presentation

---

## 15. References

- NHTSA: https://www.nhtsa.gov/
- NHTSA Datasets and APIs: https://www.nhtsa.gov/nhtsa-datasets-and-apis
- NHTSA Vehicle Complaints API: https://api.nhtsa.gov/complaints/complaintsByVehicle
- scikit-learn: https://scikit-learn.org/stable/
- pandas: https://pandas.pydata.org/docs/
