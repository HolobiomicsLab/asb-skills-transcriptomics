---
name: classification-accuracy-comparison
description: Use when you have per-cell gene set scores (e.g., summed_up scores from
  gssnng) for both unsmoothed and NNG-smoothed AnnData objects plus per-cell condition
  labels, and need to quantify whether smoothing improves a simple median-cut-point
  classifier.
when_to_use_negative:
- When no per-cell condition/treatment labels exist to serve as ground truth for a
  supervised accuracy comparison
- When the goal is differential gene set scoring or visualization rather than evaluating
  whether smoothing improves classification — use gssnng's per-group scoring outputs
  directly instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: gssnng
  role: Produces the per-cell gene set scores (summed_up method) on unsmoothed and
    nearest-neighbor-smoothed AnnData that are thresholded at their medians for classification
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Reads the h5ad AnnData objects and provides the neighborhood graph used for
    smooth_mode='connectivity' smoothing with recomputed per-group neighbors
- name: AnnData
  role: Container format for the cell-by-gene matrices, smoothed counts, and obs metadata
    holding LPS treatment labels
- name: Python
  role: Implements the median cut-point labeling, accuracy computation, and CSV report
    generation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:6cacff7edb6f3e87aa377c8cd4cc4e289c7aac0419bbeaf58e8d9b1e82c4e095
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# classification-accuracy-comparison

## Summary

Compares binary classification accuracy of gene set scores from unsmoothed vs nearest-neighbor-smoothed single-cell data by thresholding each score distribution at its median and matching predicted labels to true treatment labels (e.g., LPS status in the Hagai phagocyte dataset).

## When to use

Use when you have per-cell gene set scores (e.g., summed_up scores from gssnng) for both unsmoothed and NNG-smoothed AnnData objects plus per-cell condition labels, and need to quantify whether smoothing improves a simple median-cut-point classifier.

## When NOT to use

- When no per-cell condition/treatment labels exist to serve as ground truth for a supervised accuracy comparison
- When the goal is differential gene set scoring or visualization rather than evaluating whether smoothing improves classification — use gssnng's per-group scoring outputs directly instead

## Inputs

- Per-cell summed_up gene set score table for the unsmoothed AnnData (e.g., adata.obsm['gssnng_estimate'] or equivalent CSV)
- Per-cell summed_up gene set score table for the smoothed AnnData (smooth_mode='connectivity')
- Per-cell LPS treatment label metadata table indexed by cell barcode

## Outputs

- lps_accuracy_report.csv with per-condition classification accuracies (expected ~0.88 unsmoothed, ~0.97 smoothed)
- Pass/fail flag indicating whether smoothed accuracy exceeds unsmoothed accuracy

## How to apply

Load the per-cell summed_up gene set score tables produced upstream for the unsmoothed AnnData and the smoothed AnnData (smooth_mode='connectivity' on the nearest neighbor graph, with recompute_neighbors set so neighbors are recomputed within each groupby group), and load the LPS treatment labels from the cell metadata. For each condition, compute the median of the summed_up scores across cells and use that median as the binary cut point to assign predicted LPS treatment status per cell. Compare predicted labels to true LPS labels per condition and compute classification accuracy. Write the per-condition accuracies to lps_accuracy_report.csv; accept the run if the smoothed accuracy exceeds the unsmoothed accuracy (published Section 7.2 values: 0.97 smoothed vs 0.88 unsmoothed), otherwise flag the run as failing the paper's qualitative acceptance check.

## Related tools

- **gssnng** (Produces the per-cell gene set scores (summed_up method) on unsmoothed and nearest-neighbor-smoothed AnnData that are thresholded at their medians for classification) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Reads the h5ad AnnData objects and provides the neighborhood graph used for smooth_mode='connectivity' smoothing with recomputed per-group neighbors)
- **AnnData** (Container format for the cell-by-gene matrices, smoothed counts, and obs metadata holding LPS treatment labels)
- **Python** (Implements the median cut-point labeling, accuracy computation, and CSV report generation)

## Examples

```
import pandas as pd; med = scores['summed_up'].median(); pred = (scores['summed_up'] > med).astype(int); acc = (pred == labels).mean()
```

## Evaluation signals

- Unsmoothed classification accuracy falls near 0.88 (published Section 7.2 value)
- Smoothed classification accuracy falls near 0.97 and exceeds the unsmoothed accuracy; if not, flag the run as failing the paper's qualitative acceptance check
- Predicted label counts per condition are consistent with the number of cells in the score tables (no cells dropped during median thresholding)
- lps_accuracy_report.csv contains exactly one accuracy row per condition (unsmoothed, smoothed) with values in a plausible range around the published values

## Limitations

- A median cut point is a simplistic classifier; accuracy here is a qualitative acceptance check, not a rigorous model evaluation
- The smoothed accuracy depends on upstream smoothing choices (smooth_mode='connectivity', recompute_neighbors within groupby groups); different settings may change the accuracy gap
- Ranking-based conclusions assume the median threshold cleanly separates the two treatment classes; ties at the median can distort label assignment

## Evidence

- [intro] The GSSNNG method is based on using the nearest neighbor graph of cells for data smoothing.: "The GSSNNG method is based on using the nearest neighbor graph of cells for data smoothing."
- [abstract] smoothing rationale: [evidence span withheld — anchor status: not_found]
- [readme] summed_up scoring method: "**summed_up**:      Sum up the ranks or counts."
- [readme] smooth_mode connectivity weighting: "`adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
