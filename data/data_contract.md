
# DATA1030 Automotive Safety — Data Contract

## 1. Purpose

This document defines the shared data conventions, dataset schemas, preprocessing rules, target definitions, and validation strategies for the DATA1030 Automotive Safety Project.

All team members should follow these conventions to maintain consistency, reproducibility, and prevent data leakage.

The project includes two stages:

- **Midterm:** Exploratory analysis of automotive safety complaints.
- **Final:** Binary classification of future NHTSA investigation activity using historical complaint features.

---

## 2. Data Organization

The repository uses the following data directories:

| Directory | Purpose |
|---|---|
| `data/raw/` | Original NHTSA source datasets |
| `data/processed/` | Cleaned complaints, investigation events, and modeling datasets |
| `data/modeling/` | Reserved for additional modeling-related artifacts |

Raw source files must remain unchanged.

Large raw datasets and intermediate complaint-level datasets are excluded from normal Git tracking.

The following small modeling handoff files are intended to be tracked:

- `modeling_dataset.csv`
- `train.csv`
- `test.csv`
- `cv_fold_assignments.csv`
- `broad_events.csv`
- `model_config.json`

These files are stored in `data/processed/`.

---

## 3. Dataset Granularity and Primary Keys

### 3.1 Complaint-Level Data

Each row represents one unique NHTSA complaint.

**Primary key:** `complaint_id`

The final expanded complaint dataset contains 39,905 unique complaint records.

File:

`data/processed/complaints_final.csv`

Complaint identifiers must be unique.

### 3.2 Investigation Event Data

Each row represents one NHTSA investigation action associated with one vehicle model-year group.

Primary keys:

- `make`
- `model_family`
- `model_year`
- `action_number`

File:

`data/processed/broad_events.csv`

The matched event dataset contains:

| Item | Count |
|---|---:|
| Action–Vehicle Events | 104 |
| Unique Action Numbers | 33 |
| Vehicle Groups with Actions | 54 |

### 3.3 Modeling Data

Each row represents one vehicle model-year group observed at one quarterly snapshot date.

**Primary keys:**

- `make`
- `model_family`
- `model_year`
- `snapshot_date`

The final dataset contains:

| Item | Count |
|---|---:|
| Vehicle model families | 10 |
| Vehicle model-year groups | 90 |
| Total samples | 2,480 |
| Numeric features | 19 |
| Target columns | 1 |
| Identifier columns | 4 |
| Total columns | 24 |

Primary keys must be unique.

File:

`data/processed/modeling_dataset.csv`

---

## 4. Shared Column Naming and Data Types

All datasets should follow these conventions:

- Use lowercase `snake_case` for column names.
- Store dates in `YYYY-MM-DD` format when exporting CSV files.
- Use standardized uppercase vehicle make and model-family values.
- Store `model_year` as an integer.
- Preserve original NHTSA complaint identifiers.
- Store numeric counts as nonnegative values.
- Preserve missing numerical values as missing rather than using arbitrary replacements.
- Avoid modifying original complaint or investigation records manually.

### Key Columns

| Column | Description |
|---|---|
| `make` | Standardized vehicle manufacturer |
| `model_family` | Standardized vehicle model family |
| `model_year` | Vehicle model year |
| `snapshot_date` | Quarterly prediction snapshot |
| `future_investigation` | Binary future investigation target |

---

## 5. Complaint Preprocessing Rules

### 5.1 Vehicle Standardization

Vehicle make and model names must be standardized consistently across complaints and investigations.

Examples include:

- BMW 3 Series model variants mapped to `3 SERIES`.
- Mercedes-Benz C-Class variants mapped to `C-CLASS`.
- Audi Q5 Hybrid/PHEV variants mapped to `Q5`.
- Toyota Camry Hybrid mapped to `CAMRY`.

Model aliases must be documented and reviewed before changes are introduced.

### 5.2 Date Processing

Complaint filing and incident dates are converted to datetime values.

