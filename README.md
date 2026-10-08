# Aviation Accident Analysis
# Aviation Accident Data Preprocessing & Cleaning Pipeline

## Overview
This repository contains the data cleaning and preprocessing workflow for the NTSB Aviation Accident dataset (1948–2023). The objective of this phase is to transform raw, noisy aviation safety data into a structured, standardized dataset (`cleaned_aviation_data.csv`) ready for downstream risk and statistical analysis.

The cleaning pipeline filters the dataset to active, professionally built aircraft and standardizes feature columns while engineering key derived metrics for occupant numbers and total hull loss.

---

## Required Libraries
- **Python 3.x**
- **Pandas**
- **NumPy**
- **Matplotlib**
- **Seaborn**

---

## Data Cleaning & Feature Engineering Workflow

### 1. Column Selection & Temporal Filtering
- **Selected Columns**: Extracted 18 relevant features from the raw dataset covering event details, aircraft specifications, injury counts, phase of flight, and environmental conditions.
- **Active Operational Window (1983–2023)**: Filtered `Event.Date` to include incidents from **1983 onwards**, assuming a maximum 40-year operational lifespan for active aircraft makes and models.

### 2. Occupant Imputation & Injury Metrics
- **Injury Severity**: Replaced missing entries in `Injury.Severity` with `'UNKNOWN'` and standardized strings to uppercase.
- **Injury Tally Imputation**: Imputed missing values in `Total.Fatal.Injuries`, `Total.Serious.Injuries`, `Total.Minor.Injuries`, and `Total.Uninjured` with `0`.
- **Derived Metric (`Total_Occupants`)**: Created `Total_Occupants` by summing all four injury and uninjured tallies per event to estimate onboard occupancy.

### 3. Damage Standardization & Hull Loss Metric
- **Damage Categorization**: Standardized `Aircraft.damage` values to uppercase and imputed missing values as `'UNKNOWN'`.
- **Derived Metric (`Is_Destroyed`)**: Engineered a binary indicator (`1` for total loss/`DESTROYED`, `0` otherwise) to isolate complete hull write-offs.

### 4. Professional Build & Manufacturer Filtering
- **Professional Build Filter**: Retained only factory-built aircraft (`Amateur.Built == 'NO'`), excluding homebuilt and experimental kits.
- **Make Standardization**: Cleaned `Make` entries by stripping whitespace and converting text to uppercase.
- **Statistical Volume Threshold**: Filtered out low-frequency manufacturers, retaining only makes with **$\ge 50$ recorded incidents** (`robust_makes`) to ensure statistical sample reliability.

### 5. Aircraft Unique Identification
- **Model Cleaning**: Removed records with missing `Model` designations.
- **Derived Identifier (`Make_Model`)**: Concatenated `Make` and `Model` into a single standardized string (e.g., `CESSNA 172P`) to uniquely classify plane types.

### 6. Categorical Variable Cleaning & Consolidation
- **Engine Type (`Engine.Type`)**: Standardized text formatting, filled nulls with `'UNKNOWN'`, and mapped invalid/cryptic entries (`'UNK'`, `'NONE'`, `'LR'`) to `'UNKNOWN'`.
- **Weather Conditions (`Weather.Condition`)**: Unified weather strings into standard uppercase categories (`VMC`, `IMC`, `UNKNOWN`), resolving duplicate labels (`UNK` $\rightarrow$ `UNKNOWN`).
- **Engine Count (`Number.of.Engines`)**: Replaced non-viable zero counts (`0.0`) with `NaN` and imputed missing values using the dataset median (`1.0`).
- **Purpose of Flight (`Purpose.of.flight`)**: Consolidated fragmented sub-categories (e.g., merging `PUBLIC AIRCRAFT - FEDERAL/STATE/LOCAL` into `PUBLIC AIRCRAFT`, and consolidating air race variants into `AIR RACE / SHOW`).
- **Phase of Flight (`Broad.phase.of.flight`)**: Standardized casing, filled missing entries with `'UNKNOWN'`, and remapped `'OTHER'` to `'UNKNOWN'`.

### 7. Feature Pruning & Export
- **Column Dropping**: Removed sparse or redundant columns:
  - `Amateur.Built` (redundant after filtering to professional builds)
  - `FAR.Description` (excessive missing values)
  - `Aircraft.Category` (excessive missing values)
- **Data Export**: Exported the final processed dataset (70,452 rows × 19 columns) to `cleaned_aviation_data.csv` without row indices.

---

## Output Dataset Schema (`cleaned_aviation_data.csv`)

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| `Event.Id` | Object | Unique incident report identifier |
| `Event.Date` | Datetime | Date of the aviation incident |
| `Make` | Object | Standardized aircraft manufacturer name |
| `Model` | Object | Aircraft model designation |
| `Aircraft.damage` | Object | Standardized damage level (`SUBSTANTIAL`, `DESTROYED`, `MINOR`, `UNKNOWN`) |
| `Injury.Severity` | Object | Standardized severity level |
| `Total.Fatal.Injuries` | Float | Count of fatal injuries (imputed) |
| `Total.Serious.Injuries` | Float | Count of serious injuries (imputed) |
| `Total.Minor.Injuries` | Float | Count of minor injuries (imputed) |
| `Total.Uninjured` | Float | Count of uninjured occupants (imputed) |
| `Number.of.Engines` | Float | Engine count (imputed) |
| `Engine.Type` | Object | Standardized engine propulsion type |
| `Weather.Condition` | Object | Weather conditions (`VMC`, `IMC`, `UNKNOWN`) |
| `Broad.phase.of.flight` | Object | Phase during incident (`LANDING`, `TAKEOFF`, `CRUISE`, etc.) |
| `Purpose.of.flight` | Object | Consolidated flight purpose |
| `Year` | Integer | Extracted incident year ($\ge 1983$) |
| `Total_Occupants` | Float | **Engineered**: Total occupants onboard |
| `Is_Destroyed` | Integer | **Engineered**: Binary hull destruction flag (`1` = Destroyed) |
| `Make_Model` | Object | **Engineered**: Composite unique plane identifier |
