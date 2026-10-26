# Medicare Part D Pharmacy Benefit Analytics

Analyzing Medicare Part D claims data (2019–2023) to benchmark pharmacy spend, identify cost outliers, and quantify generic substitution savings — the kind of analysis used in pharmacy benefit consulting to help clients manage rising drug costs.

**[View Interactive Dashboard on Tableau Public →](https://public.tableau.com/views/MedicarePartDPharmacyBenefitAnalytics/Dashboard1)**

[![Dashboard Preview](https://public.tableau.com/static/images/Me/MedicarePartDPharmacyBenefitAnalytics/Dashboard1/1.png)](https://public.tableau.com/views/MedicarePartDPharmacyBenefitAnalytics/Dashboard1)

---

## What This Project Does

Medicare Part D spend grew from $178.9B in 2019 to $275.9B in 2023 — a 54% increase in five years. This project breaks down where that money is going, flags drugs that look like pricing outliers, and calculates how much could be saved by shifting patients to generic equivalents.

The workflow mirrors what a pharmacy benefit analyst would do in a real consulting engagement: pull and clean the data, run exploratory analysis, query specific business questions in SQL, surface anomalies using statistics, and package the findings into something a client team can actually use.

---

## Key Findings

- Brand drugs are 83% of drug count but **90.6% of total spend** — $250.1B vs $25.8B for generics
- Average cost per brand claim: **$4,474** vs **$925** for generics (nearly 5x difference)
- **53 drugs** flagged as high-cost outliers via z-score analysis, including Acthar ($65,725/claim, 38,640 claims) and Revcovi ($331,752/claim)
- **$8.5B in potential savings** identified through generic substitution (conservative 70% adoption rate)
- Spend projected to reach **$340.2B by 2026** based on linear regression forecast (R² = 0.9753)

---

## Project Structure

```
pharmacy-analytics/
├── data/
│   ├── cms_partd_final.csv          # Cleaned long-format dataset (16,151 rows)
│   ├── anomaly_data_2023.csv        # 2023 drug-level data with z-score flags
│   ├── generic_opportunity.csv      # Generic substitution savings estimates
│   └── spend_forecast.csv           # Actual + forecast spend 2019–2026
├── notebooks/
│   ├── data_cleaning.ipynb          # Raw data ingestion, reshaping, classification
│   ├── eda.ipynb                    # Exploratory analysis and trend visualization
│   ├── analysis.ipynb               # Anomaly detection and generic opportunity
│   └── predictive_model.ipynb       # Linear regression spend forecast
├── sql/
│   └── sql_queries.sql              # Business queries against PostgreSQL
└── README.md
```

---

## Data Source

**CMS Medicare Part D Drug Spending by Manufacturer and Product (PY2023)**
Longitudinal file covering 2019–2023, available at [data.cms.gov](https://data.cms.gov/medicare-part-d/medicare-part-d-prescribers-by-drug)

The raw file contains 14,309 records across 46 columns — one row per drug per manufacturer with year-suffixed spend columns (e.g. `Tot_Spndng_2019`, `Tot_Spndng_2020`).

---

## Methodology

### Data Cleaning
The raw CMS file required several non-trivial transformations before it was usable:

- Filtered to `Mftr_Name = 'Overall'` to remove per-manufacturer duplicates (14,309 → 3,598 records)
- Missing values in early years (4,205 missing in 2019 vs 0 in 2023) were documented as new drug approvals — not dropped
- Brand/generic classification derived by comparing `Brnd_Name` to `Gnrc_Name` — no column exists for this in the raw file
- Reshaped from wide to long format using `pd.melt()`, then pivoted metrics back into columns — required for Tableau time series charts
- Final dataset: 16,151 rows, one row per drug per year

### Anomaly Detection
Z-score methodology applied to 2023 cost-per-claim data:

```
z = (drug CPC - mean CPC) / std CPC
```

- **MEDIUM flag:** z > 2.0 (~top 2.5% of drugs)
- **HIGH flag:** z > 3.0

Results: 53 HIGH, 37 MEDIUM, 3,508 Normal. These flags are triage for contract audit, not compliance determinations.

### Generic Substitution Opportunity
Brand drugs matched to generic equivalents by `generic_name`. Savings calculated as:

```
savings per claim = brand CPC - generic CPC
potential savings = savings per claim × brand claim volume
conservative estimate = potential savings × 0.70
```

The 70% rate reflects realistic formulary substitution outcomes — 100% switching is theoretical and doesn't account for medical necessity carve-outs or patient/physician preferences.

### Spend Forecast
Linear regression trained on 5 years of historical spend data (2019–2023):

| Metric | Value |
|--------|-------|
| R-squared | 0.9753 |
| Mean Absolute Error | $4.9B |
| Annual growth rate | $23.8B/year |
| 2026 forecast | $340.2B |

This is a directional forecast suitable for illustrating trend magnitude. A production forecast would incorporate drug mix changes, biosimilar launches, and rebate fluctuations.

---

## SQL Queries

Seven business queries written against a PostgreSQL database loaded with the cleaned dataset, covering:

- Total spend trend by year
- Brand vs generic spend and claims by year
- Top 10 drugs by total spend (2023)
- Average cost per claim by brand vs generic
- Most expensive drugs by cost per claim
- Drugs with the most generic manufacturer competition
- Spend growth by drug from 2019 to 2023

---

## Dashboard

Built in Tableau Public using the cleaned CSV outputs. The dashboard includes:

- 7 KPI tiles across the top (total spend, avg cost/claim, brand share, 2026 forecast, annual growth, generic savings rate)
- Total spend trend 2019–2023
- Spend forecast 2024–2026 with shaded forecast region and dashed line
- Brand vs generic spend comparison over time
- Top 10 drugs by spend (2023)
- Generic substitution opportunity ranked by potential savings
- Cost anomaly detection scatter plot (log scale) with annotated outliers

**[View on Tableau Public →](https://public.tableau.com/views/MedicarePartDPharmacyBenefitAnalytics/Dashboard1)**

---

## Tools

| Tool | Use |
|------|-----|
| Python / pandas / NumPy | Data cleaning, analysis, modeling |
| Matplotlib / Seaborn | Exploratory visualizations |
| scikit-learn | Linear regression forecast |
| PostgreSQL / pgAdmin | SQL business queries |
| Tableau Public | Interactive dashboard |
| ReportLab | Project write-up PDF |
| CMS.gov | Primary data source |

---

## Author

**Thao Le** | Data Analyst | Kansas City, MO
[github.com/xoxothaole](https://github.com/xoxothaole)
