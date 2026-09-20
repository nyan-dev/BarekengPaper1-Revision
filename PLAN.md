# Project Plan — BAREKENG Manuscript 21575 Revision

**Single authoritative plan.** Supersedes `NOTEBOOK_GUIDE.md` and `NOTEBOOK_PIPELINE_PLAN.md`, both removed.

| | |
|---|---|
| **Manuscript** | A Comparative Analysis of Isolation Forest and Local Outlier Factor for Anomaly Detection in Healthcare Provider Data |
| **Journal** | BAREKENG: Journal of Mathematics and Its Applications (Scopus) |
| **MS ID / Decision** | 21575 — Accepted with Major Corrections |
| **Corresponding author** | Nyan Lynn Htet, INTI International University |
| **Plan date** | 2026-09-21 |

---

## 1. Two goals, two tracks

| Track | Goal | Owner | Gating |
|---|---|---|---|
| **A — Codebase** | A repo that runs correctly, 01→07, and reproduces every published number | Claude generates, author runs on Colab | Sequential; each notebook needs the previous handshake |
| **B — Manuscript** | A correct resubmission answering all 10 comments | Author + co-authors, Claude drafts | Partly gated on Track A outputs, partly independent |

Both must land. A perfect pipeline with an unrevised manuscript does not resubmit.

---

## 2. Status: the parity gate is already closed

**Every published value in Tables 1–4 was reproduced on 2026-09-21, before any notebook was written**, by porting the original notebook's cells [3], [4], [5] and [33] against `data/raw/healthcare_providers.csv`.

| Target | Published | Reproduced |
|---|---|---|
| IF count | 5,000 | 5,000 ✅ |
| LOF(auto) count | 2,565 | 2,565 ✅ |
| LOF(0.05) count | 5,000 | 5,000 ✅ |
| IF ∩ LOF(auto) | 341 | 341 ✅ |
| IF ∩ LOF(0.05) | 545 | 545 ✅ |
| Jaccard ×3 | 0.047 / 0.058 / 0.513 | 0.0472 / 0.0576 / 0.5130 ✅ |
| Mann-Whitney U ×4 | 5.10e8 / 5.36e8 / 1.21e7 / 1.46e7 | all ✅ |
| Chi-square ×4 | 7496.33 / 1855.13 / 2408.72 / 157.39 | all ✅ |

**Consequence:** every notebook is a *transcription of verified code*, not an experiment. A parity miss on Colab means an environment difference — check `REPRODUCIBILITY.md` §2 before suspecting the model.

Full record, including the recovered group definitions and the currency trap: `REPRODUCIBILITY.md`.

---

## 3. Ground-truth contract

Do not change any value in this section without logging it in `REPRODUCIBILITY.md` first.

### 3.1 Data

- Source: `data/raw/healthcare_providers.csv` — 100,000 rows × 27 cols
- Colab path: `/content/drive/MyDrive/BarekengPaper1-Revision/data/raw/healthcare_providers.csv`
- No ground-truth labels. This is a disclosed limitation, not a gap to fix.

### 3.2 The 7 numerical features

```python
FEATURES = [
    'Number of Services',
    'Number of Medicare Beneficiaries',
    'Number of Distinct Medicare Beneficiary/Per Day Services',
    'Average Medicare Allowed Amount',
    'Average Submitted Charge Amount',
    'Average Medicare Payment Amount',
    'Average Medicare Standardized Amount',
]
```

Categorical (Table 4 only): `Provider Type`, `Entity Type of the Provider`.

### 3.3 Cleaning — the one step that must not be skipped

All 7 columns are `object` dtype because of **thousands separators**. The file contains no `$` characters at all.

```python
def clean_numeric(series):
    return (series.astype(str)
            .str.replace(r'[\$,]', '', regex=True)
            .replace("nan", np.nan)
            .astype(float))
```

Then **assert zero NaN and halt on failure**:

```python
assert df_num.isna().sum().sum() == 0, "Currency stripping failed — do NOT impute"
```

Coercing without stripping turns every value ≥ 1,000 into NaN — 12,962 of them, all in the upper tail. Median-imputing those rewrites a 282,739-service provider as 43 services and a \$62,694 charge as \$146, erasing exactly the records the models exist to find. This destroyed the first rebuild. See `REPRODUCIBILITY.md` §4.1.

