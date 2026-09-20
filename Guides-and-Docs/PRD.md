# Project Requirements Document (PRD)
## Project: BAREKENG Major-Revision — Anomaly Detection in Healthcare Provider Data

**Project codename:** Project Sept 2
**Manuscript number:** 21575
**Title:** A Comparative Analysis of Isolation Forest and Local Outlier Factor for Anomaly Detection in Healthcare Provider Data: An Applied Study
**Target journal:** BAREKENG: Journal of Mathematics and Its Applications (Scopus-indexed, Pattimura University)
**Decision:** Accepted with major corrections
**Corresponding author:** Nyan Lynn Htet (INTI International University)

---

## 1. Purpose

Produce a fully revised manuscript and a rebuilt, reproducible codebase that satisfies all editor and reviewer comments from the BAREKENG major-correction decision, then resubmit.

## 2. Background

The original submission applied Isolation Forest (IF) and Local Outlier Factor (LOF) to a 100,000-record Kaggle healthcare claims dataset (`tamilsel/healthcare-providers-data`), compared anomaly sets via Jaccard Index, and validated group differences via Mann-Whitney U and Chi-square tests. The dataset has no ground-truth fraud labels — this is an inherent, disclosed limitation, not an oversight.

The original codebase (`nyan-dev/Anomaly-Detection-IF-LOF-OCSVM`) will be **archived and renamed** (e.g., `Anomaly-Detection-IF-LOF-OCSVM-v1-archive`) as a frozen, citable snapshot of what generated the original submitted numbers. A new repository will be built from the ground up for this revision cycle.

## 3. Scope

### In scope
- Rebuilding the analysis pipeline into 7 traceable notebooks.
- Reproducing the original submitted numbers exactly (parity check) before adding anything new.
- Adding three new/extended analyses required by the editor.
- Rewriting three manuscript sections/elements flagged by the second reviewer.
- Producing a response-to-reviewers letter.
- Tagging and archiving the original repo for provenance.

### Out of scope
- Introducing new external datasets (e.g., `rohitrox/healthcare-provider-fraud-detection-analysis`) as primary evidence — rejected in favor of self-contained synthetic validation + expert review (see Section 6, Comment 1).
- Adding new algorithms beyond IF and LOF (OCSVM exists in the old repo's `Tuning/` folder but is not part of the submitted manuscript's scope; do not introduce it unless a future reviewer explicitly asks).
- Restructuring the paper's core argument or conclusions about IF/LOF complementarity — that finding stands; only its evidentiary support is being strengthened.

## 4. Success Criteria (Definition of Done)

1. All 6 comments (3 editor + 3 reviewer-form) have a documented, traceable fix.
2. New repo reproduces original Table 1–4 values exactly under pinned environment + seed (parity check passes).
3. Three new analyses (parameter sensitivity report, synthetic validation, expert face-validity table) exist as notebooks with outputs exported.
4. Effect sizes (rank-biserial correlation, Cramér's V) added to Tables 3–4.
5. Abstract includes limitations and originality/value.
6. Conclusion explicitly maps back to stated purpose and limitations.
7. Reference list reviewed for IEEE style and 10-year recency ratio.
8. Response-to-reviewers letter drafted, comment-by-comment.
9. Manuscript and repo tagged and packaged for resubmission.

## 5. Stakeholders

- **Author/PI:** Grama Ma (project owner, decision-maker on methodology choices)
- **Co-authors:** Deshinta Arrova Dewi, Marwan Alshare, Mun San Chye (not directly involved in this AI-assisted workflow but must review final outputs)
- **Journal:** BAREKENG editorial team + Reviewer 2 (structured assessment form)

## 6. Requirements Traceability (Comment → Fix → Deliverable)

| ID | Source | Comment | Fix type | Deliverable |
|---|---|---|---|---|
| C1 | Editor | Strengthen model validation (§2.4/§3.1) via labeled data, expert validation, or other method | Code (new) | `04_validation_synthetic_and_expert.ipynb`: synthetic injection precision/recall + expert face-validity table on the 545-record IF∩LOF overlap set |
| C2 | Editor | Justify parameter selection (§3.1); test multiple settings for stability | Code (extend existing `Tuning/`) | `03_parameter_sensitivity.ipynb`: consolidated sweep over IF (`n_estimators`, `contamination`) and LOF (`n_neighbors`, `contamination`), stability measured via pairwise Jaccard across configs |
| C3 | Editor | Add magnitude/effect-size measures to statistical tests (§3.4) | Code (extend existing) | `06_statistical_tests_effect_sizes.ipynb`: add rank-biserial correlation (Mann-Whitney) and Cramér's V (Chi-square) to Tables 3–4 |
| C4 | Reviewer form | Abstract missing limitations, originality/value | Writing only | Revised Abstract paragraph |
| C5 | Reviewer form | Conclusion doesn't answer purpose/research problem/limitations explicitly | Writing only | Revised Conclusion, restructured to map back to stated purpose |
| C6 | Reviewer form | ~80% references within last 10 years + IEEE style compliance | Writing only | Reference list audit; retain justified exceptions (Liu et al. 2008, Breunig et al. 2000, Jaccard 1901, Mann-Whitney 1947, Pearson 1900 — foundational/methodological citations, explicitly justified in cover letter if flagged again) |

## 7. Sequencing Constraint

C1–C3 require new analysis output before their corresponding manuscript text can be written truthfully. C4–C6 are writing-only and can proceed in parallel, independent of the rebuild. Final Abstract/Conclusion pass happens **after** C1–C3 results exist, since both sections must reference the finalized validation/effect-size findings.

## 8. Non-Negotiables

- Original submitted numbers must remain reproducible and citable (frozen in the archived repo, tagged `v1.0-submitted`).
- No silent changes to previously reported figures — any discrepancy between old and rebuilt numbers must be logged in `REPRODUCIBILITY.md` and disclosed in the cover letter if material.
- All new random processes (synthetic injection, any resampling) must use a fixed, documented seed.
