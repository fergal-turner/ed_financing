# e Education Financing: Indicator Inventory and Definitions

This README documents exactly which indicators are pulled in the data preparation notebooks, where they come from, and how they are defined.

## Analysis Summary From Notebook 01 (First Two Headings)

This section summarizes the content under the first two headings in `Code/Data Analysis/01. Spending, Income & LAYS.ipynb`:

- Does GDPPC Correlate with LAYS?
- What is the relationship between GDPPC and LAYS?

It excludes the "Other Drivers of Learning" section.

### Methodology

1.  Build analysis sample:

- Start from `all_data`.
- Keep observations with non-missing `lays` and `gdppc_2015_usd`.
- Drop countries labeled "Not classified" in income group.
- Retain core columns (`iso3`, `year`, `income_group`, `lays`, `gdppc_2015_usd`).
- Collapse to one row per country (`groupby("iso3").first()`) for cross-country analysis.

2.  Visual diagnostics:

- Plot LAYS against GDP per capita in levels and with a log-scaled x-axis.
- Color points by income group to inspect heterogeneity.

3.  Functional-form tests for GDPPC-LAYS relationship:

- Linear OLS model using standardized GDP per capita (`gdppc_s`).
- Log-linear OLS model using `log_gdppc`.
- Quartic OLS model using polynomial terms (`gdppc_s`, `gdppc2`, `gdppc3`, `gdppc4`).
- Predict fitted values on a common grid and compare curve shapes and model fit.

4.  Income-group slope heterogeneity:

- Estimate an interaction model (log GDPPC with income-group dummies and interactions).
- Estimate separate log-GDPPC regressions by income group (LIC, LMIC, UMIC, HIC).
- Compare sign and magnitude of coefficients and visualize group-specific trend lines.

### Key Findings

1.  GDP per capita and learning outcomes are positively related overall, but the association is nonlinear.
2.  The log-GDPPC specification provides the best fit among tested forms and is consistent with diminishing returns: gains in LAYS are steeper at lower income levels and flatter at higher income levels.
3.  The quartic model captures additional curvature and turning points, but does not outperform the log model in explanatory power in this notebook.
4.  The relationship differs by income group:

- For low-income countries, the within-group slope is weakly negative and not statistically distinguishable from zero in the notebook write-up.
- For lower-middle- and upper-middle-income countries, the slope is positive and stronger.
- High-income countries continue to show positive association but with flatter marginal gains than poorer groups.

5.  Visual diagnostics suggest clusters of positive outliers among LMICs and UMICs (better-than-predicted LAYS) and weaker-than-predicted performance among some LICs; the notebook notes these as visual patterns that were not fully validated in additional diagnostics.

### Interpretation

The first two sections support a threshold-style interpretation: income growth is strongly associated with improved learning outcomes at lower income levels, but additional income alone becomes less predictive as countries grow richer. This motivates the later analysis on additional drivers while preserving GDPPC as a central baseline control.

## Quick Variable Dictionary