`fillna(median)` is retained for fidelity to the original but is a verified no-op and must never be reached with a non-zero count.

### 3.4 Models — from original notebook cell [33] (canonical) and cell [5]

| Model | Config | Input | Expected |
|---|---|---|---|
| Isolation Forest | `n_estimators=100, contamination=0.05, random_state=42` | `X_scaled` | 5,000 *(trivial)* |
| LOF auto | `n_neighbors=20, contamination='auto'` | `X_scaled` | **2,565** *(diagnostic)* |
| LOF 0.05 | `n_neighbors=20, contamination=0.05` | `X_scaled` | 5,000 *(trivial)* |

`X_scaled = StandardScaler().fit_transform(df_num_filled)` — **both** models fit on scaled data, all 7 features.

⚠️ The original notebook contains three LOF configurations. Cell **[10]** (2 unscaled features, `contamination=0.01`) and cell **[31]** are superseded. **Only cell [33] is canonical.**

⚠️ Counts of 5,000 are `contamination × n` and prove nothing — they pass on corrupted input. **LOF(auto) = 2,565 is the first real check in the pipeline.**

### 3.5 Group definitions for Tables 3–4 (recovered, undocumented in the manuscript)

| Group | Definition | n |
|---|---|---|
| **Anomaly** | **IF ∪ LOF(0.05)** — flagged by at least one | **9,455** |
| Normal | flagged by neither | 90,545 |
| IF Only | `IF \ LOF(0.05)` | 4,455 |
| LOF Only | `LOF(0.05) \ IF` | 4,455 |

LOF(auto) plays no role in Tables 3–4. Guessing "Anomaly = IF's 5,000" misses every statistic by ~40%.

### 3.6 Environment (parity-verified)

`numpy==2.0.2`, `pandas==2.3.3`, `scikit-learn==1.6.1`, `scipy==1.13.1`.

The original repo pinned nothing — this is the reference environment. Earlier bounds (`numpy<2.0.0`, `scikit-learn<1.6.0`) **excluded** these versions; do not reinstate them.

⚠️ **Colab does not match it.** The NB01 run resolved python 3.13.15, numpy 2.1.3, pandas 2.2.3. All NB01 statistics still came out byte-identical, but descriptive statistics are insensitive to these differences while IF and LOF are not — and Colab's `scikit-learn` version is still unobserved because NB01 does not import it. NB02 must capture it. See `REPRODUCIBILITY.md` §2.1.

---

## 4. Notebook architecture

Every notebook follows the 3-zone standard (`Notebook-Guide/notebook-standards.md`).

```
ZONE 1  PREAMBLE          Cell 00 [MD]   title, I/O contract, manuscript link
                          Cell 01 [code] Colab/local bootstrap, mkdir tree
                          Cell 02 [code] imports, SEED=42, rcParams, record versions
                          Cell 03 [code] validate upstream handshake  (NB02+ only)
ZONE 2  EXECUTION         Cell 04..N-2   one responsibility each
                          header: # Cell XX — <Category>: <Action>
                          Category ∈ Load|Transform|Feature|Test|Model|Plot|Export
                          every cell ends with a status print
ZONE 3  HANDOVER          Cell N-1 [code] export figures (300 dpi) + tables
                          Cell N   [code] write summary_NB{X}.json
```

**Conventions**

| | |
|---|---|
| Labels | sklearn native: `-1` anomaly, `+1` normal |
| Seeds | `SEED = 42`, `np.random.seed(SEED)`, `random_state=42` |
| Figures | `dpi=300`, `bbox_inches='tight'`, **PNG only** |
| Naming | `outputs/figures/nb{X}_*.png`, `outputs/tables/nb{X}_*.csv` |
| Paths | never hardcode `/content/...`; use `PROCESSED / 'file.parquet'` |
| Assertions | any number appearing in the manuscript gets a hard assert that **halts** |

PNG-only is deliberate. The pipeline standard mandates a vector `.pdf` alongside each PNG for camera-ready use, which is correct for a LaTeX manuscript. This manuscript is a Word document — the editor's comments appear in the submitted PDF as Word comment balloons (`Commented [DM1]`) — and Word cannot cleanly embed vector PDF, so the PNG is what actually gets pasted. The PDFs were dead weight and are not produced.

