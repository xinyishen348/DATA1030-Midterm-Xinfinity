# DATA1030 Midterm Project — Xinfinity

This repository contains our DATA1030 midterm project, focusing on automotive safety complaints and investigations using data from the National Highway Traffic Safety Administration (NHTSA).

## 1. Project Overview

Our project explores vehicle safety complaint patterns and their potential relationship with formal NHTSA investigations.

**Research Question:** Can historical vehicle complaint patterns help predict whether a vehicle model will be subject to a formal NHTSA investigation within the following 12 months?

The final prediction target and evaluation strategy are still under development.

## 2. Data Sources

### NHTSA Vehicle Complaints API

https://api.nhtsa.gov/complaints/complaintsByVehicle

Our initial dataset includes:

- 10 vehicle model families
- Model years 2018–2020
- 30 make–model–year combinations

### Data Collection Snapshot

| Item | Value |
|---|---|
| API retrieval date (UTC) | 2026-10-14 21:49:03 |
| Total complaints | 9,461 |
| Vehicle groups | 30 |
| Earliest filing date | 2017-08-04 |
| Latest filing date | 2026-10-12 |

The dataset includes complaint dates, vehicle information, reported components, crash and fire indicators, injury counts, and death counts.

**Note:** NHTSA API data may be updated over time. Complaint counts and date coverage may change when the notebook is rerun.

### NHTSA Investigations Data

Investigation data will be collected separately to study potential relationships between complaint patterns and formal investigations.

## 3. Repository Structure

```text
DATA1030-Midterm-Xinfinity/
├── data/
│   ├── raw/
│   ├── processed/
│   ├── modeling/
│   └── data_contract.md
├── figures/
├── results/
├── report/
├── src/
│   ├── analysis/
│   │   ├── 01_EDA.ipynb
│   │   ├── 02_EDA.ipynb
│   │   └── 03_EDA.ipynb
│   ├── preprocessing/
│   │   └── 00_shared_preprocessing.ipynb
│   ├── collect/
│   └── features/
├── tests/
├── environment.yml
├── .gitignore
├── LICENSE
└── README.md

```

## 4. Environment

Python 3.14.6

Main packages:

- NumPy 2.4.6
- pandas 3.0.3
- Matplotlib 3.11.0
- seaborn 0.13.2
- scikit-learn 1.9.0

Create and activate the Conda environment:

```bash
conda env create -f environment.yml
conda activate data1030
```

The environment name should match the name defined in `environment.yml`.

## 5. Data Collection and Preprocessing

The shared preprocessing notebook is located at:

`src/preprocessing/00_shared_preprocessing.ipynb`

It performs:

1. **Data Collection:** Retrieves complaint records from the NHTSA API and documents the retrieval date and data coverage.
2. **Vehicle Standardization:** Standardizes vehicle make and model names, including BMW 3 Series and Mercedes-Benz C-Class variants.
3. **Date Processing:** Converts complaint filing dates and incident dates into datetime format.
4. **Missing Value Handling:** Standardizes missing VIN values.
5. **Component Cleaning:** Standardizes component text while preserving NHTSA categories.
6. **Data Validation:** Checks duplicate complaint IDs, vehicle years, and implausible incident dates.
7. **Export:** Creates a shared complaint-level CSV for EDA.

Implausible incident dates are flagged rather than overwritten. Reporting delay is treated as missing for flagged records.

### Cleaned Dataset

Output: `data/processed/complaints_clean.csv`

The latest validated dataset contains:

- 9,461 complaint records
- 15 columns
- 0 duplicate complaint IDs
- 0 missing filing dates
- 27 flagged implausible incident dates
- 13 complaints with multiple Vehicle products
- 23 reporting delays exceeding five years after date filtering

The dataset preserves the original complaint-level records while standardizing fields needed for analysis.

### Data Reproducibility

Raw and processed data files are excluded from Git tracking.

Team members can reproduce the dataset by running the shared preprocessing notebook. Because the NHTSA API is updated over time, running the notebook on different dates may produce slightly different datasets.

For consistent team EDA results, team members should use the same agreed-upon data snapshot.

## 6. How to Run

Clone the repository and navigate to the project root directory.

Create and activate the Conda environment:

```bash
conda env create -f environment.yml
conda activate data1030
```

Launch Jupyter Notebook:

```bash
jupyter notebook
```

The shared preprocessing notebook is located at:

`src/preprocessing/00_shared_preprocessing.ipynb`

The EDA notebooks are located at:

- `src/analysis/01_EDA.ipynb`
- `src/analysis/02_EDA.ipynb`
- `src/analysis/03_EDA.ipynb`

The shared preprocessing notebook generates:

`data/processed/complaints_clean.csv`

**Note:** The notebooks have been moved into the `src/` directory. Their file paths and working-directory assumptions should be checked before rerunning the complete workflow.

For consistent EDA results, use the same validated data snapshot.

## 7. Team Collaboration

We use GitHub for version control and team collaboration.

- Work on separate branches for individual tasks.
- Create Pull Requests before merging changes into `main`.
- Have another team member review changes before merging.
- Use the shared cleaned dataset as the starting point for EDA.
- Keep EDA-specific transformations in individual notebooks.
- Discuss and review changes to shared preprocessing before applying them.

See `data/data_contract.md` for shared data conventions and preprocessing rules.

## 8. Project Progress

- [x] Repository setup
- [x] Initial complaint data collection
- [x] Shared core preprocessing
- [x] Initial cleaned complaint dataset
- [x] Data collection date and filing date coverage documentation
- [ ] Additional vehicle matching and date validation checks
- [x] Initial Exploratory Data Analysis (EDA)
- [ ] Investigation data collection and matching
- [ ] Feature engineering and modeling dataset
- [ ] Machine learning models and evaluation
- [ ] Final report and presentation
