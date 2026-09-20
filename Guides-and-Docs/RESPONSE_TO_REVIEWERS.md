# Response to Reviewers — Manuscript 21575 (BAREKENG)

Living checklist tracking each comment through to resolution. This doubles as the draft source for the formal response-to-reviewers letter at resubmission.

---

## Editor Comments

### Comment 1 — Model Validation (§2.4 / §3.1)

> "Strengthen the model validation (Section 2.4 and Results) by adding a clearer evaluation approach so that the model performance can be assessed more convincingly, for example through labeled data, expert validation, or other relevant evaluation methods."

- **Decision:** Synthetic anomaly injection, reporting precision/recall/F1 against a known injected ground truth. The editor offered three options — labeled data, expert validation, *or other relevant evaluation methods* — and synthetic injection is a self-contained instance of the third. External labeled dataset rejected as out of scope (schema mismatch risk). **Expert validation is not performed**; it stays in the Conclusion as future work, unchanged from the submitted version.
- **What changed:** New notebook `05_validation_synthetic.ipynb`; new manuscript subsection reporting precision/recall/F1 on injected anomalies, plus a descriptive table characterising the 545 consensus anomalies by percentile position and values against the distribution — statistical description only, not expert judgment.
- **Status:** Not started
- **Draft response text:** _to be written once results exist_

### Comment 2 — Parameter Justification and Stability (§3.1)

> "Provide clearer justification for the selected model parameters (Section 3.1) and test several parameter settings to ensure that the results are stable and not dependent on a single configuration."

- **Decision:** Consolidate existing `Tuning/` notebooks into one sensitivity sweep, report stability via pairwise Jaccard across configurations.
- **What changed:** New notebook `03_parameter_sensitivity.ipynb`; new manuscript subsection with justification narrative for chosen contamination/n_neighbors values plus a stability table/figure.
- **Status:** Not started
- **Draft response text:** _to be written once results exist_

### Comment 3 — Effect Size / Practical Significance (§3.4)

> "Improve the statistical analysis (Section 3.4) by including additional measures that reflect the magnitude of differences, since the current tests only indicate the presence of differences, not their practical significance."

- **Decision:** Add rank-biserial correlation to Mann-Whitney results (Table 3) and Cramér's V to Chi-square results (Table 4).
- **What changed:** Extended `06_statistical_tests_effect_sizes.ipynb`; Tables 3–4 gain effect-size columns; §3.4 text updated to interpret magnitude, not just significance.
- **Status:** Not started
- **Draft response text:** _to be written once results exist_

---

## Reviewer 2 — Structured Assessment Form

### Comment 4 — Abstract Missing Limitations and Originality/Value

> Form flagged "NO" on: "The abstract includes purpose, methodology/approach/design, findings, limitations, originality/value, and keywords."

- **Decision:** Rewrite abstract to explicitly add one clause on limitations (no ground-truth labels; addressed via synthetic validation) and one clause on originality/value (multi-model complementarity framework for healthcare claims).
- **What changed:** Abstract text only.
- **Status:** Not started

### Comment 5 — Conclusion Doesn't Answer Purpose/Problem/Limitations

> Form flagged "NO" on: "Conclusion(s) answer the purpose, research problems and limitation."

- **Decision:** Restructure conclusion into explicit paragraphs: (1) restate purpose and how it was met, (2) key findings, (3) limitations (now partially mitigated by new validation work), (4) future work.
- **What changed:** Conclusion text only.
- **Status:** Not started

### Comment 6 — Reference Recency and IEEE Style

> Form flagged "NO" on: "About 80% of the reference/bibliography uses the latest sources (at least the last 10 years) and follows the format of style IEEE."

- **Decision:** Audit all 24 references. Foundational/methodological citations older than 10 years (Liu et al. 2008 — Isolation Forest; Breunig et al. 2000 — LOF; Jaccard 1901; Mann-Whitney 1947; Pearson 1900) are retained with explicit justification in the response letter as they define the core methods used, not general background. Verify remaining citations for strict IEEE formatting compliance.
- **⚠️ Audit correction (2026-09-21):** the submitted reference list contains **7** pre-2016 citations, not 5. The two unaccounted for are **[8] Pedregosa et al. 2011** (scikit-learn) and **[10] Rokach 2010** (ensemble-based classifiers) — neither is foundational to IF or LOF in the way the other five are, so neither is straightforward to justify. Arithmetic: granting all five exceptions still leaves 19/24 = **79.2%**, fractionally below the reviewer's 80% threshold. Adding 2–3 recent citations is the cleanest fix and clears the bar without argument.
- **What changed:** Reference list formatting; cover-letter justification paragraph for the foundational exceptions; 2–3 recent citations added.
- **Status:** Not started