**Handshake schema** — `outputs/notebook_exports/summary_NB{X}.json`:

```json
{
  "notebook": "NB01",
  "status": "SUCCESS",
  "timestamp": "<ISO>",
  "environment": {"python": "", "numpy": "", "pandas": "", "scikit_learn": "", "scipy": ""},
  "inputs_consumed": [],
  "outputs": {},
  "key_results": {},
  "assertions": {},
  "downstream_note": ""
}
```

`key_results` is how findings reach the next notebook and the review loop — it must carry the actual numbers, not just file paths.

---

## 5. The seven notebooks

Build order = run order = numeric order. No exceptions.

| NB | File | Manuscript | Answers | Depends on |
|---|---|---|---|---|
| 01 | `01_data_and_eda.ipynb` | §2.1, Figs 1–2 | — | raw CSV |
| 02 | `02_baseline_models.ipynb` | §2.2–2.3, Table 1, Figs 4–5 | — | NB01 |
| 03 | `03_parameter_sensitivity.ipynb` | new §3.1 | **C2** | NB01 |
| 04 | `04_comparative_analysis.ipynb` | §3.2–3.3, Table 2, Figs 6–7 | **E1** | NB02 |
| 05 | `05_validation_synthetic.ipynb` | new §3.1/3.2 | **C1** | NB04 |
| 06 | `06_statistical_tests_effect_sizes.ipynb` | §3.4, Tables 3–4 | **C3** | NB02 |
| 07 | `07_manuscript_export.ipynb` | all assets | — | NB01–06 |

*Renumbered 2026-09-21: comparative analysis 05→04, validation 04→05, so that dependency order and numeric order coincide. Validation dropped "expert" from its name — see §7.*

### NB01 — Data & EDA
Load raw CSV, assert `(100000, 27)`. Extract 7 features, **strip → coerce → assert zero NaN**. Figure 1 (distributions) and Figure 2 (correlation heatmap) rebuilt from scratch — the published Figure 1 is an unreadable dense bar chart and must not be reproduced. Export `features_raw.parquet`.
→ `summary_NB01.json` carries the per-column NaN counts (expected all zero), shapes, and resolved package versions.

### NB02 — Baseline models
`StandardScaler` → IF and both LOF variants per §3.4. Assert IF = 5000, LOF(0.05) = 5000, and **LOF(auto) = 2565**. Figures 4–5 scatter plots. Export `anomaly_labels.parquet` with `IF_Label`, `LOF_Auto_Label`, `LOF_05_Label`.
→ handshake carries all three counts.

### NB03 — Parameter sensitivity (C2)
Sweep IF over `n_estimators` × `contamination`; LOF over `n_neighbors` × `contamination`. Report count stability and pairwise Jaccard across configs, anchored on the manuscript default. **Grid ranges need a citable justification, not just a span — see §7.**

### NB04 — Comparative analysis (E1)
Table 2 overlaps and Jaccard; assert 341 / 545 / 2565 and `LOF(auto) ⊆ LOF(0.05)`. Venn diagrams (Fig 6); Figure 7 **recomputed, not ported** — the published version's cell counts do not reconcile with Table 2.

**E1 mathematical depth.** Interpret Jaccard against a hypergeometric independence baseline rather than asserting it is "low": for two random 5,000-subsets of 100,000, expected overlap ≈ 250 (E[J] ≈ 0.026) against an observed 545 — roughly 2.2× chance. IF vs LOF(auto): expected ≈ 128, observed 341, ≈2.7×. Report the exact test and a bootstrap CI on J.

⚠️ This **reinterprets an accepted finding** — low overlap currently reads as the models disagreeing; against the null they agree well above chance while each keeping a large private subset. Complementarity survives and strengthens, but the change must be disclosed in the cover letter and approved by co-authors.

### NB05 — Synthetic validation (C1)
Inject synthetic anomalies at a fixed documented seed into a clean subset; rerun IF/LOF; report precision / recall / F1 against the known injected set. This is the quantitative core of the C1 response and stands on its own.

