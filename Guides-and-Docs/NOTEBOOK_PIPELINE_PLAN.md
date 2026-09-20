# Notebook Pipeline Plan — BAREKENG Manuscript Revision (MS 21575)

> **Read this entire document before generating any notebook.**  
> Notebooks are generated and run **one at a time, in order**. NB02 is only created after NB01's JSON handshake file has been verified. The JSON handshake is the machine-readable proof that upstream outputs are valid.

---

## 1. Project Context

| Item | Value |
|---|---|
| **Paper** | Anomaly Detection using Isolation Forest and LOF on Medicare provider data |
| **Journal** | BAREKENG (MS 21575) — Under revision |
| **Goal** | Fix 9 bugs from the monolithic original notebook, preserve all accepted manuscript numbers exactly, and add new analyses for Editor Comments C1, C2, C3 |
| **Original notebook** | `Old-repo/Anomaly_Detection_Isolation_Forest_Explained (4).ipynb` (monolithic, 75 cells, DO NOT MODIFY) |
| **Ground truth numbers** | 100,000 records, 7 features, 5,000 IF anomalies (contamination=0.05), 5,000 LOF anomalies (contamination=0.05), 545 IF∩LOF overlap, Jaccard = 0.058 |

---

## 2. Folder Architecture

The folder structure below **must exist on Google Drive** (created by NB01 Cell 01). Local execution mirrors the same relative paths from the project root.

```text
BarekengPaper1-Revision/            ← Project root
├── data/
│   ├── raw/
│   │   └── healthcare_providers.csv  ← Immutable. DO NOT modify.
│   ├── interim/                      ← (reserved for future use)
│   └── processed/
│       ├── features_raw.parquet      ← Written by NB01, read by NB02–NB06
│       └── anomaly_labels.parquet    ← Written by NB02, read by NB04–NB06
├── outputs/
│   ├── figures/                      ← All .png exports
│   ├── tables/                       ← All .csv exports
│   ├── models/                       ← (reserved for future use)
│   ├── notebook_exports/             ← JSON handshake files (one per notebook)
│   └── manuscript_ready/             ← Written by NB07 only
└── notebooks/
    ├── 01_data_and_eda.ipynb
    ├── 02_baseline_models.ipynb
    ├── 03_parameter_sensitivity.ipynb
    ├── 04_validation_synthetic_and_expert.ipynb
    ├── 05_comparative_analysis.ipynb
    ├── 06_statistical_tests_effect_sizes.ipynb
    └── 07_manuscript_export.ipynb
```

---

## 3. Dual-Environment Path Protocol

**Every notebook's Cell 01** must implement this exact dual-path detection:

```python
# Cell 01: Mount Storage & Bootstrap Paths
import os, sys
from pathlib import Path

if 'google.colab' in sys.modules:
    from google.colab import drive
    drive.mount('/content/drive')
    BASE_DIR = Path('/content/drive/MyDrive/BarekengPaper1-Revision')
else:
    # Local: assumes notebook is run from the notebooks/ subdirectory
    BASE_DIR = Path.cwd().parent if Path.cwd().name == 'notebooks' else Path.cwd()

# Idempotent folder bootstrap
for folder in ['data/raw', 'data/interim', 'data/processed',
               'outputs/figures', 'outputs/tables', 'outputs/models',
               'outputs/notebook_exports', 'outputs/manuscript_ready']:
    (BASE_DIR / folder).mkdir(parents=True, exist_ok=True)

print(f"Base directory set to: {BASE_DIR}")
```

> **Rule:** The variable `BASE_DIR` must be used for every file read/write throughout the notebook. Never hardcode paths.

---

## 4. JSON Handshake Protocol

Each notebook writes a `summary_NB{N}.json` file to `outputs/notebook_exports/` as its **final cell**. The next notebook reads this file as its **Cell 03 (Upstream Handshake Validation)** before doing anything else.

### 4.1 Schema — What every handshake file must contain

```json
{
  "notebook": "NB01",
  "status": "SUCCESS",
  "timestamp": "2026-09-20T23:00:00",
  "outputs": {
    "key_output_name": "relative/path/to/output.parquet"
  },
  "assertions": {
    "description_of_check": true
  }
}
```

- `"status"` must be `"SUCCESS"`. Any other value blocks the next notebook.
- `"assertions"` records the result of every parity check run inside that notebook.