| Type | Working variable | Source | Indicator ID / construction |
|------------------|------------------|------------------|------------------|
| Outcome | `lays` | Our World in Data (HCI/LAYS series) | Mean of `hd_hci_lays_ma` and `hd_hci_lays_fe` |
| Outcome | `lays_male` | Our World in Data (HCI/LAYS series) | `hd_hci_lays_ma` |
| Outcome | `lays_female` | Our World in Data (HCI/LAYS series) | `hd_hci_lays_fe` |
| Outcome | `pcr_pct` | UNESCO UIS API | `CR.1` |
| Outcome | `pcr_modelled_pct` | UNESCO UIS API | `CR.MOD.1` |
| Outcome | `pcr_wpia` | UNESCO UIS API | `CR.1.WPIA` |
| Predictor | `expenditure_pctbudget_imf` | UNESCO UIS API | `XGOVEXP.IMFCALC` |
| Predictor | `expenditure_pctbudget_uis` | UNESCO UIS API | `XGOVEXP.IMF` |
| Predictor | `expenditure_pctgdp` | UNESCO UIS API | `XGDP.FSGOV` |
| Predictor | `capex_pcttotal` | UNESCO UIS API | `XSPENDP.FDPUB.FNCAP` |
| Predictor | `currex_pcttotal` | UNESCO UIS API | `XSPENDP.FDPUB.FNCUR` |
| Predictor | `all_staff_pcttotal` | UNESCO UIS API | `XSPENDP.FDPUB.FNS` |
| Predictor | `teaching_staff_pcttotal` | UNESCO UIS API | `XSPENDP.FDPUB.FNTS` |
| Predictor | COFOG sector shares | IMF SDMX API (`IMF.STA,GFS_COFOG`) | `GF01_T` to `GF10_T`, transformation `POTO_PT` |
| Predictor | COFOG education sub-shares | IMF SDMX API (`IMF.STA,GFS_COFOG`) | `GF09*` family normalized by `GF09_T` |
| Control | `sap` | UNESCO UIS API | `SAP.1` |
| Control | `pop_total` | World Bank WDI | `SP.POP.TOTL` |
| Control | `sap_share` | Derived (UIS + WDI) | `sap / pop_total` |
| Control | `gdppc_2015_usd` | World Bank WDI | `NY.GDP.PCAP.KD` |
| Control | `gdp_growth_pct` | World Bank WDI | `NY.GDP.MKTP.KD.ZG` |
| Control | `inflation_pct` | World Bank WDI | `FP.CPI.TOTL.ZG` |
| Control | `debt_pct_gdp` | World Bank WDI | `GC.DOD.TOTL.GD.ZS` |
| Control | FSI score/rank and pillars | Fragile States Index (Fund for Peace) | `Total`, `Rank`, `C1-C3`, `E1-E3`, `P1-P3`, `S1-S2`, `X1`, `Change from Previous Year` |
| Control | private primary enrollment share | UNESCO UIS API | `ETOIP.1.PR` |
| Control | household private education spending (% GDP) | UNESCO UIS API | `XGDP.FSHH.FFNTR` |

## Sources used

- UNESCO UIS API (via `api_helpers.uis_api`)
- World Bank API, Source 2: World Development Indicators (via `api_helpers.world_bank_api`)
- IMF SDMX API, flow `IMF.STA,GFS_COFOG` (via `api_helpers.imf_api`)
- Fragile States Index (Fund for Peace), annual Excel files in `Data/FSI Raw Data/`
- Our World in Data grapher extract for Human Capital Index LAYS series

## 1) Outcomes

### Outcome A: Learning-Adjusted Years of Schooling (LAYS)

| Working variable | Pulled field(s) | Source | Definition |
|------------------|------------------|------------------|------------------|
| `lays` | Derived from `hd_hci_lays_ma` and `hd_hci_lays_fe` | Our World in Data grapher CSV: `learning-adjusted-years-of-schooling-gender-scatter` | Learning-adjusted years of schooling combines quantity of schooling with learning quality. In this project, `lays` is calculated as the mean of male and female LAYS. |
| `lays_male` | `hd_hci_lays_ma` | Our World in Data (HCI series) | Male learning-adjusted years of schooling. |
| `lays_female` | `hd_hci_lays_fe` | Our World in Data (HCI series) | Female learning-adjusted years of schooling. |

### Outcome B: Primary Completion Rate (PCR)

Pulled in `Code/Data Preparation/02. Y - Outcomes.ipynb` and stored in `Data/pcr.csv`.

| Working variable | Indicator ID | Source | Definition (source metadata) |
|------------------|------------------|------------------|------------------|
| `pcr_pct` | `CR.1` | UNESCO UIS API | Completion rate, primary education, both sexes (%). |
| `pcr_modelled_pct` | `CR.MOD.1` | UNESCO UIS API | Completion rate, primary education, both sexes (modelled data) (%). |
| `pcr_wpia` | `CR.1.WPIA` | UNESCO UIS API | Completion rate, primary education, adjusted wealth parity index (WPIA). |

## 2) Predictors

### Predictor block A: Government expenditure by function (COFOG)

Pulled in `Code/Data Preparation/01. X - Predictors.ipynb` using:

- `imf_get_gfs_cofog(transformations="POTO_PT", frequency="A", start_year=2000, end_year=2026)`

Definition of transformation:

- `POTO_PT`: percent of total outlays (share of total government expenditure).

#### A1. Aggregate COFOG sectors (saved to `Data/cofog_all.csv`)

Indicators are the COFOG aggregate `GF01_T` to `GF10_T`, renamed to labels in the output table:

- General public services, Transactions
- Defence, Transactions
- Public order and safety, Transactions
- Economic affairs, Transactions
- Environmental protection, Transactions
- Housing and community amenities, Transactions
- Health, Transactions
- Recreation, culture and religion, Transactions
- Education, Transactions
- Social protection, Transactions

Definition: each column is the share of total outlays allocated to that COFOG function (from IMF GFS COFOG data, transformation `POTO_PT`), converted from percent to proportion in the notebook.

#### A2. Education COFOG family (saved to `Data/cofog_edu.csv`)

Indicators are from the `GF09` family (`^GF09\d*_T$`) and output under label names:

- Education, Transactions (total education function)
- Pre-primary and primary education, Transactions
- Secondary education, Transactions
- Post-secondary non-tertiary education, Transactions
- Tertiary education, Transactions
- Education not definable by level, Transactions
- Education n.e.c., Transactions
- Subsidiary services to education, Transactions
- R&D Education, Transactions

Definition: each education sub-function is normalized by the education total (`GF09_T`) in the notebook, so these are shares of education spending by sub-function.

### Predictor block B: Recurrent vs capital education expenditure (UIS)

Pulled in `Code/Data Preparation/01. X - Predictors.ipynb` and saved to:

- `Data/expenditure_long.csv`
- `Data/expenditure_wide.csv`

All pulled as percentages, then divided by 100 in the notebook to store proportions.

| Working variable | Indicator ID | Source | Definition (source metadata) |
|------------------|------------------|------------------|------------------|
| `expenditure_pctbudget_imf` | `XGOVEXP.IMFCALC` | UNESCO UIS API | Expenditure on education as a percentage of total government expenditure (%) (IMF calculation). |
| `expenditure_pctbudget_uis` | `XGOVEXP.IMF` | UNESCO UIS API | Expenditure on education as a percentage of total government expenditure (%) (UIS calculation). |
| `expenditure_pctgdp` | `XGDP.FSGOV` | UNESCO UIS API | Government expenditure on education as a percentage of GDP (%). |
| `capex_pcttotal` | `XSPENDP.FDPUB.FNCAP` | UNESCO UIS API | Capital expenditure as a percentage of total expenditure in public institutions (%). |
| `currex_pcttotal` | `XSPENDP.FDPUB.FNCUR` | UNESCO UIS API | Current expenditure as a percentage of total expenditure in public institutions (%). |
| `all_staff_pcttotal` | `XSPENDP.FDPUB.FNS` | UNESCO UIS API | All staff compensation as a percentage of total expenditure in public institutions (%). |
| `teaching_staff_pcttotal` | `XSPENDP.FDPUB.FNTS` | UNESCO UIS API | Teaching staff compensation as a percentage of total expenditure in public institutions (%). |

## 3) Controls

### Control block A: Education system controls (UIS + WB)

Pulled in `Code/Data Preparation/03. X - Controls.ipynb`.

| Working variable | Indicator ID / construction | Source | Definition |
|------------------|------------------|------------------|------------------|
| `sap` | `SAP.1` | UNESCO UIS API | School age population, primary education, both sexes (number). |
| `pop_total` | `SP.POP.TOTL` | World Bank WDI | Total population (midyear estimate; de facto resident population). |
| `sap_share` | `sap / pop_total` | Derived from UIS + WDI | Share of population that is of primary-school age. |

Related pull also in the controls notebook (education system/private financing context):

| Pulled variable (long format) | Indicator ID | Source | Definition |
|------------------|------------------|------------------|------------------|
| private primary enrollment share | `ETOIP.1.PR` | UNESCO UIS API | Percentage of enrolment in primary education in private institutions, both sexes (%). |
| private household education spending vs GDP | `XGDP.FSHH.FFNTR` | UNESCO UIS API | Initial private expenditure on education (household) as a percentage of GDP (%). |

### Control block B: State fragility (FSI)

Pulled from annual FSI files `fsi-YYYY.xlsx` (2011-2023) and saved to `Data/fsi.csv`.

| Variable(s) in file | Source | Definition |
|------------------------|------------------------|------------------------|
| `Rank`, `Total` | Fragile States Index (Fund for Peace, CAST framework) | Composite fragility rank and total score for each country-year. |
| `C1` to `C3`, `E1` to `E3`, `P1` to `P3`, `S1` to `S2`, `X1` columns | Fragile States Index | The 12 CAST indicators: Security Apparatus, Factionalized Elites, Group Grievance, Economy/Economic Decline, Economic Inequality/Uneven Development, Human Flight and Brain Drain, State Legitimacy, Public Services, Human Rights and Rule of Law, Demographic Pressures, Refugees and IDPs, External Intervention. |
| `Change from Previous Year` | Fragile States Index | Year-over-year change in total fragility score. |

