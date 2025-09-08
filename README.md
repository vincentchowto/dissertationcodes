# dissertationcodes
Codes for the dissertation of 'Analysing Political Risk Regimes: Identifying Systematic Patterns and Shifts in Country Risk Measures Over Time'
# ICRG Monthly Data Analysis Pipeline — README

This toolkit provides a structured workflow to transform ICRG Monthly data into standardised monthly datasets, constructing aggregated risk indicators, applying Hidden Markov Models (HMMs) and K-means Clustering to identify underlying risk regimes, and generating visualisations and summary tables to support the analysis.

---

## 1) What you need

* **Python** 3.9–3.12
* Put the monthly ICRG Excel file (e.g., `ICRG_T3Ba-1.xlsx`) in the same folder.
* Required packages:

```bash
pip install pandas openpyxl numpy matplotlib hmmlearn scikit-learn seaborn
```

**Workbook layout assumption**

* Sheets: `GovStab`, `SocioEcoCond`, `InvProf`, `IntConf`, `ExtConf`
* Row 6 has country names; data start on row 7.

---

## 2) What the steps do

**A. Wide → Long**
Reads the Excel, cleans dates/country names, and outputs a neat file.

* Output: `ICRG_long_format.csv`

**B. Select & Composites**
Keeps study countries + window and builds aggregated monthly indicators.

* Outputs: `ICRG_monthly_by_country_selected.csv`, `ICRG_monthly_aggregated_selected.csv`

**C. HMM **

* `hmmlearn` version (standard library) — per‑country regimes + plots + segments.
* Pure NumPy version — same idea, also writes a monthly states file + volatility stats.
* Outputs:

  * `icrg_hmm_plots/` (PNG per country)
  * `icrg_hmm_segments.csv` (start/end by regime)
  * `icrg_hmm_states_monthly.csv` (monthly state per country)
  * `icrg_hmm_state_volatility.csv` & `_rolling12.csv`

**D. K-means Clustering**
Cluster countries by their decoded regime paths and MeanState.

* Outputs: `country_clusters_with_risk.csv`, `appendix_cluster_summary.csv`, `cluster_risk_list.txt`

**E. Global regimes plot**
A single figure showing average monthly regime of all countries with a few known events shaded.

* Output: `hmm_regime_switching_volatility.png`

---

## 3) How to run (quick version)

Copy each code block into its own file and run in order (or run cells in a notebook).

```
# A) Wide → Long
python step_A_to_long.py

# B) Select countries & composites
python step_B_build_composites.py

# C1) HMM (hmmlearn)
python step_D_hmm_hmmlearn.py

# C2) HMM (pure NumPy) + monthly states & volatility
python step_E_hmm_numpy.py

# D) K-means clustering
python step_F_clustering.py

# E) Global regimes chart
python step_G_global_plot.py
```
---

## 4) Files you will get

| File / Folder                             | Meaning                                  |
| ----------------------------------------- | ---------------------------------------- |
| `ICRG_long_format.csv`                    | Tidy panel (`Date, Country, indicators`) |
| `ICRG_monthly_by_country_selected.csv`    | Country‑month indicators + composites    |
| `ICRG_monthly_aggregated_selected.csv`    | Cross‑country monthly average            |
| `icrg_hmm_plots/`                         | Regime plot per country                  |
| `icrg_hmm_segments.csv`                   | Continuous regime segments               |
| `icrg_hmm_states_monthly.csv`             | Monthly decoded states + change flag     |
| `icrg_hmm_state_volatility.csv`           | Annual switch counts & rates             |
| `icrg_hmm_state_volatility_rolling12.csv` | Rolling 12‑month switch rate             |
| `country_clusters_with_risk.csv`          | K‑means cluster per country + risk label |
| `appendix_cluster_summary.csv`            | Cluster stats and members                |
| `cluster_risk_list.txt`                   | Country lists by risk level              |
| `hmm_regime_switching_volatility.png`     | Global switching figure                  |

---

## 5) Common edits (where to change things)

* **Countries & aliases** → in Steps A & B (`SELECTED_COUNTRIES`, `ALIASES`)
* **Date window** → Step B (`mask_t`)
* **Composite used by HMM** → choose `AllIndicators_Mean`
* **Number of HMM states** → `N_STATES` (default 3)
* **Smoothing of plots** → `SMOOTH_WINDOW` (odd number, e.g., 3 = smoother, 1 = off).
* **Event windows** on the global plot → edit the `events = [...]` list.
* **Clusters** → `N_CLUSTERS` (default 3)

---

## 6) Quick troubleshooting

* **Excel read fails** → check sheet names; ensure `openpyxl` is installed.
* **Dates look wrong** → adjust `parse_to_month()` patterns in Step A.
* **Missing columns** → confirm the CSV file has `Date`, `Country`, and either `ICRG_composite` or the five indicators.
* **Flat series / zero variance** → the code protects with `VAR_FLOOR`, but you can drop constant series from the composite.
* **Too few months** → countries with < `MIN_MONTHS` are skipped.

---

## 7) One‑paragraph methods blurb

Five ICRG components (Government Stability, Socioeconomic Conditions, Investment Profile, Internal Conflict, External Conflict) are combined into aggregated monthly risk indices per country (simple mean by default). A 1D Gaussian HMM with `K=3` regimes is fit to the composite, order regimes by their mean levels, and summarises regime segments, switching rates, and cross‑country co‑movement via K‑means on decoded states.