### 4.2 How the next notebook validates it

```python
# Cell 03: Upstream Handshake Validation (in every NB except NB01)
import json

handshake_path = BASE_DIR / 'outputs' / 'notebook_exports' / 'summary_NB01.json'
assert handshake_path.exists(), f"Handshake not found: {handshake_path}. Run NB01 first."

with open(handshake_path) as f:
    summary = json.load(f)

assert summary['status'] == 'SUCCESS', f"NB01 did not complete successfully: {summary}"
print(f"Upstream handshake OK — NB01 completed at {summary['timestamp']}")
```

---

## 5. Global Conventions (apply to all notebooks)

| Convention | Rule |
|---|---|
| **Cell headers** | Every code cell begins with `# Cell NN: <Short description>` |
| **Encoding** | All anomaly labels use scikit-learn native: `-1 = Anomaly`, `+1 = Normal` |
| **Figure DPI** | `dpi=300`, `bbox_inches='tight'`, saved as `.png` |
| **Seaborn style** | `plt.style.use('default')` with `rcParams` font/size overrides in Cell 02 |
| **Randomness** | `random_state=42` for all stochastic models. `np.random.seed(42)` in notebooks with synthetic generation |
| **Missing values** | Use `pd.to_numeric(..., errors='coerce')` when loading. Apply `fillna(median)` as precaution, but assert or log the actual count |
| **Assertions** | Parity assertions are mandatory for numbers that appear in the manuscript. Failures must halt the notebook |

---

## 6. Core Parameters (must never be changed in NB01–NB02)

| Parameter | Value | Notebook |
|---|---|---|
| `n_estimators` | 100 | NB02 (IF) |
| `contamination` (IF) | 0.05 | NB02 |
| `contamination` (LOF) | `'auto'` and `0.05` | NB02 |
| `n_neighbors` (LOF) | 20 | NB02 |
| `random_state` | 42 | NB02 (IF only; LOF is deterministic) |
| Scaler | `StandardScaler` applied before **both** IF and LOF | NB02 |
| Expected IF anomalies | **5,000** | NB02 |
| Expected LOF-0.05 anomalies | **5,000** | NB02 |
| Expected IF∩LOF overlap | **545** | NB05 |
| Expected Jaccard (IF vs LOF-0.05) | **0.058** | NB05 |

---

## 7. The 7 Numerical Features

These exact column names must be used throughout the pipeline. Any typo will cause a KeyError.

```python
FEATURES = [
    'Number of Services',
    'Number of Medicare Beneficiaries',
    'Number of Distinct Medicare Beneficiary/Per Day Services',
    'Average Medicare Allowed Amount',
    'Average Submitted Charge Amount',
    'Average Medicare Payment Amount',
    'Average Medicare Standardized Amount'
]
```

---

## 8. Notebook Specifications

---

### NB01 — `01_data_and_eda.ipynb`

**Status:** ✅ Exists and has been run on Google Colab. Outputs verified.  
**Manuscript sections:** §2.1, Figure 1, Figure 2  
**Reviewer comments addressed:** None (baseline)  
**Bugs fixed:** BUG-08 (no duplicate cells), BUG-09 (no AI prose cells), DOC-05 (logs missing value counts)

#### Inputs
| Source | Path |
|---|---|
| Raw dataset | `data/raw/healthcare_providers.csv` |

#### Outputs
| Artifact | Path |
|---|---|
| Features matrix | `data/processed/features_raw.parquet` — shape `(100000, 7)` |
| Distribution plots | `outputs/figures/fig1_distribution.png` |
| Correlation heatmap | `outputs/figures/fig2_correlation_heatmap.png` |
| JSON handshake | `outputs/notebook_exports/summary_NB01.json` ← **STILL MISSING — must be added** |

#### Cell-by-Cell Specification

