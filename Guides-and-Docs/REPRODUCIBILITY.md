# Reproducibility Record

This file is the single source of truth for anything that could cause computed results to differ between the original submission and the rebuilt repository. Any value changed here must be logged with a reason and date.

## 1. Data Provenance

- **Source:** Kaggle — "Healthcare Providers Data For Anomaly Detection", `tamilsel/healthcare-providers-data`
- **Records:** 100,000, 27 columns (7 numerical features used)
- **Ground truth:** None. Dataset is explicitly unsupervised/unlabeled per its own Kaggle description. This is a disclosed limitation, not a gap to silently fix.
- **Local copy checksum (SHA-256):** `3ab905111f47b32ab030ed5b106bbaf602961ed04d4a16ff615d2453911b9991`
- **Size:** 23,362,829 bytes
- Recorded 2026-09-21. NB01 Cell 03 recomputes this and **warns** on mismatch — a different snapshot invalidates every parity anchor in §5, since those were verified against this exact file.

## 2. Environment

Pin exact versions in `requirements.txt`. Record them here as well for quick reference:

| Package | Version (original, if known) | Version (parity-verified 2026-09-21) |
|---|---|---|
| python | _unpinned in original_ | 3.9.13 local / **3.13.15 Colab** |
| scikit-learn | _unpinned in original_ | 1.6.1 local / **Colab UNKNOWN — capture in NB02** |
| pandas | _unpinned in original_ | 2.3.3 local / **2.2.3 Colab** |
| numpy | _unpinned in original_ | 2.0.2 local / **2.1.3 Colab** |
| scipy | _unpinned in original_ | **1.13.1** (Tables 3–4 verified) |
| matplotlib | _unpinned in original_ | 3.9.4 local / **3.10.0 Colab** |
| seaborn | _unpinned in original_ | **0.13.2** (both) |
| matplotlib-venn | _unpinned in original_ | _pending NB04_ |

### 2.1 NB01 Colab run, 2026-09-20

Colab resolved a **different environment** from the one parity was demonstrated in (python 3.13.15, numpy 2.1.3, pandas 2.2.3). Every NB01 statistic nevertheless came out byte-identical to the local run — medians, means, skewness, and `max_abs_offdiag_correlation` = 0.9987039748961641 — and the raw-file SHA-256 matched.

⚠️ That agreement is weaker evidence than it looks: descriptive statistics are insensitive to these version differences, whereas **Isolation Forest and LOF are not**. NB01 does not import scikit-learn, so the version that actually governs the parity anchors is still unobserved. **NB02 Cell 02 must record `scikit-learn` and `scipy` before fitting anything** — if LOF(auto) misses 2,565 on Colab, the sklearn version is the first thing to check.

⚠️ **The original repository pinned nothing** — the notebook installs no versions and states none. Full parity for the submitted numbers therefore rests on the combination above, which is the first environment in which they have been demonstrably reproduced.

**Pinning procedure.** Pins are never guessed. The four packages above were pinned from the environment in which parity was demonstrated; the rest stay range-constrained until NB01/NB04/NB06 report what Colab actually resolves, at which point those observed values are written back as `==` pins.

⚠️ **`requirements.txt` constraints were wrong and have been corrected.** They specified `numpy<2.0.0` and `scikit-learn<1.6.0`, but parity was achieved on numpy **2.0.2** and scikit-learn **1.6.1** — both *excluded* by those bounds. Left as written, Colab would have been forced to downgrade away from the one combination known to reproduce the manuscript.

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
| Currency cleaning | strip `$` and `,` **before** `to_numeric` | **mandatory** | ⚠️ See §4.1. Manuscript §2.2: the 7 columns are object-typed "due to formatting characters (e.g., '$', ',')". Stripping must precede coercion. |
| StandardScaler | `fillna` before fit | `df_num.fillna(df_num.median())` | ✅ **Verified no-op 2026-09-21.** Stripping first leaves 0 NaN across all 7 columns, so the median fill changes nothing. The original's claim was correct. The danger was that it was *assumed* rather than *asserted* — which is exactly how rebuild attempt 1 slipped through (§4.1). Keep the call for fidelity, but assert zero-NaN at runtime. |

