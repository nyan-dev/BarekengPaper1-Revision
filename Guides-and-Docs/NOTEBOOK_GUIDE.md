# Notebook Guide

Maps each notebook in the rebuilt repository to its manuscript section, inputs/outputs, and key parameters. Every manuscript figure, table, or equation must trace back to exactly one notebook here — no orphaned outputs, no duplicated logic across notebooks.

---

## 01_data_and_eda.ipynb

- **Manuscript link:** §2.1 (Dataset and Features), Figure 1, Figure 2
- **Purpose:** Load raw Kaggle CSV, inspect schema, select the 7 numerical features, produce distribution plot (Fig 1) and correlation heatmap (Fig 2).
- **Inputs:** `data/healthcare_providers_raw.csv` (Kaggle: `tamilsel/healthcare-providers-data`)
- **Outputs:** `outputs/figures/fig1_distribution.png`, `outputs/figures/fig2_correlation_heatmap.png`, `data/processed/features_raw.parquet`
- **Key parameters:** feature subset list (documented explicitly, not inferred at runtime)
- **Status:** Port from original `medical_fraud_analysis_analytics.ipynb`, verify identical feature selection.

## 02_baseline_models.ipynb

- **Manuscript link:** §2.2 (Preprocessing), §2.3 (IF & LOF), Table 1, Figure 4, Figure 5
- **Purpose:** StandardScaler preprocessing (Eq. 1), fit Isolation Forest (contamination=0.05) and LOF (auto + contamination=0.05), produce anomaly counts (Table 1) and scatter plots (Fig 4–5).
- **Inputs:** `data/processed/features_raw.parquet`
- **Outputs:** `data/processed/anomaly_labels.parquet` (IF, LOF-auto, LOF-0.05 flags per record), `outputs/figures/fig4_if_scatter.png`, `outputs/figures/fig5_lof_scatter.png`, `outputs/tables/table1_anomaly_counts.csv`
- **Key parameters (must match original exactly):** `random_state` (log exact value in `REPRODUCIBILITY.md`), IF `contamination=0.05`, LOF `n_neighbors` (log exact value), LOF `contamination='auto'` and `0.05`
- **Status:** Port from original, run parity check against original Table 1 values before proceeding to any new notebook.

## 03_parameter_sensitivity.ipynb

- **Manuscript link:** New subsection in §3.1, responds to Editor Comment C2
- **Purpose:** Consolidate and extend the original `Tuning/isolation_forest_tuning.ipynb` and `Tuning/LOF.ipynb`. Sweep IF over `n_estimators` and `contamination`; sweep LOF over `n_neighbors` and `contamination`. Report anomaly count stability and pairwise Jaccard Index across configurations to demonstrate results are not artifacts of a single parameter choice.
- **Inputs:** `data/processed/features_raw.parquet`
- **Outputs:** `outputs/tables/table_sensitivity_if.csv`, `outputs/tables/table_sensitivity_lof.csv`, `outputs/figures/fig_sensitivity_heatmap.png`
- **Key parameters:** grid ranges must be explicitly stated and justified (e.g., "contamination tested at 0.01, 0.03, 0.05, 0.07, 0.10 to bracket plausible fraud prevalence rates cited in [X]")
- **Status:** New notebook, reuses existing tuning code as a starting point. Do not port OCSVM tuning — out of scope (see PRD §3).

## 04_validation_synthetic_and_expert.ipynb

- **Manuscript link:** New subsection in §3.1/§3.2, responds to Editor Comment C1
- **Purpose:** Two-part validation without external datasets:
  1. **Synthetic injection:** inject a known count/percentage of synthetic extreme-value anomalies into a held-out clean subset; rerun IF/LOF; report precision/recall/F1 against the known injected set.
  2. **Expert face-validity table:** take the 545-record IF∩LOF overlap set (from `05_comparative_analysis.ipynb`) and annotate the top 10–15 highest-confidence anomalies with a qualitative justification (e.g., feature values relative to distribution, provider-type context).
- **Inputs:** `data/processed/features_raw.parquet`, `data/processed/anomaly_labels.parquet`
- **Outputs:** `outputs/tables/table_synthetic_validation.csv`, `outputs/tables/table_expert_review.csv`
- **Key parameters:** injection rate/method (fixed seed, documented), number of records reviewed for face validity
- **Status:** New notebook. Depends on `05_comparative_analysis.ipynb` for the overlap set — build after that one, or compute overlap independently here.

## 05_comparative_analysis.ipynb

- **Manuscript link:** §3.2 (Visualization), §3.3 (Comparative Analysis), Table 2, Figure 6, Figure 7
- **Purpose:** Compute Jaccard Index (Eq. 5) across model pairs, produce Venn diagrams (Fig 6) and comparison heatmap (Fig 7), output overlap counts (Table 2).
- **Inputs:** `data/processed/anomaly_labels.parquet`
- **Outputs:** `outputs/tables/table2_jaccard_overlap.csv`, `outputs/figures/fig6_venn.png`, `outputs/figures/fig7_heatmap.png`
- **Key parameters:** none beyond upstream label sets
- **Status:** Port from original, verify Table 2 values match (545 overlap, Jaccard 0.058, etc.) before adding anything new.

## 06_statistical_tests_effect_sizes.ipynb

- **Manuscript link:** §3.4, Table 3, Table 4, responds to Editor Comment C3
- **Purpose:** Run Mann-Whitney U (numerical features) and Chi-square (categorical features) as in the original, then **add** rank-biserial correlation for Mann-Whitney results and Cramér's V for Chi-square results to quantify effect magnitude, not just significance.
- **Inputs:** `data/processed/features_raw.parquet`, `data/processed/anomaly_labels.parquet`
- **Outputs:** `outputs/tables/table3_mannwhitney_effectsize.csv`, `outputs/tables/table4_chisquare_effectsize.csv`
- **Key parameters:** none beyond feature/group definitions from original
- **Status:** Port original test logic, extend with effect-size computation only — do not change the underlying test methodology.

## 07_manuscript_export.ipynb

- **Manuscript link:** All figures/tables, final assembly
- **Purpose:** Single notebook that re-imports outputs from notebooks 01–06 and regenerates every manuscript-ready figure/table in final formatting (captions, consistent styling, resolution) for direct embedding in the revised manuscript.
- **Inputs:** All `outputs/tables/*.csv` and `outputs/figures/*.png` from notebooks 01–06
- **Outputs:** `outputs/manuscript_ready/` (final versions of every figure and table)
- **Key parameters:** none — this notebook does not compute anything new, only formats and consolidates
- **Status:** New notebook, purely for traceability and camera-ready packaging.

---

## Execution Order

Notebooks must be run in numeric order (01 → 07) since each depends on outputs from prior notebooks via the `data/processed/` and `outputs/` directories. No notebook should recompute logic that belongs in an earlier one.