| Cell | Type | Purpose | Key code |
|---|---|---|---|
| 00 | Markdown | Title, I/O contract | Header + bullet lists |
| 01 | Code | Mount Drive, bootstrap folders | See §3 template |
| 02 | Code | Imports + `rcParams` styling | `pandas`, `numpy`, `matplotlib`, `seaborn` |
| 03 | Code | Load CSV, assert shape `(100000, *)` | `pd.read_csv(raw_data_path, low_memory=False)` |
| 04 | Code | Extract 7 features, `pd.to_numeric(..., errors='coerce')`, log NaN counts | `df_features = df_raw[FEATURES].copy()` |
| 05 | Code | Figure 1: 3×3 histplot grid | `sns.histplot`, save to `fig1_distribution.png` |
| 06 | Code | Figure 2: lower-triangle heatmap | `sns.heatmap` with `mask`, save to `fig2_correlation_heatmap.png` |
| 07 | Code | Export `features_raw.parquet` | `df_features.to_parquet(...)` |
| **08** | **Code** | **Write `summary_NB01.json`** | **See handshake template in §4.1** |

#### NB01 Handshake File — `summary_NB01.json`
```json
{
  "notebook": "NB01",
  "status": "SUCCESS",
  "timestamp": "<runtime ISO timestamp>",
  "outputs": {
    "features_raw_parquet": "data/processed/features_raw.parquet",
    "fig1": "outputs/figures/fig1_distribution.png",
    "fig2": "outputs/figures/fig2_correlation_heatmap.png"
  },
  "assertions": {
    "raw_row_count_is_100000": true,
    "feature_shape_is_100000x7": true
  }
}
```

#### Known Corrections Applied (from Colab run)
- `BASE_DIR` uses `BarekengPaper1-Revision` (not `Paper1_Revision`)
- `pd.to_numeric(..., errors='coerce')` added in Cell 04 to handle mixed-type columns
- The strict `assert missing_counts.sum() == 0` was softened — just print, don't halt, because some column values may have non-numeric strings that become NaN after coercion
- `fillna(median)` applied as a precautionary step after logging

---

### NB02 — `02_baseline_models.ipynb`

**Status:** 🔴 Not yet generated. Generate only after `summary_NB01.json` exists.  
**Manuscript sections:** §2.2, §2.3, Table 1, Figure 4, Figure 5  
**Reviewer comments addressed:** None (baseline parity)  
**Bugs fixed:** BUG-01 (LOF on all 7 features), BUG-02 (no contamination=0.01), BUG-05 (document StandardScaler before IF), BUG-06 (canonical column names), BUG-07 (-1/+1 encoding)

#### Inputs
| Source | Path |
|---|---|
| Handshake | `outputs/notebook_exports/summary_NB01.json` |
| Features | `data/processed/features_raw.parquet` |

#### Outputs
| Artifact | Path |
|---|---|
| Anomaly labels | `data/processed/anomaly_labels.parquet` — columns: `IF_Label`, `LOF_Auto_Label`, `LOF_05_Label` |
| IF scatter plot | `outputs/figures/fig4_if_scatter.png` |
| LOF scatter plot | `outputs/figures/fig5_lof_scatter.png` |
| Anomaly counts table | `outputs/tables/table1_anomaly_counts.csv` |
| JSON handshake | `outputs/notebook_exports/summary_NB02.json` |

#### Cell-by-Cell Specification

| Cell | Type | Purpose | Key code |
|---|---|---|---|
| 00 | Markdown | Title, I/O contract, bug-fix notes | |
| 01 | Code | Mount Drive, bootstrap folders | See §3 template |
| 02 | Code | Imports: `sklearn`, `pandas`, `seaborn` | `from sklearn.preprocessing import StandardScaler` etc. |
| 03 | Code | Upstream handshake validation | Read `summary_NB01.json`, assert status=SUCCESS |
| 04 | Code | Load `features_raw.parquet`, assert shape `(100000, 7)` | |
| 05 | Code | `StandardScaler` fit + transform → `X_scaled` | `scaler = StandardScaler(); X_scaled = scaler.fit_transform(df_features)` |
| 06 | Code | Fit IF `(n_estimators=100, contamination=0.05, random_state=42)`, assert `(if_labels == -1).sum() == 5000` | |
| 07 | Code | Fit LOF-Auto `(n_neighbors=20, contamination='auto')`, Fit LOF-05 `(n_neighbors=20, contamination=0.05)`, assert `(lof_05 == -1).sum() == 5000` | Do NOT use contamination=0.01 (BUG-02) |
| 08 | Code | Build `anomaly_labels.parquet` with columns `IF_Label`, `LOF_Auto_Label`, `LOF_05_Label`; assert all values in `{-1, 1}` | |
| 09 | Code | Build `table1_anomaly_counts.csv` with Normal/Anomaly counts per model | `display(df_table1)` |
| 10 | Code | Figure 4: IF scatter plot (color by `IF_Label`) | Axes: `Number of Services` vs `Average Medicare Payment Amount` |
| 11 | Code | Figure 5: LOF-05 scatter plot (color by `LOF_05_Label`) | Same axes as Figure 4 |
| 12 | Code | Export all outputs, write `summary_NB02.json` | |