**Ground truth is the original notebook's cell [33]** (`Old-repo/Anomaly_Detection_Isolation_Forest_Explained (4).ipynb`), together with cell [5] for IF and cell [3] for cleaning.

⚠️ **Trap: the original contains three different LOF configurations.** This table previously cited "Cells 10, 31, 33" for `n_neighbors`, which is true — all three use 20 — but obscures that only one is canonical:

| Cell | Features | Input | contamination | Status |
|---|---|---|---|---|
| [10] | 2 (`Avg Medicare Payment`, `No. of Services`) | `df_num` — **unscaled** | `0.01` | ❌ Early exploration. Do not port. |
| [31] | 7 | `X_scaled` | `0.05` | ❌ Superseded by [33]. |
| **[33]** | **7** | **`X_scaled`** | **`'auto'` and `0.05`** | ✅ **Canonical — produced the published 2,565 / 5,000.** |

Porting from cell [10] would yield an unscaled 2-feature model and miss parity entirely.

### 4.1 Known trap: currency coercion (falsified rebuild attempt 1)

The original notebook's cell [3] cleans correctly:

```python
series.astype(str).str.replace(r'[\$,]', '', regex=True).replace("nan", np.nan).astype(float)
```

The first (Gemini-generated) rebuild dropped the `.str.replace` and called `pd.to_numeric(col, errors='coerce')` directly. Recorded result: **12,962 NaNs**, silently median-imputed.

| Feature | NaN produced |
|---|---|
| Average Submitted Charge Amount | 6,723 |
| Number of Services | 2,653 |
| Number of Distinct Medicare Beneficiary/Per Day Services | 1,500 |
| Average Medicare Allowed Amount | 745 |
| Average Medicare Standardized Amount | 470 |
| Average Medicare Payment Amount | 466 |
| Number of Medicare Beneficiaries | 405 |

These are not missing data. They are the values carrying a thousands separator — i.e. **every value ≥ 1,000**.

**Verified directly against `data/raw/healthcare_providers.csv` on 2026-09-21.** All 7 columns are `object` dtype. The file contains **no `$` characters at all** — commas alone cause the entire failure. The cut is exact:

| Column | Naive NaN | Dropped range | Kept max | Column median |
|---|---|---|---|---|
| Average Submitted Charge Amount | 6,723 | 1,000.00 – 62,694.00 | 999.74 | 146.00 |
| Number of Services | 2,653 | 1,000.00 – 282,739.00 | 999.00 | 43.00 |

Every dropped value is ≥ 1,000.00 and every surviving value is < 1,000 — a clean partition at the thousands separator, not a missingness pattern. Stripping first yields **0 NaN in all 7 columns**, confirming the fill is a no-op on a correct pipeline.

The severity is the point: median imputation rewrites a 282,739-service provider as 43 services, and a \$62,694 charge as \$146. The single most extreme record in the dataset — the one both algorithms exist to find — is replaced by the centre of the distribution. The largest ~6.7% of the distribution is erased before either model is fitted.

**Rule:** NB1 strips `$` and `,`, coerces, then asserts `isna().sum() == 0` and **raises** on failure. No silent imputation under any circumstance. If genuine NaNs ever appear, stop and escalate rather than filling.

## 5. Parity Check Log

Before any new analysis (notebooks 03–07) is trusted, notebooks 01–02 and 05–06 must reproduce these original submitted values exactly:

**Not every row below carries equal weight.** Rows marked *trivial* are `contamination × 100,000` and hold for any input data whatsoever — they would have passed even under the falsified §4.1 pipeline. They confirm the models were configured correctly; they say nothing about whether the data reached them intact. Only the *diagnostic* rows can actually fail.

