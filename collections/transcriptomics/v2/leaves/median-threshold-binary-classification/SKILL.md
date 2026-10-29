---
name: median-threshold-binary-classification
description: Use when you have per-cell gene set scores for two experimental conditions
  and want a simple median cut point to quantify whether a smoothing or scoring change
  improves binary label prediction.
when_to_use_negative:
- The goal is multiclass or probabilistic classification — a single median cut point
  only supports binary label assignment at one threshold.
- Gene set scores have not yet been computed for both the unsmoothed and smoothed
  conditions, so no comparison of smoothing benefit is possible.
- The condition labels are continuous rather than binary, making a median threshold
  cut point inappropriate.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: gssnng
  role: Produces the per-cell summed_up gene set scores (median of which forms the
    cut point) on both unsmoothed and nearest-neighbor-smoothed AnnData
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Reads the h5ad AnnData objects and supplies the cell metadata (obs) with LPS
    treatment labels
- name: AnnData
  role: Container format holding the count matrix, neighborhood graph, and per-cell
    gene set scores being classified
- name: Python
  role: Language in which the median cut point, label assignment, and accuracy computation
    are implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:b89b378d60c197eaa7a10b921563f2c3b47ab3b1ce974f6108639cef9d97161e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# median-threshold-binary-classification

## Summary

A validation skill that binarizes per-cell gene set scores (e.g. gssnng summed_up scores) at their median to predict a binary condition label (LPS treatment status), then computes classification accuracy to test whether nearest neighbor smoothing improves separation.

## When to use

Use when you have per-cell gene set scores for two experimental conditions and want a simple median cut point to quantify whether a smoothing or scoring change improves binary label prediction.

## When NOT to use

- The goal is multiclass or probabilistic classification — a single median cut point only supports binary label assignment at one threshold.
- Gene set scores have not yet been computed for both the unsmoothed and smoothed conditions, so no comparison of smoothing benefit is possible.
- The condition labels are continuous rather than binary, making a median threshold cut point inappropriate.

## Inputs

- Per-cell summed_up gene set score table for the unsmoothed AnnData (e.g. adata.obsm['gssnng_estimate'] or equivalent per-cell score export)
- Per-cell summed_up gene set score table for the smoothed AnnData (smooth_mode='connectivity', recompute_neighbors within groupby groups)
- Cell metadata table with binary LPS treatment labels per cell

## Outputs

- lps_accuracy_report.csv with per-condition classification accuracies (expected ~0.88 unsmoothed, ~0.97 smoothed)

## How to apply

Load the per-cell summed_up gene set score tables produced by gssnng for both the unsmoothed AnnData and the smoothed AnnData (smooth_mode='connectivity' on the nearest neighbor graph with recomputed neighbors within each groupby group), and load the true LPS treatment labels from the cell metadata. For each condition, compute the median of the summed_up scores across all cells and use that median as the binary cut point, assigning each cell a predicted LPS treatment status based on which side of the median its score falls. Compare the predicted labels to the true LPS labels per condition and compute classification accuracy. In the Hagai phagocyte dataset this yields approximately 0.88 accuracy on unsmoothed data and 0.97 on smoothed data, so the key decision point is whether the smoothed condition's accuracy exceeds the unsmoothed condition's; if it does not, the run fails the paper's qualitative acceptance check (Section 7.2).

## Related tools

- **gssnng** (Produces the per-cell summed_up gene set scores (median of which forms the cut point) on both unsmoothed and nearest-neighbor-smoothed AnnData) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Reads the h5ad AnnData objects and supplies the cell metadata (obs) with LPS treatment labels)
- **AnnData** (Container format holding the count matrix, neighborhood graph, and per-cell gene set scores being classified)
- **Python** (Language in which the median cut point, label assignment, and accuracy computation are implemented)

## Examples

```
median = scores['summed_up'].median(); pred = (scores['summed_up'] > median).astype(int); acc = (pred == labels).mean()
```

## Evaluation signals

- Unsmoothed-data classification accuracy falls in a plausible range around 0.88 (published Section 7.2 value).
- Smoothed-data classification accuracy falls in a plausible range around 0.97 and strictly exceeds the unsmoothed accuracy; if not, flag the run as failing the paper's qualitative acceptance check.
- The median cut point used per condition equals the observed median of the summed_up scores across all cells in that condition.
- Every cell receives exactly one predicted label, and the count of predicted labels matches the count of true LPS labels.
- Output file lps_accuracy_report.csv contains one row per condition (unsmoothed, smoothed) with an accuracy column.

## Limitations

- Median-threshold accuracy is a coarse validation metric; it assumes the median score aligns with a meaningful decision boundary and does not tune or cross-validate the cut point.
- The expected accuracies (0.88 vs 0.97) are specific to the Hagai phagocyte dataset and the summed_up scoring method; other datasets, gene sets, or scoring methods may not reproduce this qualitative ordering.
- Smoothing must be configured as in the paper (smooth_mode='connectivity' with neighbors recomputed within each groupby group); other smoothing settings may change the accuracy comparison.

## Evidence

- [methods] summed_up scoring at median cut point: "summed_up:      Sum up the ranks or counts."
- [abstract] smoothing improves scores: "The GSSNNG method is based on using the nearest neighbor graph of cells for data smoothing."
- [readme] median_score option: "median_score:   Median of counts or ranks"
- [readme] smooth_mode parameter: "`adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