#### NB02 Handshake File — `summary_NB02.json`
```json
{
  "notebook": "NB02",
  "status": "SUCCESS",
  "timestamp": "<runtime ISO timestamp>",
  "outputs": {
    "anomaly_labels_parquet": "data/processed/anomaly_labels.parquet",
    "table1": "outputs/tables/table1_anomaly_counts.csv",
    "fig4": "outputs/figures/fig4_if_scatter.png",
    "fig5": "outputs/figures/fig5_lof_scatter.png"
  },
  "assertions": {
    "if_anomaly_count_is_5000": true,
    "lof05_anomaly_count_is_5000": true,
    "label_encoding_is_neg1_pos1": true
  }
}
```

---

### NB03 — `03_parameter_sensitivity.ipynb`

**Status:** 🔴 Not yet generated. Generate only after `summary_NB02.json` exists.  
**Manuscript sections:** New subsection in §3.1  
**Reviewer comments addressed:** **C2 — Parameter Justification and Stability**  
**Bugs fixed:** None

#### Inputs
| Source | Path |
|---|---|
| Handshake | `outputs/notebook_exports/summary_NB02.json` |
| Features | `data/processed/features_raw.parquet` |

#### Outputs
| Artifact | Path |
|---|---|
| IF sensitivity table | `outputs/tables/table_sensitivity_if.csv` |
| LOF sensitivity table | `outputs/tables/table_sensitivity_lof.csv` |
| Jaccard stability table | `outputs/tables/table_stability_jaccard.csv` |
| Sensitivity heatmap | `outputs/figures/fig_sensitivity_heatmap.png` |
| JSON handshake | `outputs/notebook_exports/summary_NB03.json` |

#### Parameter Grid

| Model | Parameter | Values |
|---|---|---|
| IF | `contamination` | `[0.01, 0.03, 0.05, 0.07, 0.10]` |
| IF | `n_estimators` | `[50, 100, 200, 300]` |
| LOF | `contamination` | `[0.01, 0.03, 0.05, 0.07, 0.10]` |
| LOF | `n_neighbors` | `[10, 20, 30, 50]` |

All models in this notebook use `random_state=42` (IF) and the same `X_scaled` from NB02's scaler (re-fit from `features_raw.parquet`).

#### Key Computation: Jaccard Stability

```python
def compute_jaccard(labels_a, labels_b):
    a = set(np.where(labels_a == -1)[0])
    b = set(np.where(labels_b == -1)[0])
    intersection = len(a & b)
    union = len(a | b)
    return intersection / union if union else 1.0

# Compare each config against the manuscript default (n=100, c=0.05)
```

#### Cell-by-Cell Specification

| Cell | Type | Purpose |
|---|---|---|
| 00 | Markdown | Title, I/O contract, rationale for C2 |
| 01 | Code | Mount Drive, bootstrap folders |
| 02 | Code | Imports (`itertools`, `sklearn`) |
| 03 | Code | Upstream handshake validation (NB02) |
| 04 | Code | Load features, re-fit StandardScaler, compute `X_scaled` |
| 05 | Code | Define parameter grids |
| 06 | Code | IF sensitivity sweep (nested loop over `n_estimators` × `contamination`) |
| 07 | Code | LOF sensitivity sweep (nested loop over `n_neighbors` × `contamination`) |
| 08 | Code | Compute pairwise Jaccard for each config vs manuscript default |
| 09 | Code | Plot heatmaps (IF count heatmap + LOF count heatmap) |
| 10 | Code | Export all tables, write `summary_NB03.json` |

---

### NB04 — `04_validation_synthetic_and_expert.ipynb`

**Status:** 🔴 Not yet generated. Generate only after `summary_NB02.json` exists (NB04 depends on `anomaly_labels.parquet`).  
**Manuscript sections:** New subsection in §3.1/§3.2  
**Reviewer comments addressed:** **C1 — Model Validation**  
**Bugs fixed:** None

