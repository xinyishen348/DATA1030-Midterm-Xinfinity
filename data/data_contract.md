# DATA1030 Automotive Safety — Data Contract

## 1. Purpose

This document defines the shared data conventions
for the DATA1030 Automotive Safety Project.

All team members must use consistent data formats,
column names, and shared preprocessing rules.

Individual EDA and modeling responsibilities
will be determined later.

## 2. Data Structure

Raw data:
data/raw/

Cleaned data:
data/processed/

Modeling data:
data/modeling/

All raw source data must remain unchanged.

## 3. Data Granularity and Keys

### Complaint-Level Data

Each row represents one NHTSA complaint.

Record identifier:
- complaint_id

Multiple complaints may have the same make,
model, and model_year.

### Modeling Data (Proposed)

Each row represents one vehicle model-year
at a particular snapshot date.

Proposed primary keys:
- make
- model
- model_year
- snapshot_date

Each combination must be unique.

Snapshot frequency and prediction horizon
will be finalized after data exploration.

## 4. Column Naming and Data Types

- Use lowercase snake_case for column names.
- Dates must use YYYY-MM-DD.
- model_year must be an integer when available.
- make and model must use consistent normalized names.
- Preserve original source identifiers.
- Document column definitions and data types.

## 5. Missing Values

- Missing values in CSV files use empty fields.
- Do not write literal "NA", "None", or "null"
  as missing-value placeholders.
- In pandas, use pd.isna() and pd.notna()
  to detect missing values.
- Do not replace missing values with zero by default.
- Document imputation and exclusion decisions.

## 6. Shared Complaint Dataset

Planned output:
data/processed/complaints_clean.csv

Expected standardized columns:
- complaint_id
- make
- model
- model_year
- filed_date
- incident_date
- components

Additional source fields should be preserved
when useful for EDA or feature engineering.

The final schema will be confirmed after
inspecting the NHTSA API response.

Team members may create additional derived
columns in their own analysis modules.

Shared cleaned data must only be changed
through the approved preprocessing pipeline.

## 7. Shared Preprocessing Rules

- Never manually edit raw data.
- Deduplicate using verified complaint IDs.
- Standardize column names and data types.
- Normalize make/model names consistently.
- Validate dates and flag invalid values.
- Preserve missing values when appropriate.
- Document cleaning and exclusion decisions.
- Keep all transformations reproducible.
- Avoid future information in historical features.

## 8. Data Validation

Shared datasets should be checked for:

- Missing values
- Duplicate complaint identifiers
- Invalid dates
- Inconsistent vehicle names
- Incorrect data types
- Unexpected changes in row counts

Modeling datasets must additionally have
unique and non-missing primary keys.

## 9. Collaboration Rules

- Develop changes on separate Git branches.
- Do not push directly to main.
- Use Pull Requests for changes.
- Obtain at least one teammate approval
  before merging into main.
- Update documentation when changing
  shared data schemas or preprocessing.
- Review shared changes before rebuilding
  downstream datasets.

## 10. Future Decisions

The following remain open:

- Individual team responsibilities
- Final modeling features
- Snapshot frequency
- Prediction horizon
- Investigation matching rules
- Final ML experiments

These decisions will be documented after
the team completes initial data exploration.