Reporting delay is calculated as:

`reporting_delay = filed_date - incident_date`

The result is measured in days.

Incident dates are flagged as implausible when:

`incident_year < model_year - 2`

This is a project-specific heuristic, not an official NHTSA rule.

Reporting delay is treated as missing when the incident date is flagged as implausible.

### 5.3 Reporting Delay Validation

Final expanded complaint dataset:

| Check | Result |
|---|---:|
| Total complaints | 39,905 |
| Missing reporting delays | 87 |
| Negative reporting delays | 0 |
| Reporting delays over five years | 209 |

Unusually long reporting delays are retained unless there is evidence that the dates are incorrect.

### 5.4 Complaint Deduplication

Each unique complaint identifier should appear once in the final complaint-level dataset.

Duplicate complaint entries must be resolved before generating historical features.

Multiple component categories associated with a complaint may be preserved in the same complaint record.

---

## 6. Investigation Matching Rules

Investigation records are matched to selected vehicle model-year groups using:

- `make`
- `model_family`
- `model_year`

The project considers the following action prefixes:

| Prefix | Action Type |
|---|---|
| PE | Preliminary Evaluation |
| EA | Engineering Analysis |
| RQ | Recall Query |
| AQ | Audit Query |
| DP | Defect Petition |

Only investigation records with valid opening dates are eligible for target construction.

Duplicate records for the same action and vehicle group are removed.

A new action number is treated as a distinct action even when it may relate to an earlier investigation of the same safety issue.

Investigation matching rules should not be changed without reviewing their effects on the target labels.

---

## 7. Snapshot Construction

The modeling dataset uses quarterly snapshots.

Snapshot dates range from:

- **First snapshot:** 2016-01-01
- **Last snapshot:** 2025-10-01

The dataset includes 90 possible vehicle model-year groups, covering model years 2015–2023.

Snapshots are included when the snapshot calendar year is at least the vehicle model year.

The final dataset contains 2,480 snapshot observations.

---

## 8. Target Definition

### 8.1 Binary Target

**Column:** `future_investigation`

The target indicates whether a new qualifying NHTSA action opens within the 12 months following the snapshot.

Target values:

- `1`: At least one qualifying action opens during the prediction window.
- `0`: No qualifying action opens during the prediction window.

The prediction interval is:

`[snapshot_date, snapshot_date + 12 months)`

The start date is included and the end date is excluded.

### 8.2 Matching Logic

A qualifying action must:

1. Match the same vehicle make.
2. Match the same standardized vehicle model family.
3. Match the same model year.
4. Have an eligible action prefix.
5. Have an opening date within the prediction window.

One qualifying action is sufficient for a positive target.

### 8.3 Target Distribution

| Target | Count |
|---|---:|
| Negative (0) | 2,123 |
| Positive (1) | 357 |
| Total | 2,480 |

Overall positive rate: 14.40%.

### 8.4 Target Validation

The target labels were recomputed from the reconstructed investigation event table.

Validation results:

- Samples checked: 2,480
- Original positives: 357
- Recomputed positives: 357
- Mismatched labels: 0

All saved labels matched the reconstructed labels.

---

## 9. Feature Definitions

The final modeling dataset contains 19 historical numerical features.

All features must use only complaints with filing dates before the corresponding snapshot date.

### 9.1 Complaint Frequency

- `complaints_last_12m`
- `complaints_last_6m`
- `complaints_last_3m`
- `complaint_trend_3m`

Complaint counts represent historical filing activity.

The trend compares the complaint rate in the previous three months with the rate in the preceding nine months.

### 9.2 Safety Severity

- `crash_count_12m`
- `fire_count_12m`
- `injuries_12m`
- `deaths_12m`
- `crash_rate_12m`
- `fire_rate_12m`

Crash and fire rates are calculated relative to the historical complaint count.

### 9.3 Component Categories

- `engine_complaints_12m`
- `brake_complaints_12m`
- `electrical_complaints_12m`
- `powertrain_complaints_12m`
- `airbag_complaints_12m`
- `component_diversity_12m`