---

## Editor Decision Letter — Scope-Fit Comments

Sourced from `21575-Article Text-155003-1-18-20260430.md`. These were **not tracked** in the original matrix. E1 is the letter's opening comment and is the highest-risk item in the revision: it questions whether the paper fits a mathematics journal at all.

### Comment E1 — Strengthen Mathematical Aspects / Depth of Analysis

> "The paper is currently more application-oriented (machine learning). It should include deeper mathematical or statistical analysis, such as theoretical justification, formulation, or analytical discussion of the algorithms."
> "The discussion is mostly descriptive. More rigorous statistical and mathematical interpretation is needed (e.g., interpretation of Jaccard Index, distribution analysis)."

- **Decision:** Interpret the Jaccard index against a hypergeometric independence baseline instead of asserting it is "low" in the abstract. For IF vs LOF(0.05), two random subsets of 5,000 drawn from 100,000 would intersect in ~250 records by chance (E[J] ≈ 0.026); the observed 545 is ~2.2× that. IF vs LOF(auto): expected ~128, observed 341, ~2.7×. Report the exact test and a bootstrap CI on J, and add an analytical discussion of why Eq. 2 and Eqs. 3–4 induce different detection biases.
- **Note:** this **reinterprets an accepted finding**. The current text reads low J as the models disagreeing; against the null they agree well above chance while each retaining a large private subset. The complementarity conclusion is unchanged and better supported, but the reinterpretation must be disclosed in the cover letter and approved by co-authors.
- **What changed:** New analysis in `04_comparative_analysis.ipynb`; §3.3 rewritten; abstract wording on overlap revised.
- **Status:** Approved in principle 2026-09-21; values pending NB5.

### Comment E2 — Clarify Novelty / Contribution

> "The contribution compared to existing studies should be explicitly highlighted."

- **Decision:** Explicit contribution statement in the Introduction, positioned against [17] Fadul 2023, [18] Mostafa Mohamed et al. 2024, [21] Kurniawan et al. 2024.
- **Status:** Not started

### Comment E3 — Complete Missing Sections

> "Complete missing sections (Author Contributions, Funding, etc.)"

- **Decision:** Manuscript p.58 has Author Contributions, Funding Statement, Acknowledgment and Declarations all as `XXX` placeholders. All four to be written before resubmission.
- **Status:** Not started

### Comment E4 — Academic Writing, Grammar, Terminology

> "Improve academic writing and grammar"; "Ensure consistency of terminology"

- **Decision:** Full editorial pass. Fix inconsistent use of *anomaly* / *outlier* / *abnormality* / *irregularity*, which are currently interchangeable.
- **Status:** Not started

---

## Resubmission Package Checklist

- [ ] Parity check passed (see `REPRODUCIBILITY.md` §5)
- [ ] Notebooks 03–07 complete with outputs exported
- [ ] Manuscript Methods/Results updated with new subsections (C1, C2, C3)
- [ ] Abstract rewritten (C4)
- [ ] Conclusion rewritten (C5)
- [ ] Reference list audited, 2–3 recent citations added to clear 80% (C6)
- [ ] Jaccard null-model analysis added; §3.3 and abstract overlap wording revised (E1)
- [ ] Contribution statement added to Introduction (E2)
- [ ] Author Contributions / Funding / Acknowledgment / Declarations written (E3)
- [ ] Editorial pass + terminology consistency (E4)
- [ ] Co-authors have signed off on the E1 reinterpretation of the overlap finding
- [ ] This document finalized into formal response-to-reviewers letter
- [ ] Original repo archived and tagged `v1.0-submitted`
- [ ] New repo tagged at resubmission point (e.g., `v2.0-resubmitted`)
