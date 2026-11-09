---
name: rejection-threshold-calibration
description: Use when training a reference-based identifier (MarkerCount_Ref) on a
  pre-annotated reference dataset whose taxonomy does not cover the test cells, and
  you need non-parametric per-cell-type thresholds tm on normalized marker counts
  for unknown-population rejection.
when_to_use_negative:
- The reference taxonomy already contains the cell types present in the test data
  (no unknown populations to reject), so rejection thresholds trade off sensitivity
  without benefit
- You are running marker-based mode (MarkerCount or HiCAT) with pre-existing marker
  lists and no pre-annotated reference from which to learn thresholds
- Cell populations are not well clustered, since cluster-basis reassignment after
  thresholding relies on within-cluster gene expression similarity
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MarkerCount
  role: Provides MarkerCount_Ref for training the reference model, computing normalized
    marker counts, learning rejection thresholds, and performing cluster-basis reassignment
  repo: https://github.com/combio-dku/MarkerCount
- name: SCANPY
  role: Loads .h5ad single-cell datasets and supports visualization of results (not
    required by the MarkerCount functions themselves)
  repo: https://github.com/combio-dku/MarkerCount
- name: scikit-learn
  role: Supplies PCA, Gaussian mixture model clustering, and Mahalanobis distance
    used in the cluster-basis reassignment step
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:5c9b00b6dff09cba023785ecc545ef36e31507c3a9477e42b3c8fafd7a6a85d8
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
  - SCANPY
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# rejection-threshold-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calibrates MarkerCount's conservative rejection thresholds on normalized marker counts so that cells from populations absent from the reference taxonomy (e.g., tumor cells when training on blood references) are left unassigned (CUA) instead of erroneously assigned, followed by cluster-basis reassignment to limit unnecessarily unassigned cells.

## When to use

Use when training a reference-based identifier (MarkerCount_Ref) on a pre-annotated reference dataset whose taxonomy does not cover the test cells, and you need non-parametric per-cell-type thresholds tm on normalized marker counts for unknown-population rejection.

## When NOT to use

- The reference taxonomy already contains the cell types present in the test data (no unknown populations to reject), so rejection thresholds trade off sensitivity without benefit
- You are running marker-based mode (MarkerCount or HiCAT) with pre-existing marker lists and no pre-annotated reference from which to learn thresholds
- Cell populations are not well clustered, since cluster-basis reassignment after thresholding relies on within-cluster gene expression similarity

## Inputs

- Pre-annotated reference gene expression matrix (e.g., GSE100866 CBMC 8K or GSE93421 PBMC 68K .h5ad) with broad cell-type annotations
- Learned per-cell-type marker sets (up to 18 markers per type) and target/non-target occurrence frequencies
- Target false positive rate p for the FPR-based threshold criterion

## Outputs

- Per-cell-type rejection thresholds tm within [0,1], saved alongside marker lists to markers_and_thresholds
- Per-cell assignment labels with unassigned (unknown) calls from the test phase (e.g., GSE72056 Melanoma 5K, GSE103322 HeadNeck 6K)
- Per-dataset metric table tumor_cross_evaluation_metrics.csv with fields C, E, EA, CUA, EUA in [0,1]

## How to apply

After marker discovery (binarize reference expression with bij=1 if xij>0, marker score sm,j with exponent n=2, fth=0.9, top N=18 markers per cell type), compute normalized marker counts yi,m on the reference cells. For each cell type m, sort yi,m in descending order and find the minimum threshold tm such that the false positive rate FPR(t) <= p using the non-parametric true-positive-set intersection criterion; every threshold must lie within the [0,1] range of normalized marker counts. In the test phase, assign a cell only if its maximum normalized marker count exceeds the cell type's tm, rejecting uncertain cells conservatively. Because a conservative threshold can leave many correctly identified cells unassigned, run PCA with 15 components and Gaussian mixture model clustering, then Mahalanobis-distance cluster-basis reassignment (variance regularization rho=0.1 of average variances), performing reassignment only if the portion of unassigned cells is below 0.8 (best in the 0.1–0.9 range). Evaluate with the five measures C, E, EA, CUA, EUA to confirm unknown populations are correctly unassigned rather than erroneously assigned.

## Related tools

- **MarkerCount** (Provides MarkerCount_Ref for training the reference model, computing normalized marker counts, learning rejection thresholds, and performing cluster-basis reassignment) — https://github.com/combio-dku/MarkerCount
- **SCANPY** (Loads .h5ad single-cell datasets and supports visualization of results (not required by the MarkerCount functions themselves)) — https://github.com/combio-dku/MarkerCount
- **scikit-learn** (Supplies PCA, Gaussian mixture model clustering, and Mahalanobis distance used in the cluster-basis reassignment step)

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref; df_res = MarkerCount_Ref(reference_data, test_data, marker_list, ...)
```

## Evaluation signals

- Every learned threshold tm lies within the [0,1] range of normalized marker counts
- The per-cell-type entry in markers_and_thresholds contains up to 18 markers per reference cell type
- On tumor test sets evaluated with a blood reference, CUA is high while E and EA are low, matching the deposited figshare reference results
- tumor_cross_evaluation_metrics.csv exists and contains C, E, EA, CUA, EUA fields with values between 0 and 1
- Cluster reassignment step runs only when the portion of unassigned cells is below 0.8

## Limitations

- Proper identification requires each cell population to be well clustered, which is not always the case
- For some cell types (e.g., beta/delta/epsilon/gamma cells in pancreas, dendritic cells in blood, B cells in tumors, T cells in lung), identification via normalized marker counts alone would yield considerable errors
- Handling not-clearly-separable sub-populations such as monocytes, macrophages, and dendritic cells requires further studies

## Evidence

- [intro] conservative rejection threshold: [evidence span withheld — anchor status: not_found]
