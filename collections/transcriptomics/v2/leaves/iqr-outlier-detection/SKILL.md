---
name: iqr-outlier-detection
description: Use when you have a per-cell continuous FiRE rareness score vector and
  need to convert it to binary rare-cell calls via a distribution-based IQR cutoff.
when_to_use_negative:
- Input is already a binary rare/abundant label table (e.g., RaceID or GiniClust output)
  — no thresholding needed
- The score distribution is not the FiRE rareness score (e.g., expression matrix or
  cluster IDs); IQR fence cutoffs are not meaningful there
- A fixed expected rare-fraction is required instead of a distribution-derived cutoff
  — the IQR fence yields data-dependent rare-cell counts (4238/68k in the PBMC analysis)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FiRE
  role: Produces the continuous per-cell rareness scores that this skill dichotomizes
    into binary rare/not-rare labels
  repo: github.com/princethewinner/FiRE
- name: Python
  role: 'Implementation environment: numpy percentile and scipy.stats.iqr compute
    the threshold; numpy builds the 0/1 prediction vector'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:c774be85347bc89a496ed301329d736aa960d0eceffe4a0b0f4ea9f9612a3618
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# iqr-outlier-detection

## Summary

Dichotomizes continuous FiRE rareness scores into binary rare/not-rare cell labels using Tukey-style IQR thresholding (score >= q3 + 1.5*IQR), as used in the FiRE paper to flag rare cells (e.g., 727 in mouse brain, 4238 in ~68k PBMC data).

## When to use

Use when you have a per-cell continuous FiRE rareness score vector and need to convert it to binary rare-cell calls via a distribution-based IQR cutoff.

## When NOT to use

- Input is already a binary rare/abundant label table (e.g., RaceID or GiniClust output) — no thresholding needed
- The score distribution is not the FiRE rareness score (e.g., expression matrix or cluster IDs); IQR fence cutoffs are not meaningful there
- A fixed expected rare-fraction is required instead of a distribution-derived cutoff — the IQR fence yields data-dependent rare-cell counts (4238/68k in the PBMC analysis)

## Inputs

- Per-cell FiRE rareness score vector (np.array[nCells], higher values = rarer)
- Number of input cells nCells (to enforce one label per cell)

## Outputs

- Binary rare-cell label vector (one 0/1 label per cell)
- rare_cell_labels.csv (one-column table, one row per cell)

## How to apply

Load the per-cell FiRE score array produced by the upstream scoring step (model.score(preprocessedData) over the log2-transformed top-1000 variable genes matrix). Compute the third quartile q3 = np.percentile(score, 75) and IQR = stats.iqr(score) over all cells, then set the threshold th = q3 + 1.5*IQR (the Tukey upper fence). Mark a cell rare (label 1) when its score >= th, and 0 otherwise, yielding a one-column binary label vector aligned one-to-one with the input cells; FiRE's convention is that higher score values represent rare cells. Rationale: the paper distinguishes this score-to-call step from RaceID/GiniClust, which 'offer binary predictions' natively, so IQR-based thresholding supplies the binary calls FiRE lacks. Save the labels (e.g., rare_cell_labels.csv) and verify each input cell has exactly one binary (0/1) label.

## Related tools

- **FiRE** (Produces the continuous per-cell rareness scores that this skill dichotomizes into binary rare/not-rare labels) — github.com/princethewinner/FiRE
- **Python** (Implementation environment: numpy percentile and scipy.stats.iqr compute the threshold; numpy builds the 0/1 prediction vector)

## Examples

```
th = np.percentile(score, 75) + 1.5*stats.iqr(score); indIqr = np.where(score >= th)[0]; predictions = np.zeros(len(score)); predictions[indIqr] = 1
```

## Evaluation signals

- Output file exists and has exactly one label per input cell (len(labels) == nCells)
- Every label is binary: values are only 0 or 1
- Threshold reproduces the formula th = np.percentile(score, 75) + 1.5*stats.iqr(score); number of 1-labels equals np.sum(score >= th)
- All flagged rare cells have scores at or above the computed Tukey upper fence q3 + 1.5*IQR (no cell below threshold is labeled 1)
- Rerunning with the same score vector and seed-stable FiRE model yields identical label counts (e.g., 727 rare cells in the mouse brain data)

## Limitations

- The IQR fence is distribution-derived, so the number of rare cells called varies by dataset; it does not enforce a fixed rare fraction
- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types — flagged cells may include technical outliers, not just rare populations
- This cutoff is calibrated for FiRE score distributions; applying it to other score types (e.g., doublet scores, whose rates range 1–30%) may not align with the notion of cell rareness

## Evidence

- [discussion] While RaceID15 and GiniClust16 offer binary predictions, FiRE gives a rareness score to every individual expression profile.: "While RaceID15 and GiniClust16 offer binary predictions, FiRE gives a rareness score to every individual expression profile."
- [readme] IQR thresholding code from README usage: "q3 = np.percentile(score, 75)
iqr = stats.iqr(score)
th = q3 + 1.5*iqr"
- [readme] Higher values of FiRE score represent rare cells.: [evidence span withheld — anchor status: external_doc]
- [readme] Create a file with binary predictions: "#Create a file with binary predictions
predictions = np.zeros(data.shape[0])
predictions[indIqr] = 1 #Replace predictions for rare cells with '1'."
- [abstract] FiRE assigns rareness scores in seconds: "an algorithm that, in a matter of seconds, assigns a rareness score to every individual expression profile under study"
