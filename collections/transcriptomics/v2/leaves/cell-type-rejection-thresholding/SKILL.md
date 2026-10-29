---
name: cell-type-rejection-thresholding
description: Use when annotating single-cell RNA-seq cells via marker counting and
  you must conservatively reject low-confidence assignments (possible unknown cell
  types) before downstream cluster reassignment.
when_to_use_negative:
- All cell populations are well separated by normalized marker counts alone, so no
  rejection/reassignment stage is needed (e.g. clearly separable pancreas or blood
  populations)
- The task is hierarchical marker-based annotation of immune subtypes — use the successor
  tool HiCAT instead of the MarkerCount rejection pipeline
- Cell populations are not well clustered, since cluster-basis reassignment assumes
  each population forms a coherent cluster
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MarkerCount
  role: Implements the marker counting, rejection thresholding (MarkerCount() and
    MarkerCount_Ref()), and cluster-basis reassignment
  repo: https://github.com/combio-dku/MarkerCount
- name: SCANPY
  role: Loading/manipulating .h5ad single-cell data and displaying results around
    the MarkerCount pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:349142c6865165adeaad6e03c30f00edbd82220704bf5dcc94cddf315f7e20ec
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cell-type-rejection-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A conservative rejection-threshold strategy from MarkerCount that labels cells as 'unassigned' when their normalized marker count falls below a cell-type-specific threshold, deferring ambiguous cells to cluster-basis reassignment instead of forcing a wrong annotation.

## When to use

Use when annotating single-cell RNA-seq cells via marker counting and you must conservatively reject low-confidence assignments (possible unknown cell types) before downstream cluster reassignment.

## When NOT to use

- All cell populations are well separated by normalized marker counts alone, so no rejection/reassignment stage is needed (e.g. clearly separable pancreas or blood populations)
- The task is hierarchical marker-based annotation of immune subtypes — use the successor tool HiCAT instead of the MarkerCount rejection pipeline
- Cell populations are not well clustered, since cluster-basis reassignment assumes each population forms a coherent cluster

## Inputs

- Binarized single-cell RNA-seq expression matrix (cells x genes, .h5ad or similar)
- Cell-type marker sets or marker matrix (.csv) with N=18 markers per cell type (reference-based mode)
- Cell-type-specific rejection thresholds tm derived from reference data

## Outputs

- cell_assignments.csv with one label per cell drawn from the marker-set cell types plus 'unassigned'
- Set of rejected (unassigned) cells available for cluster-basis reassignment

## How to apply

Binarize the test expression matrix (bij=1 if xij>0) and compute the normalized marker count yi,m for each cell type m as the fraction of that cell type's marker set expressed in the cell (N=18 markers per cell type in reference-based mode). Perform an initial assignment by argmax_m(yi,m), but accept the label only if the maximum marker count is at or above the cell-type-specific rejection threshold tm; otherwise mark the cell 'unassigned'. After PCA to 15 components and Gaussian mixture clustering with Ncell/2 clusters, reassign unassigned cells via weighted Mahalanobis distance (covariance regularization rho=0.1, size weight 1/pi_m) only in clusters whose portion of unassigned cells is below 0.8 — a value chosen as it exhibited the best performance in the range 0.1–0.9. The rationale is that a conservative threshold potentially yields many 'unassigned' cells, but cluster-wise reassignment minimizes unnecessarily unassigned cells while still allowing unknown populations to remain rejected.

## Related tools

- **MarkerCount** (Implements the marker counting, rejection thresholding (MarkerCount() and MarkerCount_Ref()), and cluster-basis reassignment) — https://github.com/combio-dku/MarkerCount
- **SCANPY** (Loading/manipulating .h5ad single-cell data and displaying results around the MarkerCount pipeline)

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref; df_res = MarkerCount_Ref(.. arguments ..)
```

## Evaluation signals

- cell_assignments.csv exists and contains exactly one label per cell, with labels drawn from the marker-set cell types plus 'unassigned'
- Cells with maximum normalized marker count below their cell-type-specific threshold tm are labeled 'unassigned', not force-assigned
- Reassignment is applied only in clusters where the portion of unassigned cells is below the 0.8 threshold
- Normalized marker counts yi,m are computed from binarized expression (bij=1 iff xij>0) over the N=18 markers per cell type
- The fraction of conservatively unassigned cells is reduced by cluster-basis reassignment without increasing erroneous assignments (the paper's known-vs-unknown tradeoff metric)

## Limitations

- Handling not-clearly-separable sub-populations, such as monocytes, macrophages, and dendritic cells, is a greater issue, which requires further studies
- For proper identification, each cell population should be well clustered, even though this is not always the case
- The rejection threshold involves a tradeoff between erroneously unassigned and erroneously assigned measures
- Cell types and taxonomy are not standardized; researchers should prepare their own list of markers for the specific study and objective

## Evidence

- [methods] Binarize gene expression for rejection decisions: "We first convert these to binary indicator bij∈{0,1}, which indicates whether the gene is expressed in that cell or not, i.e., bij=1 if xij>0 or 0 otherwise."
- [intro] Conservative rejection threshold rationale: [evidence span withheld — anchor status: not_found]
- [methods] 0.8 reassignment threshold tuning: "Reassignment is performed if the portion of unassigned cells is below a certain threshold, say 0.8. This value was chosen as it exhibited the best performance in the range of 0.1–0.9 at 0.1 steps."
- [results] Two-stage reject-then-reassign strategy: "MarkerCount sets a rejection threshold for the normalized marker counts conservatively at first and uses cluster-basis cell type reassignment and correction utilizing similarity within a cluster."
