---
name: rejection-threshold-tuning-scrnaseq
description: Use when calibrating the unassigned-cells portion threshold that triggers
  cluster-basis reassignment in a marker-count-based scRNA-seq cell type identifier,
  and you need empirical justification for the chosen cutoff.
when_to_use_negative:
- Input has no manual annotations, so the five measures (C, E, EA, CUA, EUA) cannot
  be scored against ground truth
- The task is plain cell-type annotation rather than calibration/tuning of the reassignment
  trigger threshold
- Cell populations are not well clustered, since cluster-basis reassignment relies
  on within-cluster expression similarity
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MarkerCount
  role: Count-based cell type identifier whose cluster-basis reassignment threshold
    is swept; identification repeated per threshold value
  repo: https://github.com/combio-dku/MarkerCount
- name: SCANPY
  role: Reading and handling single-cell expression data (.h5ad) for the sweep
- name: scikit-learn
  role: PCA (15 components), Gaussian mixture clustering (Ncell/2 clusters), and Mahalanobis-distance-based
    reassignment machinery
- name: NumPy
  role: Matrix binarization and numerical computation of normalized marker counts
- name: Pandas
  role: Assembling and writing the per-threshold metric table to threshold_sensitivity_results.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:c5d881a9f4176d7137bc20394b4ce959a6d5b86fc9ad96aad5a85bb246898a8d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/combio-dku/MarkerCount
  repo_url: https://github.com/combio-dku/MarkerCount
  tools:
  - MarkerCount
  - NumPy
  - Pandas
  - SCANPY
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# rejection-threshold-tuning-scrnaseq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sweeps MarkerCount's cluster-basis reassignment trigger — the portion-of-unassigned-cells threshold — over 0.1–0.9 in 0.1 steps to empirically justify the fixed 0.8 cutoff that gave the best cell-type identification performance.

## When to use

Use when calibrating the unassigned-cells portion threshold that triggers cluster-basis reassignment in a marker-count-based scRNA-seq cell type identifier, and you need empirical justification for the chosen cutoff.

## When NOT to use

- Input has no manual annotations, so the five measures (C, E, EA, CUA, EUA) cannot be scored against ground truth
- The task is plain cell-type annotation rather than calibration/tuning of the reassignment trigger threshold
- Cell populations are not well clustered, since cluster-basis reassignment relies on within-cluster expression similarity

## Inputs

- Single-cell RNA-seq gene expression matrix (e.g., .h5ad) with manual cell-type annotations as ground truth
- Marker list or pre-annotated reference dataset for marker selection (N=18 markers per cell type, fth=0.9)

## Outputs

- threshold_sensitivity_results.csv: per-threshold metric table spanning 0.1–0.9 at 0.1 steps with columns correct (C), error (E), erroneously assigned (EA), correctly unassigned (CUA), erroneously unassigned (EUA)
- Selected optimal reassignment threshold (expected: 0.8)

## How to apply

First run the base MarkerCount identification: binarize the gene expression matrix (bij=1 if xij>0), select N=18 markers per cell type with occurrence-frequency cutoff fth=0.9, compute normalized marker counts, apply cell-type-specific rejection thresholds tm, reduce to 15 PCA components, and cluster with a Gaussian mixture model at Ncell/2 clusters. Then repeat the full identification, including cluster-basis reassignment of unassigned cells using weighted Mahalanobis distance (regularization rho=0.1 of average variances), for each reassignment portion threshold in {0.1, 0.2, ..., 0.9}. Score each run against manual annotations using the five measures correct (C), error (E), erroneously assigned (EA), correctly unassigned (CUA), and erroneously unassigned (EUA). Assemble the per-threshold metric table and verify that the 0.8 threshold exhibits the best performance, consistent with the paper's finding that 0.8 was chosen from the 0.1–0.9 sweep at 0.1 steps.

## Related tools

- **MarkerCount** (Count-based cell type identifier whose cluster-basis reassignment threshold is swept; identification repeated per threshold value) — https://github.com/combio-dku/MarkerCount
- **SCANPY** (Reading and handling single-cell expression data (.h5ad) for the sweep)
- **scikit-learn** (PCA (15 components), Gaussian mixture clustering (Ncell/2 clusters), and Mahalanobis-distance-based reassignment machinery)
- **NumPy** (Matrix binarization and numerical computation of normalized marker counts)
- **Pandas** (Assembling and writing the per-threshold metric table to threshold_sensitivity_results.csv)

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref, MarkerCount
```

## Evaluation signals

- Resulting table spans the full 0.1–0.9 threshold range in exact 0.1 increments (9 rows)
- The 0.8 threshold row corresponds to the best-performing combination of C, E, EA, CUA, EUA measures, matching the paper's reported finding
- Every sweep run uses the same fixed hyperparameters: N=18 markers, fth=0.9, 15 PCA components, Ncell/2 GMM clusters, Mahalanobis regularization rho=0.1
- Metrics are computed against manual annotations for all five measures, not just overall accuracy
- Output file threshold_sensitivity_results.csv is produced with per-threshold rows

## Limitations

- For some cell types (beta/delta/epsilon/gamma pancreas cells, dendritic cells in blood, B cells in tumors, T cells in lung), identification via normalized marker counts alone yields considerable errors, stressing the reassignment step
- Proper identification requires well-clustered cell populations, which is not always the case
- The 0.8 optimum is empirical for the datasets in this work; no guarantee it transfers to other tissues or taxonomies, which are not standardized

## Evidence
