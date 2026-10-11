# DATA1030 Midterm Project — Xinfinity

This repository contains our DATA1030 midterm project, focusing on automotive safety complaints and investigations using data from the National Highway Traffic Safety Administration (NHTSA).

## 1. Project Overview

Our project explores vehicle safety complaint patterns and their potential relationship with formal NHTSA investigations.

**Research Question:** Can historical vehicle complaint patterns help predict whether a vehicle model will be subject to a formal NHTSA investigation within the following 12 months?

The final prediction target and evaluation strategy are still under development.

## 2. Data Sources

**NHTSA Vehicle Complaints API**

https://api.nhtsa.gov/complaints/complaintsByVehicle

Our initial dataset contains:

- 10 vehicle model families
- Model years 2018–2020
- 30 make–model–year combinations
- 9,440 complaint records from the initial collection

The dataset includes complaint dates, vehicle information, reported components, crash and fire indicators, injury counts, and death counts.

**NHTSA Investigations Data**

Investigation data will be collected separately to study potential relationships between complaint patterns and formal investigations.

## 3. Repository Structure

```text
DATA1030-Midterm-Xinfinity/
├── data/
│   ├── raw/
│   ├── processed/
│   └── modeling/
├── notebooks/
│   └── 00_shared_preprocessing.ipynb
├── src/
│   ├── collect/
│   ├── preprocessing/
│   ├── features/
│   └── analysis/
├── figures/
├── results/
├── report/
├── tests/
├── environment.yml
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

`notebooks/00_shared_preprocessing.ipynb`

It performs:

1. **Data Collection:** Retrieves vehicle complaint records from the NHTSA API.
2. **Vehicle Standardization:** Standardizes manufacturer and model names, including BMW 3 Series and Mercedes-Benz C-Class variants.
3. **Date Processing:** Converts complaint filing dates and incident dates into datetime format.
4. **Missing Value Handling:** Standardizes missing VIN values.
5. **Component Cleaning:** Standardizes component text while preserving NHTSA categories.
6. **Data Validation:** Checks duplicate complaint IDs, vehicle years, and implausible incident dates.
7. **Export:** Creates a shared complaint-level CSV for EDA.

Implausible incident dates are flagged rather than overwritten. Reporting delay is excluded for flagged records.

### Output Dataset

`data/processed/complaints_clean.csv`

The initial exported dataset contains 9,440 complaints and 15 columns.

Raw and processed data files are excluded from Git tracking. Team members can generate the dataset by running the preprocessing notebook.

## 6. How to Run

From the repository root, activate the environment and launch Jupyter:

```bash
conda activate data1030
jupyter notebook
```

Open `notebooks/00_shared_preprocessing.ipynb` and run all cells in order.

The notebook retrieves data from the NHTSA API and saves the processed dataset to:

`data/processed/complaints_clean.csv`

**Note:** The API provides live data, so the number of records may change between runs.

## 7. Team Collaboration

We use GitHub for version control and team collaboration.

- Work on separate branches for individual tasks.
- Create Pull Requests before merging changes into `main`.
- Have another team member review changes before merging.
- Use the shared preprocessing code as the starting point for EDA.
- Document additional preprocessing and feature engineering decisions.

## 8. Project Progress

- [x] Repository setup
- [x] Initial complaint data collection
- [x] Shared core preprocessing
- [x] Initial cleaned complaint dataset
- [ ] Exploratory Data Analysis (EDA)
- [ ] Investigation data collection and matching
- [ ] Feature engineering and modeling dataset
- [ ] Machine learning models and evaluation
- [ ] Final report and presentation