Component features represent complaint activity involving major vehicle systems.

### 9.4 Reporting Behavior

- `avg_reporting_delay_12m`
- `median_reporting_delay_12m`
- `valid_delay_count_12m`

These summarize historical reporting delays within the previous 12 months.

---

## 10. Missing Value Rules

Missing values should remain missing when a statistic cannot be calculated.

Examples:

- A complaint rate is undefined when the complaint count is zero.
- A complaint trend may be undefined when the comparison period has zero complaints.
- Reporting-delay statistics are undefined without valid reporting delays.

Observed missing values include:

| Feature | Missing Values |
|---|---:|
| `complaint_trend_3m` | 175 |
| `avg_reporting_delay_12m` | 94 |
| `median_reporting_delay_12m` | 94 |
| `crash_rate_12m` | 93 |
| `fire_rate_12m` | 93 |

### Modeling Requirements

- Do not fit imputers on the entire dataset before cross-validation.
- Do not use validation or test observations to estimate preprocessing parameters.
- Apply model-appropriate missing-value handling.
- Fit scaling transformations using training observations only.
- Do not arbitrarily replace all missing values with zero.
- Do not apply simple mean/median imputation or row deletion as a default strategy.

All learned preprocessing should be included within the model training pipeline.

---

## 11. Train, Validation, and Test Splitting

### 11.1 Strategy

The project uses Purged Time-Based Splitting.

The 12-month prediction horizon creates overlapping label windows between neighboring quarterly snapshots.

Chronological splitting and temporal purging are used to reduce leakage risks.

### 11.2 Final Train/Test Split

| Split | Samples | Positives | Positive Rate |
|---|---:|---:|---:|
| Train Pool | 1,490 | 297 | 19.93% |
| Temporal Gap | 270 | 31 | 11.48% |
| Test | 720 | 29 | 4.03% |

**Train Pool:**

Snapshots from January 2016 through January 2023.

**Temporal Gap:**

Snapshots from April through October 2023.

**Test:**

Snapshots from January 2024 through October 2025.

The temporal gap is excluded from both model training and final evaluation.

### 11.3 Expanding Window Cross-Validation

Three expanding-window folds are defined within the training pool.

| Fold | Train | Validation | Train Positives | Validation Positives |
|---|---:|---:|---:|---:|
| 1 | 410 | 240 | 51 | 29 |
| 2 | 620 | 280 | 63 | 60 |
| 3 | 870 | 320 | 111 | 111 |

Validation periods:

- Fold 1: 2020
- Fold 2: 2021
- Fold 3: 2022

Each fold excludes training samples whose prediction-label windows overlap the validation period.

### 11.4 Reproducible CV Assignments

All model comparisons must use:

`data/processed/cv_fold_assignments.csv`

The assignments are linked to samples through:

- `make`
- `model_family`
- `model_year`
- `snapshot_date`

Do not replace these folds with randomly generated train/validation splits.

The training pool may be used for final model refitting after model selection.

The held-out test dataset must not be used for hyperparameter tuning.

---

## 12. Model Input Configuration

Modeling configuration is stored at:

`data/processed/model_config.json`

The configuration specifies:

- Target variable
- Sample identifiers
- Numerical feature list
- Primary evaluation metric
- Cross-validation strategy
- Number of folds
- Random seeds

The initial baseline uses 19 numerical historical complaint features.

The following fields are excluded from the initial numerical input matrix:

- `make`
- `model_family`
- `model_year`
- `snapshot_date`
- `future_investigation`

Vehicle identifiers and snapshot dates remain available for sample matching and temporal validation.

The target must never be included in the feature matrix.

---

## 13. Evaluation Rules

### Primary Metric

**Average Precision**

Average Precision is selected as the primary model-selection metric because the task has an imbalanced target distribution.

### Additional Metrics

Models should also be assessed using:

- Precision
- Recall
- F1 Score
- ROC-AUC
- Confusion Matrix

### Validation Requirements

- Use the three predefined chronological CV folds.
- Report performance for each fold.
- Summarize cross-validation performance using mean and standard deviation.
- Compare scores with appropriate class-prevalence baselines.
- Use multiple random seeds when model randomness is relevant.
- Perform model selection without consulting the final test scores.

### Final Test Evaluation

The held-out test set contains:

- 720 samples
- 29 positive samples
- 4 distinct NHTSA action numbers linked to positive snapshots

Final test performance should be interpreted cautiously because relatively few distinct investigation events contribute to the positive class.

---

## 14. Data Validation

The final modeling dataset passed the following checks:

| Check | Result |
|---|---:|
| Duplicate sample keys | 0 |
| Target label mismatches | 0 |
| Invalid complaint window ordering | 0 |
| Feature count violations | 0 |
| Invalid crash/fire rates | 0 |
| Negative numerical feature values | 0 |
| Infinite numerical feature values | 0 |

All saved cross-validation assignments were independently reloaded and validated.

Additional data-source limitations and potential historical record revisions should be documented when interpreting results.

---

## 15. Reproducibility

### Main Notebooks

Midterm preprocessing:

`src/preprocessing/00_shared_preprocessing.ipynb`

Final data preparation:

`src/preprocessing/01_final_data_preparation.ipynb`

Modeling dataset splitting:

`src/preprocessing/02_modeling_and_splitting.ipynb`

### Final Modeling Files

| File | Purpose |
|---|---|
| `modeling_dataset.csv` | Full modeling dataset |
| `train.csv` | Training pool |
| `test.csv` | Final held-out test dataset |
| `cv_fold_assignments.csv` | Predefined Train/Validation assignments |
| `broad_events.csv` | Investigation event reference table |
| `model_config.json` | Modeling configuration |

Large raw and intermediate datasets are stored locally and excluded from normal Git tracking.

The final modeling handoff files are intended to be shared through the repository.

Full raw-data reconstruction requires the corresponding NHTSA source files and preprocessing inputs.

---

## 16. Collaboration Rules

- Work on separate Git branches.
- Do not overwrite another member's work without coordination.
- Submit Pull Requests before merging shared changes.
- Keep modeling inputs consistent across team members.
- Use the saved feature configuration and CV assignments.
- Document any modification to preprocessing or target construction.
- Revalidate saved datasets after changing data preparation logic.
- Keep the final test set separate from model selection.

---

## 17. Known Limitations

### Investigation Event Dependence

Multiple positive snapshots may correspond to the same underlying investigation action.

Therefore, positive samples should not be treated as fully independent investigation events.

### Temporal Distribution Shift

Investigation prevalence differs across training and testing periods.

This may affect model performance and generalization.

### Vehicle Alias Matching

Vehicle-family matching depends on standardized model aliases.

Matching rules may not capture all naming variations.

### Historical Data Availability

NHTSA complaint and investigation records may be updated after their original filing or opening dates.

The available data does not necessarily preserve all historical revisions.

### Dataset Scope

The study is limited to 10 selected vehicle model families and model years 2015–2023.

Findings should not automatically be generalized to all vehicles.

---

## 18. Project Status

### Completed

- [x] Shared complaint preprocessing
- [x] Final complaint dataset construction
- [x] Investigation matching
- [x] Historical feature engineering
- [x] Binary target construction
- [x] Target validation
- [x] Feature consistency checks
- [x] Purged Train/Test splitting
- [x] Expanding Window Cross-Validation
- [x] Modeling dataset export
- [x] Modeling configuration export
- [x] Independent file loading validation

### Remaining

- [ ] Model-specific preprocessing pipelines
- [ ] Classification model development
- [ ] Hyperparameter tuning
- [ ] Cross-validation model comparison
- [ ] Final test evaluation
- [ ] Global and local model interpretation
- [ ] Final figures and results
- [ ] Final report and presentation
