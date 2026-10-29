---
name: cross-dataset-reference-evaluation
description: Use when you need to evaluate or tune a reference-based cell-type identifier
  (e.g. MarkerCount_Ref) without holding out a dedicated training set, by rotating
  multiple annotated scRNA-seq datasets through test/reference roles.
when_to_use_negative:
- You only have a single annotated dataset and cannot rotate roles — a dedicated train/test
  split is needed instead
- The task is marker-based identification using pre-existing marker lists (use the
  MarkerCount() function rather than cross-dataset reference evaluation)
- Cell populations are poorly clustered, since cluster-basis reassignment presumes
  well-clustered populations
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MarkerCount
  role: Reference-based identifier (MarkerCount_Ref) evaluated in circular cross-dataset
    mode; also supplies marker selection by marker score sm,j
  repo: https://github.com/combio-dku/MarkerCount/tree/master
- name: SCANPY
  role: Reading/processing .h5ad single-cell data during the evaluation workflow
- name: scikit-learn
  role: PCA (15 components) and Gaussian mixture model clustering used in cluster-basis
    reassignment within each cross-evaluation round
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:e834b802776a09e333d154ffe0428cb692ee0dd4b8777e28b85915624bbbfb6c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/combio-dku/MarkerCount
  tools:
  - MarkerCount
  - SCANPY
  - scikit-learn
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-dataset-reference-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Circular cross-evaluation scheme for reference-based MarkerCount in which one annotated scRNA-seq dataset serves as test data while the remaining datasets serve as references, with roles circularly shifted. It is used to tune hyper-parameters (N=18 markers, 0.8 reassignment threshold) and compare identification performance.

## When to use

Use when you need to evaluate or tune a reference-based cell-type identifier (e.g. MarkerCount_Ref) without holding out a dedicated training set, by rotating multiple annotated scRNA-seq datasets through test/reference roles.

## When NOT to use

- You only have a single annotated dataset and cannot rotate roles — a dedicated train/test split is needed instead
- The task is marker-based identification using pre-existing marker lists (use the MarkerCount() function rather than cross-dataset reference evaluation)
- Cell populations are poorly clustered, since cluster-basis reassignment presumes well-clustered populations

## Inputs

- Eight annotated scRNA-seq datasets (GSE84133, GSE85241, GSE100866, GSE93421, ERP114453, SRP218543, GSE72056, GSE103322) with manual cell-type annotations
- MarkerCount python package (MarkerCount_Ref function) with Numpy, Pandas, sklearn, scipy

## Outputs

- Performance-vs-N sweep table (marker_count_sweep.csv) showing N=18 as best for most datasets
- Threshold sensitivity table over 0.1-0.9 reassignment thresholds (threshold_sensitivity_results.csv)
- Per-rotation C, E, EA, CUA, EUA scores against manual annotations

## How to apply

Download the eight annotated datasets (GSE84133, GSE85241, GSE100866, GSE93421, ERP114453, SRP218543, GSE72056, GSE103322) and binarize expression (bij=1 if xij>0). For each dataset role rotation, run MarkerCount_Ref with the held-out dataset as test and the rest as reference: select the top-N markers per cell type using the marker score with occurrence-frequency cutoff fth=0.9 and exponent n=2, obtain cell-type-specific rejection thresholds from the reference data, then identify test cells. Apply cluster-basis reassignment (PCA with 15 components, GMM clustering at Ncell/2 clusters, weighted Mahalanobis distance with rho=0.1) whenever the unassigned portion falls below the threshold being tested (swept 0.1-0.9; best at 0.8). Score each rotation with five measures — correct (C), error (E), erroneously assigned (EA), correctly unassigned (CUA), erroneously unassigned (EUA) — against manual annotations, sweeping N per cell type (fixed to 18 as best for most datasets).

## Related tools

- **MarkerCount** (Reference-based identifier (MarkerCount_Ref) evaluated in circular cross-dataset mode; also supplies marker selection by marker score sm,j) — https://github.com/combio-dku/MarkerCount/tree/master
- **SCANPY** (Reading/processing .h5ad single-cell data during the evaluation workflow)
- **scikit-learn** (PCA (15 components) and Gaussian mixture model clustering used in cluster-basis reassignment within each cross-evaluation round)

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref; df_res = mkrcnt$MarkerCount_Ref( .. arguments .. )
```

## Evaluation signals

- The performance-vs-N sweep should show N=18 yielding the best performance for most datasets, consistent with the reported fixed hyper-parameter choice
- The threshold sensitivity table must span the full 0.1-0.9 range at 0.1 steps with the 0.8 threshold corresponding to best performance
- Each of the eight datasets must have served as the test set exactly once across the circular role shifts
- Rejection thresholds used per rotation must be derived from the reference data (not the test data) for each cell type
- Predictions must be scored on all five measures (C, E, EA, CUA, EUA) against the manual annotations

## Limitations

- Cross-evaluation assumes comparable cell-type composition across datasets; non-standardized cell type taxonomy means researchers must prepare their own marker lists for specific studies
- Proper identification requires well-clustered cell populations, which is not always the case
- For cell types whose target and non-target cells are not separable by normalized marker counts alone (e.g. beta/delta/epsilon/gamma pancreas cells, dendritic cells in blood, B cells in tumors, T cells in lung), results depend on the cluster-basis reassignment step
- It is unclear whether MarkerCount's superior performance in some comparisons stems from binarized gene expression suppressing batch effects

## Evidence
