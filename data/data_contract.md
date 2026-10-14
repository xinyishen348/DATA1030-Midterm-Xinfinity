# DATA1030 Automotive Safety — Data Contract

## 1. Purpose

This document defines the shared data conventions for the DATA1030 Automotive Safety Project.

All team members should use consistent data formats, column names, and shared preprocessing rules.

EDA-specific transformations should be performed in individual notebooks without modifying the shared cleaned dataset.

## 2. Data Structure

- **Raw data:** `data/raw/`
- **Cleaned data:** `data/processed/`
- **Modeling data:** `data/modeling/`

Raw source data should remain unchanged.

Raw and processed datasets are excluded from Git tracking. Team members can regenerate the complaint dataset using the shared preprocessing notebook.

Since the NHTSA API is updated over time, team members should use an agreed-upon data snapshot for consistent analysis.

## 3. Data Granularity and Keys

### Complaint-Level Data

Each row represents one NHTSA complaint.

**Primary key:** `complaint_id`

Multiple complaints may belong to the same make, model, and model year.

Complaint IDs must be unique within the shared complaint dataset.

### Modeling Data (Proposed)

Each row may represent one vehicle model-year at a particular snapshot date.

Proposed primary keys:

- `make`
- `model`
- `model_year`
- `snapshot_date`

Each combination must be unique.

Snapshot frequency, prediction horizon, and investigation matching rules will be finalized after data exploration and investigation data collection.

## 4. Column Naming and Data Types

- Use lowercase `snake_case` for column names.
- Dates should use `YYYY-MM-DD` in exported CSV files.
- `model_year` should be an integer.
- `make` and `model` should use standardized uppercase names.
- Preserve original NHTSA complaint identifiers.
- Use Boolean values for `crash`, `fire`, and `invalid_incident_date`.
- Use numeric values for injury counts, death counts, and reporting delay.
- Document any new columns and their definitions.

## 5. Missing Values

- Missing values in CSV files should use empty fields.
- Do not use literal `"None"` or `"null"` as placeholders for missing values.
- Use `pd.isna()` and `pd.notna()` to detect missing values in pandas.
- Do not replace missing values with zero by default.
- Do not drop complaint records solely because VIN information is missing.
- Preserve invalid incident dates in the original date field and flag them separately.
- Set reporting delay to missing for flagged invalid incident dates.
- Document any additional imputation or exclusion decisions.

## 6. Shared Complaint Dataset

**Output:** `data/processed/complaints_clean.csv`

Each row represents one NHTSA complaint.

The current dataset contains 15 standardized columns:

| Column | Description |
|---|---|
| `complaint_id` | Unique NHTSA complaint identifier |
| `make` | Standardized vehicle make |
| `model` | Standardized vehicle model family |
| `model_year` | Vehicle model year |
| `product_model` | Vehicle model extracted from the NHTSA product record |
| `filed_date` | Date the complaint was filed |
| `incident_date` | Reported incident date |
| `components` | Reported NHTSA component categories |
| `crash` | Whether a crash was reported |
| `fire` | Whether a fire was reported |
| `number_of_injuries` | Number of reported injuries |
| `number_of_deaths` | Number of reported deaths |
| `summary` | Original complaint description |
| `reporting_delay` | Days between incident and complaint filing, where valid |
| `invalid_incident_date` | Boolean flag for implausible incident dates |

### Dataset Snapshot

Initial validated collection:

- **API retrieval date (UTC):** 2026-10-14
- **Total complaints:** 9,461
- **Vehicle model families:** 10
- **Make–model–year combinations:** 30
- **Model years:** 2018–2020
- **Earliest filing date:** 2017-08-04
- **Latest filing date:** 2026-10-12

These values describe the initial documented snapshot and may change when the API is queried again.

## 7. Shared Preprocessing Rules

### Vehicle Standardization

- Normalize vehicle make and model names to uppercase.
- Extract the vehicle model from NHTSA `products` records.
- Map BMW 3 Series variants to `3 SERIES`.
- Map Mercedes-Benz C-Class variants to `C-CLASS`.
- Preserve `product_model` for reference and validation.
- Verify vehicle model years against the extracted Vehicle product.
- Review complaints containing multiple Vehicle products rather than automatically duplicating them.

### Date Processing

- Convert source dates to pandas datetime format.
- Preserve the reported incident date.
- Flag incident dates earlier than two years before the vehicle model year as implausible.

The current rule is:

`incident_year < model_year - 2`

This is a project-specific heuristic for identifying questionable dates, not an official NHTSA validity rule.

- Calculate reporting delay as `filed_date - incident_date`, measured in days.
- Treat reporting delay as missing for flagged incident dates.
- Do not automatically remove unusually long but potentially valid reporting delays.

### Component Processing

- Convert component text to uppercase.
- Remove leading and trailing whitespace.
- Preserve original NHTSA component categories.
- Do not automatically split component strings on commas, since some official categories contain commas.
- Perform additional component categorization in EDA-specific notebooks.

### General Rules

- Never manually edit source data.
- Do not remove valid complaint records without documented justification.
- Use verified complaint identifiers when checking duplicates.
- Keep preprocessing reproducible.
- Avoid introducing future information into historical prediction features.

## 8. Data Validation

The shared preprocessing pipeline checks:

- Missing values and blank strings
- Duplicate complaint identifiers
- Vehicle model-year consistency
- Standardized vehicle names
- Complaints containing multiple Vehicle products
- Incident dates occurring after filing dates
- Future incident dates
- Negative reporting delays
- Implausible incident dates
- Unusually long reporting delays

### Initial Validation Results

For the 2026-10-14 dataset of 9,461 complaints:

| Check | Result |
|---|---:|
| Duplicate complaint IDs | 0 |
| Missing filing dates | 0 |
| Vehicle model-year mismatches | 0 |
| Complaints with multiple Vehicle products | 13 |
| Flagged implausible incident dates | 27 |
| Incident dates after filing | 0 |
| Future incident dates | 0 |
| Negative reporting delays | 0 |
| Reporting delays over five years after date filtering | 23 |

The 13 multiple-Vehicle-product complaints were manually reviewed. In these cases, the first Vehicle product matched the queried vehicle, and additional Vehicle entries contained placeholder values.

Flagged incident dates and long reporting delays are retained for traceability. Reporting delays associated with flagged dates are excluded from delay-based analyses.

Validation results should be recalculated when the dataset is refreshed.

## 9. Collaboration Rules

- Develop changes on separate Git branches.
- Do not push directly to `main`.
- Use Pull Requests for shared code and documentation changes.
- Obtain at least one teammate approval before merging into `main`.
- Keep EDA-specific preprocessing in individual notebooks.
- Discuss changes affecting the shared dataset with the team before applying them.
- Update the Data Contract when changing shared schemas or preprocessing rules.
- Revalidate shared datasets after modifying the preprocessing pipeline.

## 10. Future Decisions

The following remain open:

- Final individual analysis responsibilities
- Final modeling features
- Snapshot frequency
- Prediction horizon
- Investigation matching rules and target construction
- Historical train/validation/test splitting strategy
- Final ML experiments and evaluation metrics

These decisions will be documented as the project progresses.