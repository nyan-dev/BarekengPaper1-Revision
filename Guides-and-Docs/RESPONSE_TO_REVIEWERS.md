# Response to Reviewers — Manuscript 21575 (BAREKENG)

Living checklist tracking each comment through to resolution. This doubles as the draft source for the formal response-to-reviewers letter at resubmission.

---

## Editor Comments

### Comment 1 — Model Validation (§2.4 / §3.1)

> "Strengthen the model validation (Section 2.4 and Results) by adding a clearer evaluation approach so that the model performance can be assessed more convincingly, for example through labeled data, expert validation, or other relevant evaluation methods."

- **Decision:** Synthetic anomaly injection (quantitative) + expert face-validity review of the IF∩LOF overlap set (qualitative). External labeled dataset rejected as out of scope (schema mismatch risk).
- **What changed:** New notebook `04_validation_synthetic_and_expert.ipynb`; new manuscript subsection reporting precision/recall on injected anomalies and an annotated table of top-confidence anomalies.
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
- **What changed:** Reference list formatting; cover-letter justification paragraph for the 5 foundational exceptions.
- **Status:** Not started

---

## Resubmission Package Checklist

- [ ] Parity check passed (see `REPRODUCIBILITY.md` §5)
- [ ] Notebooks 03–07 complete with outputs exported
- [ ] Manuscript Methods/Results updated with new subsections (C1, C2, C3)
- [ ] Abstract rewritten (C4)
- [ ] Conclusion rewritten (C5)
- [ ] Reference list audited (C6)
- [ ] This document finalized into formal response-to-reviewers letter
- [ ] Original repo archived and tagged `v1.0-submitted`
- [ ] New repo tagged at resubmission point (e.g., `v2.0-resubmitted`)