#### Inputs
| Source | Path |
|---|---|
| Handshake | `outputs/notebook_exports/summary_NB02.json` |
| Features | `data/processed/features_raw.parquet` |
| Labels | `data/processed/anomaly_labels.parquet` |

#### Outputs
| Artifact | Path |
|---|---|
| Synthetic validation table | `outputs/tables/table_synthetic_validation.csv` |
| Expert review CSV template | `outputs/tables/table_expert_review_template.csv` — **must be filled in manually by author before NB07** |
| JSON handshake | `outputs/notebook_exports/summary_NB04.json` |

#### Part 1: Synthetic Injection Method

1. Define a "clean" subset: records where **both** `IF_Label == 1` AND `LOF_05_Label == 1`.
2. Sample 10,000 records from that clean subset (`random_state=42`).
3. Inject 500 synthetic anomalies (5% contamination): randomly select 500 indices, then randomly perturb ≥1 feature per selected record by `Uniform(3, 5) × std_dev` in a random direction. Clip all values at 0 (no negative amounts/counts). `np.random.seed(42)`.
4. Run IF and LOF with manuscript parameters on the 10,000-record validation set.
5. Compute Precision, Recall, F1 against ground-truth injection labels.

#### Part 2: Expert Face-Validity Method

1. Identify `IF_Label == -1` AND `LOF_05_Label == -1` — the 545-record overlap set. Assert count = 545.
2. Sort by `Average Medicare Payment Amount` descending, take top 15.
3. Export to `table_expert_review_template.csv` with a blank `Expert_Clinical_Justification` column for manual annotation by the author.

#### Cell-by-Cell Specification

| Cell | Type | Purpose |
|---|---|---|
| 00 | Markdown | Title, I/O contract, rationale for C1 |
| 01 | Code | Mount Drive, bootstrap folders |
| 02 | Code | Imports, `np.random.seed(42)` |
| 03 | Code | Upstream handshake validation (NB02) |
| 04 | Code | Load features and labels |
| 05 | Code | Extract clean subset, sample 10,000 records |
| 06 | Code | Inject 500 synthetic anomalies, build ground-truth labels |
| 07 | Code | Scale validation set, fit IF + LOF, compute Precision/Recall/F1 |
| 08 | Code | Export `table_synthetic_validation.csv` |
| 09 | Markdown | `## Part 2: Expert Face-Validity` |
| 10 | Code | Extract 545-record overlap, assert count=545, sort by payment amount, take top 15 |
| 11 | Code | Export `table_expert_review_template.csv` with blank justification column |
| 12 | Code | Write `summary_NB04.json` |

---

### NB05 — `05_comparative_analysis.ipynb`

**Status:** 🔴 Not yet generated. Generate only after `summary_NB02.json` exists.  
**Manuscript sections:** §3.2, §3.3, Table 2, Figure 6, Figure 7  
**Reviewer comments addressed:** None (baseline)  
**Bugs fixed:** BUG-06 (Venn diagrams now use canonical 7-feature labels, not the 2-feature draft)

#### Inputs
| Source | Path |
|---|---|
| Handshake | `outputs/notebook_exports/summary_NB02.json` |
| Labels | `data/processed/anomaly_labels.parquet` |

#### Outputs
| Artifact | Path |
|---|---|
| Jaccard/overlap table | `outputs/tables/table2_jaccard_overlap.csv` |
| Venn diagrams | `outputs/figures/fig6_venn.png` |
| Jaccard heatmap | `outputs/figures/fig7_heatmap.png` |
| JSON handshake | `outputs/notebook_exports/summary_NB05.json` |

#### Parity Assertions (MANDATORY)

```python
assert overlap_if_lof05 == 545, "..."
assert round(jaccard_if_lof05, 3) == 0.058, "..."
```

#### Venn Diagram Note

Use `matplotlib-venn` library. The notebook should attempt `from matplotlib_venn import venn2, venn3` inside a `try/except` block. If not installed, it should run `subprocess.check_call([sys.executable, "-m", "pip", "install", "matplotlib-venn"])` before importing.

#### Cell-by-Cell Specification

