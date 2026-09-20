# Agent Instructions (Claude Code / VS Code)

Context for any AI coding assistant working in this repository. Read `PRD.md`, `PLAN.md`, and `REPRODUCIBILITY.md` before making changes.

## Project Context

This repository rebuilds the codebase behind a manuscript under major revision at BAREKENG (journal). The manuscript compares Isolation Forest and Local Outlier Factor for anomaly detection in a healthcare claims dataset. The original results (from a now-archived repo) must remain exactly reproducible; new work only adds to them.

## Hard Rules

1. **Never change a random seed, contamination value, or `n_neighbors` in notebooks 01–02 or 05–06 without explicit user approval.** These reproduce the original submitted numbers. Log any proposed change in `REPRODUCIBILITY.md` before making it.
2. **Never silently alter a previously reported figure or table value.** If a rebuilt number differs from the original (see `REPRODUCIBILITY.md` §5 parity check), stop and flag it — do not "fix" it by adjusting parameters until told to.
3. **Do not introduce new algorithms or datasets** beyond what's scoped in `PRD.md` §3 (no OCSVM, no external labeled datasets) unless the user explicitly requests it.
4. **Every notebook's outputs must land in the exact paths specified in `PLAN.md`** (`data/processed/`, `outputs/figures/`, `outputs/tables/`) so downstream notebooks can consume them without path guessing.
5. **Pin package versions** in `requirements.txt` — do not use unpinned installs.
6. **Fixed seeds for all new stochastic processes** (synthetic injection in notebook 04, any parameter grid randomization in notebook 03) must be explicit constants, documented in `REPRODUCIBILITY.md` §7, not left to library defaults.
7. **Build order, run order and numeric order are identical** (01 → 07). Notebooks were renumbered on 2026-09-21 precisely so no exception is needed. Do not implement notebook N+1 logic inside notebook N.
8. **Never coerce the 7 numeric features without stripping `$` and `,` first,** and never impute the result. Strip → coerce → `assert isna().sum() == 0` → raise on failure. Skipping the strip turns every value ≥ 1,000 into NaN, and imputing those replaces the largest ~6.7% of the distribution — the anomalies themselves — with the median. This destroyed rebuild attempt 1; see `REPRODUCIBILITY.md` §4.1.
9. **Use the recovered group definitions for Tables 3–4.** "Anomaly" means `IF ∪ LOF(0.05)` (n = 9,455), not IF's 5,000. This is undocumented in the manuscript and was recovered empirically; guessing wrong misses every statistic by ~40%. See `REPRODUCIBILITY.md` §5.2.
10. **An assertion that fires is evidence, not an obstacle.** When a check fails, diagnose the input. Never relax or delete the check — that is exactly how rebuild attempt 1 converted a loud failure into silent data destruction.

## Code Style

- Python, `snake_case` for variables/functions.
- No hardcoded absolute file paths — use relative paths from repo root via a shared `config.py` or top-of-notebook constants.
- Every notebook starts with a markdown cell stating: purpose, manuscript link, and key parameters (mirror `PLAN.md` entry).
- Docstrings on any function longer than 5 lines.
- Plots: consistent style (reuse a shared `plot_style.py` if figures need to match original formatting for Figs 1–8).

## When Uncertain

If a task requires a judgment call that affects reported results (e.g., "what contamination range should the sensitivity sweep cover?"), stop and ask the user rather than picking a default — this is a manuscript under peer review, not exploratory analysis.

## Definition of Done for Any Task

A task in this repo is not complete until:
- Its output file(s) exist in the correct path per `PLAN.md`.
- Any new parameter/seed is logged in `REPRODUCIBILITY.md`.
- If it touches notebooks 01–02/05–06, the parity check table in `REPRODUCIBILITY.md` §5 has been re-verified.