| Metric | Source NB | Original value | Diagnostic power | Rebuilt value | Match? | Date |
|---|---|---|---|---|---|---|
| IF anomaly count | NB2 | 5,000 (5.00%) | trivial — `0.05 × 100k` | 5,000 | ✅ | 2026-09-21 |
| LOF (0.05) anomaly count | NB2 | 5,000 (5.00%) | trivial — `0.05 × 100k` | 5,000 | ✅ | 2026-09-21 |
| **LOF (auto) anomaly count** | NB2 | **2,565 (2.56%)** | **diagnostic** — data-dependent threshold | **2,565** | ✅ | 2026-09-21 |
| **IF ∩ LOF(auto) overlap** | NB5 | **341** | **diagnostic** | **341** | ✅ | 2026-09-21 |
| **IF ∩ LOF(0.05) overlap** | NB5 | **545** | **diagnostic** | **545** | ✅ | 2026-09-21 |
| LOF(auto) ∩ LOF(0.05) overlap | NB5 | 2,565 | diagnostic; asserts nesting | 2,565 | ✅ | 2026-09-21 |
| IF vs LOF(auto) Jaccard | NB5 | 0.047 | derived from overlap | 0.0472 | ✅ | 2026-09-21 |
| IF vs LOF(0.05) Jaccard | NB5 | 0.058 | derived from overlap | 0.0576 | ✅ | 2026-09-21 |
| LOF(auto) vs LOF(0.05) Jaccard | NB5 | 0.513 | derived; also asserts nesting | 0.5130 | ✅ | 2026-09-21 |
| **Mann-Whitney U** (Anom vs Normal, No. of Services) | NB6 | **5.10e+08**, p<.001 | **diagnostic** | **5.098e+08** | ✅ | 2026-09-21 |
| **Mann-Whitney U** (Anom vs Normal, Avg Medicare Payment) | NB6 | **5.36e+08**, p<.001 | **diagnostic** | **5.361e+08** | ✅ | 2026-09-21 |
| **Mann-Whitney U** (IF-only vs LOF-only, No. of Services) | NB6 | **1.21e+07**, p<.001 | **diagnostic** | **1.208e+07** | ✅ | 2026-09-21 |
| **Mann-Whitney U** (IF-only vs LOF-only, Avg Medicare Payment) | NB6 | **1.46e+07**, p<.001 | **diagnostic** | **1.462e+07** | ✅ | 2026-09-21 |
| **Chi-square** (Anom vs Normal, Provider Type) | NB6 | **7,496.33**, p<.001 | **diagnostic** | **7,496.33** | ✅ | 2026-09-21 |
| **Chi-square** (Anom vs Normal, Entity Type) | NB6 | **1,855.13**, p<.001 | **diagnostic** | **1,855.13** | ✅ | 2026-09-21 |
| **Chi-square** (IF-only vs LOF-only, Provider Type) | NB6 | **2,408.72**, p<.001 | **diagnostic** | **2,408.72** | ✅ | 2026-09-21 |
| **Chi-square** (IF-only vs LOF-only, Entity Type) | NB6 | **157.39**, p<.001 | **diagnostic** | **157.39** | ✅ | 2026-09-21 |

### 5.2 ⚠️ Recovered group definitions (undocumented in the manuscript)

Tables 3–4 report comparisons labelled only "Anomaly vs Normal" and "IF Only vs LOF Only". Neither the manuscript nor any project document defines them. They were recovered empirically on 2026-09-21 by testing every plausible candidate against the published statistics:

| Group | Definition | n |
|---|---|---|
| **Anomaly** | **Union — flagged by IF `OR` LOF(0.05)** | **9,455** |
| Normal | Flagged by neither | 90,545 |
| IF Only | `IF \ LOF(0.05)` | 4,455 |
| LOF Only | `LOF(0.05) \ IF` | 4,455 |

Only the union reproduces. The alternatives are not close — IF-only alone gives U = 3.07e+08 against a published 5.10e+08, and the intersection gives 4.47e+07. This is consistent with the original notebook's cell [63], which describes the grouping as "flagged as anomalous by at least one method (IF or LOF 0.05)".

