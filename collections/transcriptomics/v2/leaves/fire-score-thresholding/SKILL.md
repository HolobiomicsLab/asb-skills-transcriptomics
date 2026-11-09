---
name: fire-score-thresholding
description: Use when you have a continuous FiRE rareness score for every cell (np.array
  [nCells]) and need a binary rare-cell annotation per cell, because downstream validation
  or visualization (e.g., t-SNE overlay of detected rare cells) requires discrete
  0/1 calls rather than raw scores.
when_to_use_negative:
- Input is already a binary rare/not-rare annotation (e.g., labels from RaceID or
  GiniClust) rather than a continuous FiRE score vector
- The analysis goal is only a continuous rareness ranking for visualization (e.g.,
  FiRE score intensity plotted on a t-SNE map) with no need for discrete calls
- You need to separate technical outliers from cells of genuinely rare cell types
  — the article notes 'FiRE, in principle, does not discriminate between an outlier
  and cells representing minor cell types', so this thresholding alone cannot resolve
  that
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FiRE
  role: Source of the continuous per-cell rareness scores that this skill dichotomizes
    via IQR thresholding
  repo: github.com/princethewinner/FiRE
- name: Python
  role: Implements the thresholding with numpy.percentile, scipy.stats.iqr, and numpy.where,
    and writes the binary labels CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:432e0d46af9b9b090d14216ff96712d1c5340fb206c45fc7fb87dab63cc85426
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# fire-score-thresholding

## Summary

Dichotomizes FiRE's continuous per-cell rareness scores into binary rare/not-rare cell labels using Tukey IQR-based thresholding (score >= q3 + 1.5*IQR), the technique FiRE's authors used to flag rare cells (e.g., 727 rare cells in the mouse brain data and 4238 in the ~68k PBMC data).

## When to use

Use when you have a continuous FiRE rareness score for every cell (np.array [nCells]) and need a binary rare-cell annotation per cell, because downstream validation or visualization (e.g., t-SNE overlay of detected rare cells) requires discrete 0/1 calls rather than raw scores.

## When NOT to use

- Input is already a binary rare/not-rare annotation (e.g., labels from RaceID or GiniClust) rather than a continuous FiRE score vector
- The analysis goal is only a continuous rareness ranking for visualization (e.g., FiRE score intensity plotted on a t-SNE map) with no need for discrete calls
- You need to separate technical outliers from cells of genuinely rare cell types — the article notes 'FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types', so this thresholding alone cannot resolve that

## Inputs

- Per-cell FiRE rareness score vector: np.array [nCells] from FiRE model.score() on the preprocessed (log2-transformed, top-1000 variable genes) data matrix
- Optionally the preprocessed data matrix np.array [nCells, nVariableGenes] to subset rare cells (dataSel = preprocessedData[indIqr,:])

## Outputs

- Binary rare-cell label vector (0/1) with one label per input cell, saved to rare_cell_labels.csv
- Indices of rare cells (indIqr) satisfying score >= q3 + 1.5*IQR

## How to apply

Load the per-cell FiRE score vector produced by model.score() of the FiRE model (L=100, M=50 estimators; higher scores mean rarer cells). Compute the third quartile with numpy (q3 = np.percentile(score, 75)) and the interquartile range (iqr = stats.iqr(score) from scipy.stats), then derive the Tukey upper fence th = q3 + 1.5*iqr. Mark a cell rare when score >= th using np.where(score >= th)[0]; because FiRE itself only outputs scores — unlike RaceID and GiniClust, which 'offer binary predictions' — this thresholding step is what yields binary calls. Build a predictions array of zeros of length nCells and set predictions[indIqr] = 1 for the rare cells, then save as a one-column CSV with exactly one 0/1 label per input cell. Validate that the output row count equals the number of input cells and that every label is binary.

## Related tools

- **FiRE** (Source of the continuous per-cell rareness scores that this skill dichotomizes via IQR thresholding) — github.com/princethewinner/FiRE
- **Python** (Implements the thresholding with numpy.percentile, scipy.stats.iqr, and numpy.where, and writes the binary labels CSV)

## Examples

```
q3 = np.percentile(score, 75); iqr = stats.iqr(score); th = q3 + 1.5*iqr; indIqr = np.where(score >= th)[0]; predictions = np.zeros(n); predictions[indIqr] = 1
```

## Evaluation signals

- Output file exists and contains exactly one binary label (0 or 1) per input cell — row count of rare_cell_labels.csv equals length of the score vector
- Recomputing q3 = np.percentile(score, 75), iqr = stats.iqr(score), and th = q3 + 1.5*iqr reproduces the exact set of flagged cells (indIqr matches the labeled-1 rows)
- No cell with score >= th is labeled 0 and no cell with score < th is labeled 1 (threshold invariant)
- The number of rare calls is a small subset of all cells, consistent with paper-scale results (e.g., 727 rare cells in the mouse brain dataset, 4238 in the ~68k PBMC dataset)

## Limitations

- FiRE 'in principle, does not discriminate between an outlier and cells representing minor cell types', so IQR-flagged cells may include outliers rather than genuine rare types
- When clustering the FiRE-detected rare cells with dropClust (which 'does not administer any special treatment to outliers'), outlier cells may get submerged into the minor cell clusters
- The threshold is fixed at the Tukey fence q3 + 1.5*IQR on the score distribution; it provides no per-cell probability and its behavior depends on the shape of the score distribution

## Evidence

- [readme] q3 = np.percentile(score, 75)
iqr = stats.iqr(score)
th = q3 + 1.5*iqr: "q3 = np.percentile(score, 75)
iqr = stats.iqr(score)
th = q3 + 1.5*iqr"
- [readme] predictions[indIqr] = 1 #Replace predictions for rare cells with '1'.: [evidence span withheld — anchor status: external_doc]
- [readme] Higher values of FiRE score represent rare cells.: [evidence span withheld — anchor status: external_doc]
- [discussion] While RaceID15 and GiniClust16 offer binary predictions, FiRE gives a rareness score to every individual expression profile.: "While RaceID15 and GiniClust16 offer binary predictions, FiRE gives a rareness score to every individual expression profile."