Second output: a descriptive case-profile table over the 545-record IF ∩ LOF intersection — percentile position and values against the distribution. **Not** framed as expert review; no clinical or fraud judgment is claimed. Expert validation stays in the Conclusion as future work.

### NB06 — Statistical tests & effect sizes (C3)
Mann-Whitney U and Chi-square using the §3.5 group definitions. Assert all eight published statistics. **Add** rank-biserial correlation (Table 3) and Cramér's V (Table 4).

Effect sizes are the substance of C3: at n = 100,000 everything is p < 0.001, which is the editor's point. Cramér's V is expected to show the IF-only vs LOF-only Provider Type difference as substantially stronger than the anomaly-vs-normal one — quantitative support for complementarity.

### NB07 — Manuscript export
No new computation. Re-import NB01–06 outputs, apply consistent styling, emit camera-ready figures (300 dpi PNG + vector PDF) and LaTeX tables to `outputs/manuscript_ready/`.

---

## 6. The working loop

```
Claude generates NB{N}  →  author runs it on Colab  →  author returns
outputs + summary_NB{N}.json to the repo  →  Claude reads the real numbers
→  Claude generates NB{N+1}
```

Rules: never generate NB{N+1} before NB{N}'s handshake is back — the numbers are not guessable. Never hand-edit a handshake. If an assertion fires, stop and diagnose the input; do not relax the assertion.

**Track B runs in parallel.** C6, E2 and E3 depend on no code and can start immediately. C4 and C5 come after Track A results exist. E4 is last.

---

## 7. Open decisions

| # | Decision | Blocks | Notes |
|---|---|---|---|
| 1 | IF/LOF grid ranges **and their justification** | NB03 | C2 asks for justification, not just a sweep. Needs a citable fraud-prevalence rationale for the contamination span. |
| 2 | Injection rate, method, seed, clean-subset definition | NB05 | Must be fixed constants logged in `REPRODUCIBILITY.md` §7. |
| 3 | Co-author sign-off on the E1 reinterpretation | manuscript §3.3, abstract | Changes how an accepted finding reads. |
| 4 | **Where the manuscript is edited** | all of Track B | The repo has only the submitted PDF. No editable source is present. Every Track B deliverable is a change to a document that is not here. |

---

## 8. Traceability

| ID | Source | Comment | Deliverable |
|---|---|---|---|
| C1 | Editor | Strengthen model validation | NB05 |
| C2 | Editor | Justify parameters, test stability | NB03 |
| C3 | Editor | Add effect-size measures | NB06 |
| C4 | Reviewer 2 | Abstract lacks limitations, originality | text — after C1–C3, E1 |
| C5 | Reviewer 2 | Conclusion doesn't answer purpose/limitations | text — after C1–C3, E1 |
| C6 | Reviewer 2 | IEEE style, ~80% within 10 years | text — **7** pre-2016 refs, not 5; 19/24 = 79.2% even after justifying five. Add 2–3 recent citations. |
| E1 | Editor letter | Strengthen mathematical aspects; deeper Jaccard/distribution interpretation | NB04 + text |
| E2 | Editor letter | Clarify novelty/contribution | text |
| E3 | Editor letter | Complete missing sections | text — Author Contributions, Funding, Acknowledgment, Declarations are all `XXX` on p.58 |
| E4 | Editor letter | Writing, grammar, terminology consistency | text — last |

C1–C3 are the editor's PDF margin comments; C4–C6 the Reviewer 2 form; E1–E4 the decision-letter body (`Guides-and-Docs/21575-Article Text-155003-1-18-20260430.md`, which despite its filename is the decision letter, not the manuscript).

E1 is the letter's **opening** comment and the highest-risk item: it questions whether an application-oriented paper fits a mathematics journal.

---

## 9. Immediate next actions

1. Generate **NB01** to this specification.
2. Confirm Drive layout: `MyDrive/BarekengPaper1-Revision/data/raw/healthcare_providers.csv`.
3. Run NB01 on Colab; return `summary_NB01.json` + `fig1` + `fig2`.
4. Generate **NB02**; confirm LOF(auto) = 2,565.
5. Continue 03 → 07 in order.
6. In parallel: C6, E2, E3. Resolve open decision 4 (editable manuscript source).