Note that **LOF(auto) plays no part in Tables 3–4** — the union is built from LOF(0.05). NB6 must use exactly these definitions; a reasonable-looking guess such as "anomaly = IF anomalies" silently destroys parity on all eight statistics.

**✅ PARITY GATE CLOSED — 2026-09-21, before any notebook was written.**

Every published value in Tables 1, 2, 3 and 4 was reproduced locally by porting the original notebook's cells [3], [4], [5] and [33] verbatim against `data/raw/healthcare_providers.csv`. `LOF(auto) ⊆ LOF(0.05)` confirmed True. Zero NaN after cleaning. All eight Tables 3–4 statistics match, including all four χ² values to the reported two decimals.

The baseline is therefore a **verified port, not a reconstruction**. Every notebook in the rebuild transcribes code already demonstrated to reproduce the manuscript. A parity miss in Colab indicates an environment difference, not a modelling error — check versions against §2 first.

**The gate closes at NB6, not NB2.** Four of the six diagnostic rows are produced by NB5 and NB6, so the parity check is not complete until both have run. Build order is therefore **01 → 02 → 05 → 06** (gate closed) **→ 03 → 04 → 07**, not strict numeric order. Filenames keep their manuscript numbering; only the construction sequence differs. This supersedes `CLAUDE.md` Hard Rule 7 for build sequencing — runtime execution order remains as documented in `README.md`.

**Do not build notebooks 03, 04 or 07, and do not rewrite Methods/Results text, until every diagnostic row above reads "Match: Yes."** If any value does not match, investigate and document the cause in §6 before continuing.

### 5.1 Internal consistency of the published values (verified 2026-09-21)

Checked on paper before any code runs — these confirm the targets are self-consistent and therefore trustworthy:

- `341 / (5000 + 2565 − 341)` = 341/7224 = **0.0472** ✓ matches reported 0.047
- `545 / (5000 + 5000 − 545)` = 545/9455 = **0.05764** ✓ matches reported 0.058
- `2565 / (2565 + 5000 − 2565)` = 2565/5000 = **0.513** ✓ matches reported 0.513

The third implies **LOF(auto) ⊂ LOF(0.05)** exactly — the intersection equals all of LOF(auto). Expected, since both rank on identical LOF scores and differ only in threshold. NB5 should assert this nesting explicitly; a violation means the two LOF runs did not see identical input.

✅ **Figure 2 independently reproduces.** NB01's correlation matrix matches every coefficient in the submitted Figure 2 (0.66, 0.68, 0.98, 0.74, 0.99, 0.73, −0.02), confirming parity at the EDA stage as well as at Tables 1–4.

⚠️ **Figure 7 does not reconcile.** Its published cell counts (4672 / 328 / 94328 / 672) give IF = 5,000 ✓ but LOF = 1,000, matching neither 2,565 nor 5,000. Table 2 implies 545 / 4455 / 4455 / 90545 for IF vs LOF(0.05). Recompute at NB5; do not port the original figure.

## 6. Discrepancy Log

Record any case where a rebuilt number differs from the original, however small, and the resolution:

| Date | Metric | Original | Rebuilt | Cause | Resolution |
|---|---|---|---|---|---|
| 2026-09-21 | Missing-value count, 7 numeric features | 0 (claimed, §4) | 12,962 | Rebuild attempt 1 coerced to numeric without stripping `$` / `,` first; NaNs then median-imputed. See §4.1 | Rebuild attempt 1 discarded in full. NB1 now strips before coercing and raises on any NaN. Original notebook still needed to confirm the original's own handling |
| | | | | | |

## 7. New Analysis Seeds (Notebooks 03–04, added for this revision)

| Notebook | Component | Seed/parameter | Value |
|---|---|---|---|
| 03 | Parameter grid | contamination range | _fill in once decided_ |
| 03 | Parameter grid | n_estimators range | _fill in once decided_ |
| 04 | Synthetic injection | injection rate | _fill in once decided_ |
| 04 | Synthetic injection | random seed | _fill in once decided_ |
