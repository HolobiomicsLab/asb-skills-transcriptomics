---
name: cross-dataset-cell-type-transfer
description: Use when a pre-annotated reference scRNA-seq dataset (e.g., .h5ad from
  GEO) must be used to label cells in a different test dataset whose taxonomy may
  contain unseen cell types, and errors from noise/batch effects need suppression.
when_to_use_negative:
- Each cell population is not well clustered in the test data — the article notes
  'for proper identification, each cell populations should be well clustered, even
  though this is not always the case'
- The task requires a standardized, prebuilt cell-type taxonomy — none exists; researchers
  must prepare their own marker lists and taxonomy
- The input is already annotated per-cell with a trusted shared taxonomy and no cross-dataset
  generalization is needed
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MarkerCount
  role: Core toolkit; MarkerCount_Ref() performs reference-based transfer with binarized
    marker counting, rejection thresholds, and cluster-basis reassignment; MarkerCount()
    supports marker-list mode
  repo: https://github.com/combio-dku/MarkerCount
- name: SCANPY
  role: Preprocessing and visualization of scRNA-seq data (results display); required
    workflow dependency alongside numpy/pandas/sklearn/scipy
- name: scmap
  role: Baseline reference-based identifier (cell and cluster modes) used for comparison
    in circular cross-evaluation
- name: CHETAH
  role: Baseline hierarchical reference-based identifier used for comparison in circular
    cross-evaluation
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:fe402342affff23a99a3cd201173adb77da299941aee843d372b4d520987e665
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
  - CHETAH
  - MarkerCount
  - SCANPY
  - numpy
  - pandas
  - scikit-learn
  - scipy
  - scmap
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cross-dataset-cell-type-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Transfer cell-type annotations across scRNA-seq datasets using MarkerCount's reference-based mode, which binarizes expression, counts expressed markers instead of using expression levels, and applies a rejection threshold plus cluster-basis reassignment to handle unknown populations and batch effects.

## When to use

Use when a pre-annotated reference scRNA-seq dataset (e.g., .h5ad from GEO) must be used to label cells in a different test dataset whose taxonomy may contain unseen cell types, and errors from noise/batch effects need suppression.

## When NOT to use

- Each cell population is not well clustered in the test data — the article notes 'for proper identification, each cell populations should be well clustered, even though this is not always the case'
- The task requires a standardized, prebuilt cell-type taxonomy — none exists; researchers must prepare their own marker lists and taxonomy
- The input is already annotated per-cell with a trusted shared taxonomy and no cross-dataset generalization is needed

## Inputs

- Pre-annotated reference scRNA-seq dataset (e.g., .h5ad; GSE100866 CBMC 8K or GSE93421 PBMC 68K)
- Unlabeled test scRNA-seq dataset with manual annotations for evaluation (e.g., GSE72056 Melanoma 5K, GSE103322 HeadNeck 6K, GSE84133/GSE85241 pancreas, ERP114453/SRP218543 lung)
- Harmonized broad cell-type taxonomy mapping (e.g., T cell subtypes collapsed to 'T cell')
- Marker lists from Panglao DB or CellMarker DB (for marker-based mode)

## Outputs

- Per-cell predicted labels with 'unassigned' for rejected cells
- Five-measure evaluation metrics per dataset pair: C, E, EA, CUA, EUA (values in 0–1)
- Aggregate cross-evaluation benchmark table (e.g., tumor_cross_evaluation_metrics.csv)

## How to apply

Load the annotated reference and test datasets (e.g., CBMC 8K GSE100866, PBMC 68K GSE93421, pancreas GSE84133/GSE85241, lung ERP114453/SRP218543, melanoma GSE72056, head-neck GSE103322) and harmonize annotation granularity to a shared broad taxonomy before transfer. Train MarkerCount_Ref on the reference: binarize expression (bij=1 if xij>0), compute marker scores sm,j with exponent n=2, filter candidates by occurrence frequency threshold fth=0.9, and select the top N=18 markers per cell type. Obtain per-type rejection thresholds tm from the reference via the non-parametric FPR-based criterion at a target false positive rate, so cells from populations absent in the reference stay unassigned rather than being forced into a wrong label. Run the test phase: compute normalized marker counts, reject cells below threshold, then perform PCA with 15 components, Gaussian-mixture clustering, and Mahalanobis-distance cluster-basis reassignment (variance regularization rho=0.1; reassign only when the unassigned fraction is below 0.8). When marker counts per cell type are uneven, apply the penalty weight w=1+3e^-(n-1)/2-1. The binarization rationale is to suppress batch effects, since raw expression profiles are subject to noise and batch effects that compromise reference-based methods.

## Related tools

- **MarkerCount** (Core toolkit; MarkerCount_Ref() performs reference-based transfer with binarized marker counting, rejection thresholds, and cluster-basis reassignment; MarkerCount() supports marker-list mode) — https://github.com/combio-dku/MarkerCount
- **SCANPY** (Preprocessing and visualization of scRNA-seq data (results display); required workflow dependency alongside numpy/pandas/sklearn/scipy)
- **scmap** (Baseline reference-based identifier (cell and cluster modes) used for comparison in circular cross-evaluation)
- **CHETAH** (Baseline hierarchical reference-based identifier used for comparison in circular cross-evaluation)

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref; df_res = MarkerCount_Ref(ref_data, test_data, n_markers=18, fth=0.9)
```

## Evaluation signals

- Output metric table contains fields C, E, EA, CUA, and EUA per dataset, each a value between 0 and 1
- For blood-reference-to-tumor transfer (GSE72056, GSE103322), correctly-unassigned tumor cells (CUA) are high while erroneously-assigned cells (EA) are low, matching the deposited figshare results
- Reproduced C/E/EA/CUA/EUA values for each identifier–dataset pair match the deposited results (figshare MarkerCount_cell_type_identification_results) within tolerance
- The unassigned fraction before reassignment is checked against the 0.8 cutoff, and cluster reassignment is only applied below it
- Marker selection yields exactly N=18 markers per cell type after fth=0.9 frequency filtering

## Limitations

- It is unclear whether MarkerCount's superior performance in some benchmarks stems from binarized gene expression suppressing batch effect
- Handling not-clearly-separable sub-populations, such as monocytes, macrophage, and dendritic cells, requires further studies
- Cell types and taxonomy are not standardized, so researchers must prepare their own marker lists
- Proper identification requires well-clustered cell populations, which is not always the case
- For some cell types (beta/delta/epsilon/gamma cells in pancreas, dendritic cells in blood, B cells in tumors, T cells in lung), identification via normalized marker counts alone yields considerable errors, requiring the clustering/reassignment step

## Evidence
