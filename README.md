# Do Government Guarantees Help Financial Stability? Evidence from an Emerging Market

This repository holds all the code I wrote as a Research Associate at the University of Hyderabad (UoH) for the project **"Do Government Guarantees Help Financial Stability? Evidence from an Emerging Market"**, funded by the **NSE – NYU Stern Initiative on the Study of Indian Capital Markets**.

**Supervisors:** Dr. Vijaya Bhaskar Marisetty, Dr. Madhu Kalimipalli, Dr. R. L. Shankar

---

## In one line

I built the end-to-end R + Stata pipeline that tests whether government capital infusions into 21 Indian public sector banks (2008–2018) reduced their default and systemic risk, using difference-in-differences, IV, Heckman-selection and placebo tests against private bank and NBFC control groups.

---

## Contents

1. [Research question](#research-question)
2. [Research design](#research-design)
3. [Repository structure](#repository-structure)
4. [Software requirements](#software-requirements)
5. [Input data and folder layout](#input-data-and-folder-layout)
6. [Run order](#run-order)
7. [Variable dictionary](#variable-dictionary)
8. [Model specifications](#model-specifications)
9. [Output tables](#output-tables)
10. [Known issues and caveats](#known-issues-and-caveats)

---

## Research question

Between 2008 and 2018 the Government of India repeatedly recapitalised public sector banks. The project asks whether these capital infusions, which act as an implicit government guarantee, made the recipient banks safer, both on their own (default risk) and for the financial system as a whole (systemic and network risk). It also asks whether the effect depends on the size of the infusion, on bank characteristics, and on periods of macro stress, and whether repeated infusions create moral hazard.

---

## Research design

**Unit of observation:** bank/financial firm × quarter, 2008–2018 (44 quarters). Firms are identified by their Prowess code (`id`). Firm `41860` is dropped throughout.

**Treatment group (`category = 0`):** the 21 public sector banks that received capital infusions.

**Control groups:** each analysis is run four times, once against each control group:

| Sample variable | Treatment | Control | `category` of control |
|---|---|---|---|
| `A_B` | Infused public banks | Public banks without infusions | 1 |
| `A_C` | Infused public banks | Private banks | 2 |
| `A_D` | Infused public banks | Public NBFCs | 3 |
| `A_E` | Infused public banks | Private NBFCs | 4 |

A regression restricted with `if A_C != .` therefore uses only treated banks plus private banks.

**Core idea:** a difference-in-differences comparison of risk measures for treated vs control firms, before vs after infusion quarters, with an extra interaction for large infusions. I then test robustness with instrumental variables, Heckman selection correction, placebo infusion dates, stress-period splits, alternative definitions of "large" infusions, and quarter-specific treatment timing.

---

## Repository structure

| File | Language | Role |
|---|---|---|
| `Extraction_and_Cleaning.R` | R | Reads all raw files, reshapes bank financials from wide to long, merges risk measures, financials, macro factors and capital infusions into one quarterly panel ("mother" file) |
| `Risk_Cleaning.do` | Stata | Labels variables, fixes categories, builds all treatment, control, post-infusion, large-infusion, lag, stress, placebo and quarter-specific variables; saves `PD, DTD and Systemic Risk.dta` |
| `Placebo.R` | R | Generates random placebo post-infusion quarters; exports the list of actual infusion quarters |
| `Risk_Analysis.do` | Stata | Main results: Tables 6, 7, 8, 9, 13, 14, 16, IV regressions, placebo regressions, t-tests |
| `Risk_Heckman.do` | Stata | Heckman selection-corrected versions of the main regressions (Table 10), three instrument sets (V1–V3) |
| `Risk_Checklist_-related_Analysis.do` | Stata | Revisions requested in the supervisor checklist of 20 July 2021 (Checklist items 2, 4, 5, 6, 7, 8) |
| `Checklist_7.do` | Stata | Standalone version of Checklist 7: Heckman models on the `A_B` sample with year-specific treatment |
| `Qtr_spec_Analysis.do` | Stata | Re-estimates Tables 7–9 with quarter-specific treatment timing and moral-hazard / median-based large-infusion dummies |
| `Agg_Risk_Measures.do` | Stata | Aggregate (sector-level) risk-spread regressions on post-infusion dummies (Table 15) |
| `Regressions_with_coefplots.do` | Stata | Table 6 with quarter-specific treatment, plotted as a coefficient plot |
| `indep_merge_loop_attempt.R` | R | Unfinished attempt to replace the repeated reshape code in `Extraction_and_Cleaning.R` with a function; not used in the pipeline |

---

## Software requirements

**R** with the packages `tidyverse`, `readxl`, `lubridate`, `haven`, `writexl` and `eeptools`.

**Stata** (version 13 or later; the R code writes `.dta` version 13) with these user-written packages from SSC:

```stata
ssc install outreg2
ssc install coefplot
ssc install estout      // estpost, esttab
ssc install ivregress2
```

---

## Input data and folder layout

The raw data is **not included** in this repository. The scripts expect this layout in Stata and are relative to the project root in R:

```
Capital Infusion/
├── Mother/
│   ├── sysriskvariables_v6.csv                     # MES, SRISK, NSRISK, CoVaR, volatilities
│   ├── All_PD_clean data.xlsx / All_PD_clean data2.xlsx     # probability of default
│   ├── All_DTD_clean data.xlsx / All_DTD_clean data2.xlsx   # distance to default
│   ├── Network Risk.csv                            # network risk decomposition, centrality
│   ├── All cap_infuse_amount data cleaned.csv      # capital infusion amounts by id-year-quarter
│   ├── Datastream_data extraction for all groups.xlsx   # company names (sheet "cleaned")
│   ├── Independent Variables/
│   │   ├── Indeps.xlsx                             # bank financials, one sheet per variable
│   │   ├── Capflows_ Ecoc Policy_Uncert_betas.xlsx # capital-flow and policy-uncertainty betas
│   │   └── Local US Factors.xlsx                   # local and US macro factors, VIX
│   └── Reformed Mother/                            # outputs of the R cleaning step
├── New folder/
│   └── Aggragate risk data_ for_sovereign risk hyp.xlsx  # aggregate risk spreads (Table 15)
├── Results/                                        # regression tables (.xls, .rtf)
│   └── Graphs/                                     # coefficient plots (.tif)
└── PD, DTD and Systemic Risk.dta                   # final analysis panel
```

---

## Run order

1. **`Extraction_and_Cleaning.R`** — builds `Mother/Reformed Mother/PD, DTD and Systemic Risk Mother.csv`, `Indeps Reformed.csv` and `Network.dta`.
2. **`Risk_Cleaning.do`, first part** (up to the first `save`) — creates `PD, DTD and Systemic Risk.dta`.
3. **`Placebo.R`** — reads that `.dta` and writes `Placebo.csv`. Convert it to `Placebo.dta` in Stata (the `import delimited` line for this is commented out in `Risk_Cleaning.do`).
4. **`Risk_Cleaning.do`, rest of the file** — merges the placebo dates and adds placebo and quarter-specific variables, then saves the final `.dta`.
5. **Analysis scripts**, in any order: `Risk_Analysis.do`, `Risk_Heckman.do`, `Qtr_spec_Analysis.do`, `Risk_Checklist_-related_Analysis.do`, `Checklist_7.do`, `Regressions_with_coefplots.do`.
6. **`Agg_Risk_Measures.do`** — independent of the panel; runs on the aggregate-spread Excel file. Checklist 6 in the checklist script reuses its output `Agg Risk Measures.dta`.

---

## Variable dictionary

### Identifiers and groups

| Variable | Description |
|---|---|
| `id` | Prowess company code |
| `co_name` | Company name |
| `year`, `quarter` | Calendar year and quarter |
| `yq` | Encoded year-quarter (used for year-quarter fixed effects) |
| `category` | 0 = infused public bank, 1 = public bank (no infusion), 2 = private bank, 3 = public NBFC, 4 = private NBFC |
| `A_B`, `A_C`, `A_D`, `A_E` | Sample filters: treated vs one control group (see table above) |

### Dependent variables: risk measures

| Variable | Description |
|---|---|
| `pd12_month_100` | 12-month probability of default × 100 |
| `slope_100` | PD term-structure slope: (60-month PD − 12-month PD) × 100 |
| `dtd` | Distance to default |
| `mes_1p_100`, `mes_5p_100` | Marginal Expected Shortfall at 1% / 5% × 100 |
| `srisk_1p_100`, `srisk_5p_100` | SRISK at 1% / 5% × 100 |
| `nsrisk_1p_100`, `nsrisk_5p_100` | Normalised SRISK at 1% / 5% × 100 |
| `covar_1p_100`, `covar_5p_100` | CoVaR at 1% / 5% × 100 |
| `totvol_100`, `ivol_100` | Total and idiosyncratic volatility × 100 |
| `network_risk` | Network risk (decomposition measure from `Network Risk.csv`) |
| `Degree`, `BetCent` | Network degree and betweenness centrality (Checklist 8) |
| `PD_/MES_/COVAR_/NSRISK_/Network_Risk_A_C_sprd`, `..._A_D_sprd` | Aggregate risk spreads between treated banks and private banks / public NBFCs (Table 15) |

### Treatment and timing variables

| Variable | Description |
|---|---|
| `cap_infuse_amount` | Capital infused in that quarter (missing if none) |
| `cap_qtr` | 1 if the firm received an infusion in that quarter |
| `treat_dummy` | 1 for infused public banks (`category = 0`) |
| `post_inf_dummy` | 1 in the calendar quarters flagged as post-infusion (hand-coded list, 2009Q1–2018Q4) |
| `median_ruled_dummy` | "Large Infusions": 1 for treated banks flagged as large recipients under the median rule (hand-coded list of ids) |
| `treat_post` | `treat_dummy × post_inf_dummy` |
| `treat_post_median` | `treat_dummy × post_inf_dummy × median_ruled_dummy` |
| `postlarge_inf` | 1 in the quarter of an infusion ≥ 937.5 and the next two quarters |
| `cap_inf_tot_assets`, `cap_inf_deposits`, `cap_inf_tier_1` | Infusion scaled by total assets, deposits and Tier 1 capital |
| `post_large_inf_ratio`, `_dep`, `_tier1` | Post-large-infusion windows (t, t+1, t+2) based on each scaled measure |
| `treat_postlarge_ratio`, `_1`, `_2` | Treatment × each of the three scaled large-infusion dummies (Table 16) |
| `stress_2011`, `stress_2016`, `stress_2018` | Macro stress year dummies |
| `post_stress_*`, `treat_post_stress_*`, `treat_post_median_stress_*`, `median_stress_*` | Stress-year interactions (Table 13) |

### Quarter-specific and moral-hazard variables

| Variable | Description |
|---|---|
| `Qtr_treat` | 1 in a quarter in which the firm received an infusion |
| `Qtr_post_inf` | 1 in the infusion quarter and the following two quarters |
| `Qtr_pre_inf` | 1 if the firm received an infusion in any of the previous eight quarters |
| `Qtr_post_pre` | `Qtr_post_inf × Qtr_pre_inf` (repeat infusion) |
| `Large_inf_MH` | Moral-hazard large infusion: firm's total infusions last year above that year's median (`A_B` sample) |
| `CurrentQ_Large_inf` | Infusion above the current quarter's median, extended two quarters forward |
| `Large_inf_8Q` | Infusion above the median of all infusions in the previous eight quarters, extended two quarters forward (missing for 2008–2009) |
| `Qtr_post_MH_Large`, `Qtr_post_CurrentQ_Large`, `Qtr_post_8Q_Large` | `Qtr_post_inf` × each large-infusion definition |
| `pub_yr_treat` | Year-specific treatment: 1 for a public bank in any year it received an infusion (Checklist 2, 7) |
| `pub_yr_treat_post`, `pub_yr_treat_post_median` | Year-specific treatment × post, and × large infusions |
| `large_inf_times` | Large infusion defined by count: 1 if the bank received more than 7 infusions (Checklist 4) |
| `treat_post_large_times` | `treat_dummy × post_inf_dummy × large_inf_times` |

### Placebo variables

| Variable | Description |
|---|---|
| `placebo_post_inf` | Random placebo post-infusion quarter (from `Placebo.R`), set to 0 in actual infusion quarters |
| `placebt1`, `placebt2` | Placebo infusion timing shifted one and two quarters earlier |
| `treat_placebo_post`, `treat_placebopost_median`, `treat_placebt1`, `treat_placeb1_median`, `treat_placebt2`, `treat_placeb2_median` | Placebo versions of the DiD interactions |

### Controls and bank financials

| Variable | Description |
|---|---|
| `local_factor` | Local (Indian) market factor |
| `default_factor`, `level_factor`, `slope_factor` | US default spread, yield level and yield slope factors |
| `vix` | Quarterly change in VIX |
| `total_assets`, `deposits`, `roe`, `loan_to_assets`, `tier_1_capital`, `tier_1_ratio` | Bank size, funding, profitability, lending and capital |
| `total_debt_to_common_equity`, `total_debt_to_total_capital`, `long_term_debt`, `debt_to_total_assets` | Leverage measures |
| `deposits_to_total_assets`, `interest_coverage_ratio`, `mkt_to_book` | Funding mix, coverage and valuation |
| `cf_beta`, `policy_beta` | Bank return sensitivity to capital flows and to economic policy uncertainty |
| `lag_*` | Four-quarter lags of financials; one-quarter lags of `cf_beta` and `policy_beta` |
| `lagg_*` | Lags built inside the checklist scripts (one quarter in Checklist 5, four quarters in Checklist 7) |
| `hi_*` | 1 if a treated bank's characteristic is above the treated-group median, 0 if below (missing for control firms); used for the channel splits in Table 14 |

---

## Model specifications

All standard errors are clustered by bank (`vce(cl id)`) unless stated otherwise. Each dependent variable is run on all four samples (`A_B`, `A_C`, `A_D`, `A_E`) unless stated otherwise.

### Baseline difference-in-differences (Tables 7, 8, 9)

$$
Risk_{it} = \beta_1 Treat_i + \beta_2 Post_t + \beta_3 Large_i + \beta_4 (Treat_i \times Post_t) + \beta_5 (Treat_i \times Post_t \times Large_i) + \gamma' X_t + \text{FE} + \varepsilon_{it}
$$

`X_t` = local factor, US default, level and slope factors, and change in VIX. Each model is estimated under five fixed-effect setups: none, year FE, quarter FE, firm + year FE, and firm + quarter FE (terms absorbed by the fixed effects are dropped).

- **Table 7:** PD risk (`pd12_month_100`, `slope_100`, `dtd`)
- **Table 8:** systemic risk (MES, NSRISK, CoVaR at 1% and 5%)
- **Table 9:** network risk

### Table 6: treated-only sample

On the `A_B` sample, each risk measure is regressed on `post_inf_dummy` (firm + year FE) and separately on `median_ruled_dummy` (firm + quarter FE). A second version uses quarter-specific treatment and is plotted in `Regressions_with_coefplots.do`.

### Instrumental variables

`ivregress 2sls`, with `treat_dummy` treated as endogenous and instrumented by lagged debt-to-equity, lagged debt-to-capital, lagged capital-flow beta and lagged deposits-to-assets. Controls: post, large, interactions, macro factors, firm FE. Robust standard errors. First stages come from `ivregress2`; endogeneity and over-identification tests from `estat endogenous` and `estat overid`. The checklist script repeats this with year-specific treatment on `A_B`, including a version that follows the "forbidden regression" rules for a binary endogenous variable.

### Heckman selection (Table 10)

Two-step / ML `heckman` models where the outcome equation is the baseline DiD and the selection equation explains `treat_dummy` (or `pub_yr_treat` in Checklist 7). The inverse Mills ratio and its standard error are reported.

| Version | Selection-equation variables |
|---|---|
| V1 | Lagged debt/equity, debt/capital, deposits/assets, CF beta, policy beta |
| V2 | Lagged debt/equity, debt/capital, deposits/assets, then adding CF beta or policy beta separately |
| V3 | As above, without robust standard errors |
| Checklist 7 | IV1: lagged leverage, coverage and Tier 1 ratio; IV2: lagged CF and policy betas; IV3: both; all with year and firm FE |

### Stress periods (Table 13)

- **V1:** baseline DiD estimated separately within each stress year (2011, 2016, 2018).
- **V2:** pooled stress dummy interacted with post, treatment and large infusions.
- **V3:** separate interactions for each stress year.

### Channels (Table 14)

Treated banks are split by each `hi_*` characteristic (high vs low). Each half is compared with the control group (`A_C`, `A_D`) with firm + year FE, using a large-infusion dummy re-defined within each split.

### Aggregate risk spreads (Table 15)

Aggregate risk spreads regressed on post-infusion dummies (`Post_Inf_Dum1`, `Post_Inf_Dum3`, each covering the infusion period and two following periods) with year FE and robust standard errors. Checklist 6 repeats this in first differences.

### Scaled infusion size (Table 16)

Baseline DiD with the large-infusion interaction replaced by infusions scaled by total assets, deposits or Tier 1 capital (`A_C`, `A_D`), with no FE and with quarter FE. Checklist 8 re-runs these with network centrality (`Degree`, `BetCent`) as the outcome (labelled Table 12).

### Placebo tests

Baseline DiD with the real post-infusion dummy replaced by random placebo quarters, and by infusion dates shifted one and two quarters earlier.

### Quarter-specific timing (`Qtr_spec_Analysis.do`)

Risk regressed on `Qtr_post_inf`, the large-infusion definitions (`Large_inf_MH`, `Large_inf_8Q`, `CurrentQ_Large_inf`) and their interactions with `Qtr_post_inf` and `Qtr_post_pre`, with firm FE and year-quarter FE.

### Descriptive t-tests

Welch t-tests of every bank financial between treated banks and each control group, exported with `esttab` to `ttest.rtf`.

---

## Output tables

All tables are written with `outreg2` (Excel) or `esttab` (RTF) to `Results/`.

| Table | Content | Script |
|---|---|---|
| 6 | Treated-only regressions on post and large infusions; coefficient plot | `Risk_Analysis.do`, `Regressions_with_coefplots.do` |
| 7 | PD risk DiD | `Risk_Analysis.do`, `Qtr_spec_Analysis.do`, checklist scripts |
| 8 | Systemic risk DiD | same as above |
| 9 | Network risk DiD | same as above |
| 10 | Heckman selection-corrected regressions | `Risk_Heckman.do`, `Checklist_7.do` |
| 12 | Network centrality with scaled infusions | `Risk_Checklist_-related_Analysis.do` |
| 13 | Macro stress periods | `Risk_Analysis.do` |
| 14 | Channels (bank characteristic splits) | `Risk_Analysis.do` |
| 15 | Aggregate risk spreads | `Agg_Risk_Measures.do`, checklist script |
| 16 | Scaled infusion size | `Risk_Analysis.do` |
| — | IV regressions (first and second stage) | `Risk_Analysis.do`, checklist script |
| — | Placebo regressions | `Risk_Analysis.do` |
| — | t-tests of bank financials | `Risk_Analysis.do` |

---

