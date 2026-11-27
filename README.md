<div align="left">

# Predicting Groundwater Levels in Columbus, OH
**Time series analysis and forecasting of groundwater levels using historical hydrological (USGS) and meteorological (NOAA) data.**

</div>

---

## 📌 Project Overview
* **Objective:** Forecast groundwater level fluctuations in **Columbus, Ohio** using multi-year time series modeling.
* **Data Sources:** [USGS National Water Information System](https://usgs.gov) (groundwater depth) & [NOAA National Centers for Environmental Information](https://noaa.gov) (precipitation & temperature).
* **Methodology:** Time series decomposition, stationarity testing, and statistical forecasting via `statsmodels`.

## Data Processing & Quality Control

1. Dataset Overview & Completeness
Both datasets span January 1, 2006 to December 31, 2025 (7,305 expected daily records) with zero duplicate timestamps. While the raw files contain no explicit NaN values, aligning the series against a complete calendar grid reveals 28 missing dates (0.38%) in the groundwater record.

| Dataset | Expected Days | Observed Rows | Missing Dates | Raw `NaN`s | Post-Merge `NaN`s |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Groundwater** (`water_level`) | 7,305 | 7,277 | 28 | 0 | 28 |
| **Climate** (`prcp`, `tmax`, `tmin`) | 7,305 | 7,305 | 0 | 0 | 0 |

2. Merged Schema & Data Types
The merged DataFrame (df) contains 7,305 rows and 4 columns indexed by a monotonic daily DatetimeIndex (freq='D').

| Column | Data Type | Description | Missing Values |
| :--- | :--- | :--- | :---: |
| `date` *(index)* | `datetime64[ns]` | Daily observation timestamp (`2006-01-01` to `2025-12-31`) | 0 |
| `water_level` | `float64` | Daily groundwater level measurement | 28 (0.38%) |
| `prcp` | `float64` | Daily precipitation | 0 |
| `tmax` | `int64` | Daily maximum temperature | 0 |
| `tmin` | `int64` | Daily minimum temperature | 0 |

3. Groundwater Gap CharacterizationInstead of immediately imputing the 28 missing groundwater observations, gaps were isolated into consecutive intervals to guide downstream modeling decisions. Four gaps are brief ($\le 5$ days), while a single 17-day gap in Spring 2018 requires dedicated handling during imputation or sequence modeling.

| Gap ID | Start Date | End Date | Missing Days | Modeling Consideration |
| :---: | :---: | :---: | :---: | :--- |
| 1 | `2011-04-25` | `2011-04-29` | 5 | Short gap; suitable for linear/spline interpolation |
| 2 | `2016-12-28` | `2016-12-28` | 1 | Single-day gap; negligible impact |
| 3 | `2018-04-23` | `2018-05-09` | **17** | **Extended gap; requires careful imputation or masking** |
| 4 | `2022-10-24` | `2022-10-25` | 2 | Short gap; suitable for linear/spline interpolation |
| 5 | `2024-05-15` | `2024-05-17` | 3 | Short gap; suitable for linear/spline interpolation |

## Data Processing & Preparation Workflow Summary

1. **Ingestion & Standardization:** Loaded raw groundwater and climate CSV files (skipping metadata header rows) and standardized column headers (`date`, `water_level`, `prcp`, `tmax`, `tmin`) by stripping whitespace and converting to lowercase.
2. **Temporal Completeness & Gap Audit:** Evaluated both datasets against a continuous daily calendar grid from `2006-01-01` to `2025-12-31` (7,305 days). Confirmed 100% completeness for climate records and identified 28 omitted dates (0.38%) in the groundwater series distributed across five discrete intervals (ranging from 1 to 17 consecutive days).
3. **Deduplication & Date Normalization:** Normalized timestamps to midnight (`datetime64[ns]`) and enforced strict uniqueness assertions (`0` duplicate dates in either source).
4. **Outer Merge & Reindexing:** Performed a validated one-to-one outer join on `date` and reindexed onto the complete 7,305-day `DatetimeIndex` (`freq='D'`), explicitly exposing the 28 missing groundwater dates as `NaN`s without premature imputation.
5. **Automated Validation:** Verified final dataset integrity via programmatic assertions confirming exact row count (`7,305`), monotonic chronological ordering, index uniqueness, and exact expected missing-value counts (`28` in `water_level`, `0` across climate variables).