| Cell | Type | Purpose |
|---|---|---|
| 00 | Markdown | Title, I/O contract |
| 01 | Code | Mount Drive, bootstrap folders |
| 02 | Code | Imports (try/except for `matplotlib-venn`) |
| 03 | Code | Upstream handshake validation (NB02) |
| 04 | Code | Load labels, convert to index sets (`if_anomalies = set(df.index[df['IF_Label'] == -1])`) |
| 05 | Code | Compute Jaccard for all 3 pairs, assert parity, build `table2_jaccard_overlap.csv` |
| 06 | Code | Figure 6: Venn diagrams (2-way IF vs LOF-05, and 3-way) |
| 07 | Code | Figure 7: Pairwise Jaccard 3×3 heatmap |
| 08 | Code | Export all outputs, write `summary_NB05.json` |

---

### NB06 — `06_statistical_tests_effect_sizes.ipynb`

**Status:** 🔴 Not yet generated. Generate only after `summary_NB02.json` exists.  
**Manuscript sections:** §3.4, Table 3, Table 4  
**Reviewer comments addressed:** **C3 — Effect Size / Practical Significance**  
**Bugs fixed:** BUG-03 (export $U$ and $\chi^2$ statistics, not just p-values), BUG-04 (exact group names matching manuscript)

#### Inputs
| Source | Path |
|---|---|
| Handshake | `outputs/notebook_exports/summary_NB02.json` |
| Raw data | `data/raw/healthcare_providers.csv` (for categorical columns) |
| Labels | `data/processed/anomaly_labels.parquet` |

#### Outputs
| Artifact | Path |
|---|---|
| Mann-Whitney + effect size | `outputs/tables/table3_mannwhitney_effectsize.csv` |
| Chi-square + effect size | `outputs/tables/table4_chisquare_effectsize.csv` |
| JSON handshake | `outputs/notebook_exports/summary_NB06.json` |

#### Comparison Groups (Fixing BUG-04)

| Group name | Definition |
|---|---|
| `Normal` | `IF_Label == 1` AND `LOF_05_Label == 1` |
| `IF Anomalies (All)` | `IF_Label == -1` |
| `LOF Anomalies (All)` | `LOF_05_Label == -1` |
| `IF-Only` | `IF_Label == -1` AND `LOF_05_Label == 1` |
| `LOF-Only` | `IF_Label == 1` AND `LOF_05_Label == -1` |
| `Overlap (Both)` | `IF_Label == -1` AND `LOF_05_Label == -1` |

#### Statistical Methods

**Numerical features (Table 3):**
- Test: Mann-Whitney U (`scipy.stats.mannwhitneyu`, `alternative='two-sided'`)
- Effect size: Rank-Biserial Correlation $r = 1 - \frac{2U}{n_1 \cdot n_2}$
- Must export: `Feature`, `Comparison`, `U_Statistic`, `p_value`, `Rank_Biserial_r`, `Mean_A`, `Mean_B`

**Categorical features (Table 4):**
- Test: Chi-Square (`scipy.stats.chi2_contingency` on contingency table)
- Effect size: Cramér's V $= \sqrt{\frac{\chi^2}{n \cdot \min(r-1, c-1)}}$
- Must export: `Feature`, `Comparison`, `Chi2_Statistic`, `p_value`, `Degrees_of_Freedom`, `Cramers_V`
- Categorical features to test: `Provider Type`, `State Code of the Provider`

#### Cell-by-Cell Specification

| Cell | Type | Purpose |
|---|---|---|
| 00 | Markdown | Title, I/O contract, rationale for C3 |
| 01 | Code | Mount Drive, bootstrap folders |
| 02 | Code | Imports (`scipy.stats`) |
| 03 | Code | Upstream handshake validation (NB02) |
| 04 | Code | Load raw data + labels, merge on index |
| 05 | Code | Define the 6 comparison groups |
| 06 | Markdown | `## 1. Mann-Whitney U & Rank-Biserial Correlation` |
| 07 | Code | Helper function `run_mwu_tests(group_a, group_b, name_a, name_b)` — iterates over 7 numerical features |
| 08 | Code | Run 3 comparisons: (Normal vs IF-All), (Normal vs LOF-All), (IF-Only vs LOF-Only) |
| 09 | Code | Export `table3_mannwhitney_effectsize.csv` |
| 10 | Markdown | `## 2. Chi-Square & Cramér's V` |
| 11 | Code | Helper function `run_chi2_tests(group_a, group_b, name_a, name_b)` |
| 12 | Code | Run same 3 comparisons on categorical features |
| 13 | Code | Export `table4_chisquare_effectsize.csv` |
| 14 | Code | Write `summary_NB06.json` |

