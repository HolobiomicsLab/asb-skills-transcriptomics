---
name: single-cell-rare-cell-labeling
description: Use when you already have a per-cell FiRE rareness score vector and need
  a binary rare-cell label (0/1) per cell via the paper's IQR-based thresholding criterion.
when_to_use_negative:
- You only have raw or preprocessed expression data and no FiRE scores yet — run FiRE
  scoring first; this skill does not compute rareness scores
- You need a continuous rareness ranking rather than binary calls — FiRE's score itself,
  not the IQR dichotomization, is the appropriate output
- Your downstream goal is doublet detection — the paper notes doublet rates (1%–30%)
  do not necessarily comply with the notion of cell rareness
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: FiRE
  role: Source of the continuous per-cell rareness scores that are dichotomized into
    binary labels by this skill
  repo: https://github.com/princethewinner/FiRE
- name: Python
  role: Implementation language; numpy.percentile and scipy.stats.iqr compute the
    IQR threshold on the FiRE scores
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:da2391f233171bf217432a6ea214fb46dc9a528f4af666ecba838e4fe47b3555
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princethewinner/FiRE
  tools:
  - FiRE
  - Python
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-rare-cell-labeling

## Summary

Converts FiRE's continuous per-cell rareness scores into binary rare/not-rare cell labels using IQR-based thresholding (Tukey upper fence q3 + 1.5*IQR), the same procedure FiRE's authors used to flag rare cells (e.g., 727 rare cells in mouse brain, 4238 in ~68k PBMC data).

## When to use

Use when you already have a per-cell FiRE rareness score vector and need a binary rare-cell label (0/1) per cell via the paper's IQR-based thresholding criterion.

## When NOT to use

- You only have raw or preprocessed expression data and no FiRE scores yet — run FiRE scoring first; this skill does not compute rareness scores
- You need a continuous rareness ranking rather than binary calls — FiRE's score itself, not the IQR dichotomization, is the appropriate output
- Your downstream goal is doublet detection — the paper notes doublet rates (1%–30%) do not necessarily comply with the notion of cell rareness

## Inputs

- Per-cell FiRE rareness score vector (np.array [nCells], from FiRE model.score on the preprocessed expression matrix)
- Number of cells in the input expression matrix (to size the label vector)

## Outputs

- Binary rare-cell label vector (one label per cell, values 0 or 1)
- rare_cell_labels.csv (one-column table, one row per cell)

## How to apply

Load the FiRE score array (one value per cell, higher = rarer; produced by model.score on the log2-transformed ranger-preprocessed expression matrix). Compute q1 and q3 with numpy.percentile and derive IQR = q3 - q1 (equivalently scipy.stats.iqr), then set the threshold at the Tukey upper fence q3 + 1.5*IQR. Label a cell rare (1) when its score >= th, otherwise 0 — the README's demo does exactly this with np.where(score >= th) followed by predictions[indIqr] = 1. Rationale: FiRE itself returns only a continuous score, unlike RaceID/GiniClust which give binary predictions, so this dichotomization step is what yields per-cell rare-cell calls. Save the labels as a one-column table (one row per cell) to rare_cell_labels.csv.

## Related tools

- **FiRE** (Source of the continuous per-cell rareness scores that are dichotomized into binary labels by this skill) — https://github.com/princethewinner/FiRE
- **Python** (Implementation language; numpy.percentile and scipy.stats.iqr compute the IQR threshold on the FiRE scores)

## Examples

```
q3 = np.percentile(score, 75); iqr = stats.iqr(score); th = q3 + 1.5*iqr; indIqr = np.where(score >= th)[0]; predictions = np.zeros(data.shape[0]); predictions[indIqr] = 1
```

## Evaluation signals

- Output file exists and contains exactly one label per input cell (len(labels) == nCells)
- Every label is strictly binary (set of unique values is a subset of {0, 1})
- Number of rare (1) labels equals len(np.where(score >= q3 + 1.5*iqr)[0]); threshold reproduces q3 = np.percentile(score, 75) and iqr = stats.iqr(score)
- Labeled rare set matches the score thresholding invariant: min score among label-1 cells >= th and max score among label-0 cells < th

## Limitations

- FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types, so binary rare-cell labels may include technical outliers
- Outlier cells may get submerged into minor cell clusters when clustering FiRE-detected rare cells (e.g., with dropClust)
- The IQR-based threshold is dataset-dependent; the paper's flagged counts (727 rare cells in mouse brain, 4238 in ~68k PBMC data) will not transfer to other datasets

## Evidence

- [discussion] binary vs continuous predictions: "While RaceID15 and GiniClust16 offer binary predictions, FiRE gives a rareness score to every individual expression profile."
- [readme] IQR thresholding rationale: [evidence span withheld — anchor status: external_doc]
- [readme] score direction: [evidence span withheld — anchor status: external_doc]
- [readme] scoring step grounding: [evidence span withheld — anchor status: external_doc]
