# Reproducibility Record

This file is the single source of truth for anything that could cause computed results to differ between the original submission and the rebuilt repository. Any value changed here must be logged with a reason and date.

## 1. Data Provenance

- **Source:** Kaggle — "Healthcare Providers Data For Anomaly Detection", `tamilsel/healthcare-providers-data`
- **Records:** 100,000, 27 columns (7 numerical features used)
- **Ground truth:** None. Dataset is explicitly unsupervised/unlabeled per its own Kaggle description. This is a disclosed limitation, not a gap to silently fix.
- **Local copy checksum:** _[fill in SHA256 of the CSV once downloaded, so the exact data snapshot is verifiable]_

## 2. Environment

Pin exact versions in `requirements.txt`. Record them here as well for quick reference:

| Package | Version (original, if known) | Version (rebuilt) |
|---|---|---|
| python | _fill in_ | _fill in_ |
| scikit-learn | _fill in_ | _fill in_ |
| pandas | _fill in_ | _fill in_ |
| numpy | _fill in_ | _fill in_ |
| scipy | _fill in_ | _fill in_ |
| matplotlib | _fill in_ | _fill in_ |
| seaborn | _fill in_ | _fill in_ |
| matplotlib-venn | _fill in_ | _fill in_ |

> Action: extract original versions from the archived repo's `requirements.txt` before writing the new one. If the original repo has no pinned versions, note that explicitly here as a known original-submission limitation.

## 3. Random Seeds

| Component | Parameter | Value | Notes |
|---|---|---|---|
| Isolation Forest | `random_state` | **42** | Confirmed from Cell 5 of original notebook |
| Synthetic injection (new, notebook 04) | seed | _to be fixed once defined_ | New — document at creation |
| Train/test or holdout splits (if any) | N/A | No train/test split used | LOF and IF both run on full 100,000-record dataset in batch mode (`novelty=False`) |

LOF has no stochastic component — deterministic given identical inputs and `n_neighbors`.

## 4. Fixed Model Parameters (must match original exactly in notebooks 01–02)

| Model | Parameter | Value | Notes |
|---|---|---|---|
| Isolation Forest | `contamination` | `0.05` | |
| Isolation Forest | `n_estimators` | **100** | Confirmed from Cell 5 of original notebook |
| Isolation Forest | `random_state` | **42** | |
| Isolation Forest | input data | **`X_scaled`** | ⚠️ IF was fitted on StandardScaler output, NOT raw `df_num`. Scale-invariant in theory but must be reproduced exactly for parity. |
| LOF | `contamination` | `'auto'` and `0.05` (both reported) | Two separate LOF runs |
| LOF | `n_neighbors` | **20** | Confirmed from Cells 10, 31, 33 of original notebook |
| LOF | `novelty` | **`False`** (default) | Batch `fit_predict` on full 100k records; do NOT use `novelty=True` |
| LOF | input data | **`X_scaled`** | Same StandardScaler output as IF |
| StandardScaler | `fillna` before fit | `df_num.fillna(df_num.median())` | Precautionary — confirmed 7 features had zero missing values; no imputation actually occurs |

## 5. Parity Check Log

Before any new analysis (notebooks 03–07) is trusted, notebooks 01–02 and 05–06 must reproduce these original submitted values exactly:

| Metric | Original value | Rebuilt value | Match? | Date checked |
|---|---|---|---|---|
| IF anomaly count | 5,000 (5.00%) | | | |
| LOF (auto) anomaly count | 2,565 (2.56%) | | | |
| LOF (0.05) anomaly count | 5,000 (5.00%) | | | |
| IF vs LOF(auto) overlap | 341 | | | |
| IF vs LOF(0.05) overlap | 545 | | | |
| IF vs LOF(0.05) Jaccard | 0.058 | | | |
| LOF(auto) vs LOF(0.05) Jaccard | 0.513 | | | |
| Mann-Whitney U (Anomaly vs Normal, Number of Services) | 5.10e+08, p<.001 | | | |
| Chi-square (Anomaly vs Normal, Provider Type) | 7,496.33, p<.001 | | | |

**Do not proceed to notebooks 03–07 (or to rewriting Methods/Results text) until every row above is marked "Match: Yes."** If any value does not match, investigate and document the cause below before continuing.

## 6. Discrepancy Log

Record any case where a rebuilt number differs from the original, however small, and the resolution:

| Date | Metric | Original | Rebuilt | Cause | Resolution |
|---|---|---|---|---|---|
| | | | | | |

## 7. New Analysis Seeds (Notebooks 03–04, added for this revision)

| Notebook | Component | Seed/parameter | Value |
|---|---|---|---|
| 03 | Parameter grid | contamination range | _fill in once decided_ |
| 03 | Parameter grid | n_estimators range | _fill in once decided_ |
| 04 | Synthetic injection | injection rate | _fill in once decided_ |
| 04 | Synthetic injection | random seed | _fill in once decided_ |