### Control block C: Macro controls (World Bank)

Pulled in `Code/Data Preparation/03. X - Controls.ipynb` via `wb_get(..., source=2)` with indicators below.

| Working variable | Indicator ID | Source | Definition (WDI metadata) |
|------------------|------------------|------------------|------------------|
| `gdppc_2015_usd` | `NY.GDP.PCAP.KD` | World Bank WDI | GDP per capita (constant 2015 US\$). |
| `gdp_growth_pct` | `NY.GDP.MKTP.KD.ZG` | World Bank WDI | GDP growth (annual %). |
| `inflation_pct` | `FP.CPI.TOTL.ZG` | World Bank WDI | Inflation, consumer prices (annual %), based on CPI annual percentage change. |
| `debt_pct_gdp` | `GC.DOD.TOTL.GD.ZS` | World Bank WDI | Central government debt, total (% of GDP). |

## Notes on units and transformations

- `expenditure_*`, `capex_pcttotal`, `currex_pcttotal`, `all_staff_pcttotal`, and `teaching_staff_pcttotal` are stored as proportions in this project (`percent / 100`).
- `cofog_all.csv` also stores sector shares as proportions (`percent / 100`).
- `cofog_edu.csv` is normalized to education total (`GF09_T`), so sub-sector values represent within-education shares.

## Dataset Outputs and Recent Updates

The data preparation notebooks currently write the following core files to `Data/`:

- `iso3.csv`: country lookup (`iso3`, `country_name`) plus `wb_income_group` pulled from World Bank country metadata.
- `lays.csv`: outcome file with `lays`, `lays_male`, `lays_female` by `iso3` and `year`.
- `pcr.csv`: primary completion outcomes from UIS API.
- `cofog_all.csv`: aggregate COFOG shares as proportions.
- `cofog_edu.csv`: COFOG education sub-function shares normalized by total education spending.
- `expenditure.csv` (plus `expenditure_long.csv`, `expenditure_wide.csv`): UIS expenditure indicators, stored as proportions.
- `uis_controls.csv`: SAP, total population, SAP share, and private financing controls.
- `fsi.csv`: Fragile States Index panel with harmonized country IDs and snake_case column labels.
- `fsi_summary.csv`: reduced FSI file with `iso3`, `year`, and `fsi_total`.
- `macro.csv`: World Bank macro controls (`gdppc_2015_usd`, `gdp_growth_pct`, `inflation_pct`, `debt_pct_gdp`) plus `income_group` mapped from `iso3.csv`.
- `all_data.csv`: merged country-year analysis dataset from outcomes, predictors, and controls.
- `lays_panels.csv`: stacked LAYS interval panels (`panel` id 1-5) used for long-difference analysis.

## How `lays_panels` Was Constructed

Construction is defined in `Code/Data Preparation/00. Combined Dataset.ipynb`.

### Step 1: Build base LAYS sample

- Start from `all_data` and retain rows with non-missing `lays`.

### Step 2: Build five two-year panels

Each panel keeps only countries with observations in both years of that interval:

- `panel1`: 2010 to 2017
- `panel2`: 2010 to 2018
- `panel3`: 2010 to 2020
- `panel4`: 2017 to 2020
- `panel5`: 2018 to 2020

For each panel, data are sorted by `iso3, year`, duplicate `iso3-year` rows are removed, and countries are retained only when `nunique(year) == 2` within the interval.

### Step 3: Build panel-level change datasets

Using `build_panel_change_dataset(...)`, each panel also has a transformed country-level dataset:

- `baseline_<var>`: value at interval start year.
- `delta_<var>`: annualized change from start to end year.

Annualization denominators:

- panel1: `/ 7`
- panel2: `/ 8`
- panel3: `/ 10`
- panel4: `/ 3`
- panel5: `/ 2`

In this transformation, key columns are excluded from delta/baseline expansion (`iso3`, `year`, `income_group`).

### Step 4: Stack panels

- Add panel IDs (`'1'` to `'5'`) to each interval panel.
- Concatenate to `all_panels`.
- Export as `Data/lays_panels.csv`.