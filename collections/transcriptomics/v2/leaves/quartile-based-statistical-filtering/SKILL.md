---
name: quartile-based-statistical-filtering
description: Use when you have a continuous per-cell FiRE rareness score vector and
  need a binary rare-cell label per cell (0/1) via a distribution-based, parameter-free
  cutoff on scRNA-seq data.
when_to_use_negative:
- 'FiRE has not yet been run: no continuous score vector exists, so there is nothing
  to threshold (this filter consumes scores, not raw expression data)'
- You need to distinguish true minor cell types from technical outliers — the paper
  notes FiRE 'does not discriminate between an outlier and cells representing minor
  cell types', so downstream clustering (e.g., dropClust) is still required
- A rare fraction with a known expected proportion is targeted and a fixed quantile
  cut (e.g., top 1%) is preferable — the 1.5*IQR fence yields a data-driven count
  that may not match a prespecified rare-cell rate
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3516
tools:
- name: FiRE
  role: Source of the continuous per-cell rareness scores (model.score) that this
    IQR-based filter dichotomizes
  repo: github.com/princethewinner/FiRE
- name: Python
  role: 'Implementation language: numpy.percentile, scipy.stats.iqr, and numpy.where
    compute the threshold and binary labels'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-07234-6
    title: Discovery of rare cells from voluminous single cell expression data
  source_content_hash: sha256:5ac682d8e082c87c51bca6473a80bbae4396c8a1a1f88743940fad35c369c3d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FiRE
  - Python
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3516
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# quartile-based-statistical-filtering

## Summary

Converts FiRE's continuous per-cell rareness scores into binary rare/not-rare calls by thresholding at the Tukey upper fence q3 + 1.5*IQR of the score distribution, mirroring the IQR-based criterion used in the FiRE paper and its reference Python demo.

## When to use

Use when you have a continuous per-cell FiRE rareness score vector and need a binary rare-cell label per cell (0/1) via a distribution-based, parameter-free cutoff on scRNA-seq data.

## When NOT to use

- FiRE has not yet been run: no continuous score vector exists, so there is nothing to threshold (this filter consumes scores, not raw expression data)
- You need to distinguish true minor cell types from technical outliers — the paper notes FiRE 'does not discriminate between an outlier and cells representing minor cell types', so downstream clustering (e.g., dropClust) is still required
- A rare fraction with a known expected proportion is targeted and a fixed quantile cut (e.g., top 1%) is preferable — the 1.5*IQR fence yields a data-driven count that may not match a prespecified rare-cell rate

## Inputs

- Per-cell FiRE rareness score vector (np.array[nCells], higher values = rarer cells)
- Number of cells in the input expression matrix (nCells, to size the binary label vector)

## Outputs

- Binary rare-cell label vector (one 0/1 label per cell) saved as rare_cell_labels.csv
- Optionally, index list indIqr of rare-cell row indices for subsetting the preprocessed data matrix

## How to apply

Load the per-cell FiRE score array produced by model.score(preprocessedData), where higher scores indicate rarer cells. Compute q3 = np.percentile(score, 75) and the interquartile range via scipy.stats.iqr, then set the threshold th = q3 + 1.5*iqr (the Tukey upper fence). Mark every cell with score >= th as rare: initialize a zero vector of length nCells and set predictions[indIqr] = 1 for indices where score >= th, then write the one-column label table (rare_cell_labels.csv). This mirrors the README's jurkat demo, which builds binary predictions from indIqr = np.where(score >= th)[0]. Rationale: FiRE itself only yields a rareness score, so an IQR-based fence on the score distribution supplies the binary rare/not-rare annotation the way RaceID and GiniClust produce binary predictions.

## Related tools

- **FiRE** (Source of the continuous per-cell rareness scores (model.score) that this IQR-based filter dichotomizes) — github.com/princethewinner/FiRE
- **Python** (Implementation language: numpy.percentile, scipy.stats.iqr, and numpy.where compute the threshold and binary labels)

## Examples

```
q3 = np.percentile(score, 75); iqr = stats.iqr(score); indIqr = np.where(score >= q3 + 1.5*iqr)[0]; predictions = np.zeros(nCells); predictions[indIqr] = 1
```

## Evaluation signals

- Output file exists and contains exactly one 0/1 label per input cell (length matches nCells)
- Every label value is binary (only 0 or 1 present)
- Number of cells labeled 1 equals len(np.where(score >= th)[0]) with th = q3 + 1.5*iqr computed from the score vector
- Cells flagged rare should correspond to the highest FiRE scores; no cell below the fence is labeled 1
- On the paper's datasets, the filter should flag rare populations consistent with reported counts (e.g., 727 rare cells in mouse brain data, 4238 in ~68k PBMC data)

## Limitations

- FiRE does not discriminate between an outlier and cells representing minor cell types, so IQR-flagged cells may include technical outliers rather than rare populations
- The threshold is data-driven and depends on the score distribution shape; a skewed distribution can change the number of rare calls substantially
- When doublet rate varies between 1% and 30%, doublets may or may not align with the rare-cell label, limiting use for doublet detection

## Evidence

- [readme] IQR-based thresholding criteria: [evidence span withheld — anchor status: external_doc]
- [readme] threshold computation: "q3 = np.percentile(score, 75)
iqr = stats.iqr(score)
th = q3 + 1.5*iqr"
- [readme] binary predictions: "predictions = np.zeros(data.shape[0])
predictions[indIqr] = 1 #Replace predictions for rare cells with '1'."
- [discussion] FiRE gives scores, others give binary: [evidence span withheld — anchor status: not_found]
- [discussion] outlier limitation: "FiRE, in principle, does not discriminate between an outlier and cells representing minor cell types."