---

### NB07 — `07_manuscript_export.ipynb`

**Status:** 🔴 Not yet generated. Generate only after **all** of `summary_NB01` through `summary_NB06` exist.  
**Manuscript sections:** All figures/tables — final assembly  
**Reviewer comments addressed:** All (consolidation)  
**Bugs fixed:** None

#### Inputs
| Source | Path |
|---|---|
| Handshakes | `summary_NB01.json` through `summary_NB06.json` — all must exist and be `"SUCCESS"` |
| All figures | `outputs/figures/*.png` |
| All tables | `outputs/tables/*.csv` |

#### Outputs
| Artifact | Path |
|---|---|
| All submission-ready files | `outputs/manuscript_ready/` |

#### Asset Checklist (NB07 validates this list before copying)

**Figures:**
- `fig1_distribution.png`
- `fig2_correlation_heatmap.png`
- `fig4_if_scatter.png`
- `fig5_lof_scatter.png`
- `fig_sensitivity_heatmap.png`
- `fig6_venn.png`
- `fig7_heatmap.png`

**Tables:**
- `table1_anomaly_counts.csv`
- `table_sensitivity_if.csv`
- `table_sensitivity_lof.csv`
- `table_stability_jaccard.csv`
- `table_synthetic_validation.csv`
- `table_expert_review_template.csv`
- `table2_jaccard_overlap.csv`
- `table3_mannwhitney_effectsize.csv`
- `table4_chisquare_effectsize.csv`

> **Note:** `table_expert_review_template.csv` must be manually filled with `Expert_Clinical_Justification` by the author before NB07 is run.

#### Cell-by-Cell Specification

| Cell | Type | Purpose |
|---|---|---|
| 00 | Markdown | Title, purpose, I/O contract |
| 01 | Code | Mount Drive, bootstrap folders |
| 02 | Code | Imports (`shutil`, `json`) |
| 03 | Code | Validate ALL 6 handshake files exist and are SUCCESS |
| 04 | Code | Check and copy all required figures |
| 05 | Code | Check and copy all required tables |
| 06 | Code | Print final submission checklist |

---

## 9. Execution Order & Dependency Graph

```
NB01 → [summary_NB01.json] → NB02 → [summary_NB02.json] ─┬→ NB03
                                                            ├→ NB04
                                                            ├→ NB05
                                                            └→ NB06
                                                                ↓
                                              All 6 summaries → NB07
```

- **NB03, NB04, NB05, NB06** all depend on `summary_NB02.json` and can technically be run in any order after NB02.
- **NB07** depends on all 6 handshakes being SUCCESS.
- **Never skip a notebook.** Never delete or overwrite a handshake file manually.

---

## 10. Reviewer Comment Cross-Reference

| Reviewer Comment | Addressed In | New Content |
|---|---|---|
| C1 — Model Validation | NB04 | Synthetic precision/recall, expert face-validity table |
| C2 — Parameter Stability | NB03 | Hyperparameter sweep + Jaccard stability table/heatmap |
| C3 — Effect Size | NB06 | Rank-biserial correlation (Table 3), Cramér's V (Table 4) |
| C4 — Abstract | Manual edit only | Add limitations and originality clause |
| C5 — Conclusion | Manual edit only | Restructure into purpose/findings/limitations/future |
| C6 — Reference Recency | Manual edit only | Audit IEEE format; justify 5 foundational pre-2015 citations |

---

## 11. Immediate Next Actions

1. **Add Cell 08 to NB01** — write `summary_NB01.json` to `outputs/notebook_exports/`. Re-run NB01 on Colab to generate this file.
2. **Upload to Google Drive** (if not already): the project root `BarekengPaper1-Revision/` must be in the user's Google Drive `MyDrive` folder with the `data/raw/healthcare_providers.csv` present.
3. **Generate NB02** — only after the `summary_NB01.json` output is confirmed.
4. **Generate NB03–NB06** — only after `summary_NB02.json` is confirmed.
5. **Generate NB07** — only after all handshakes exist and are SUCCESS.
