---
name: lineage-bifurcation-analysis
description: Use when you have a scRNA-seq count matrix with pre-defined cell clusters
  and want to reconstruct developmental lineages whose start point is chosen objectively
  as the most pluripotent cells rather than arbitrarily.
when_to_use_negative:
- Your goal is only to score pluripotency/annotate cells (use plain ORIGINS activity()
  or ORIGINS2/PyOrigins) without reconstructing a trajectory
- The dataset has no plausible stem/progenitor cluster or known cell-type labels,
  so no root cluster exists in which to select highest-activity cells
- Expression matrix is not gene-by-cell normalized data compatible with activity(),
  e.g., it is already a per-cell summary table
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: ORIGINS
  role: Computes per-cell differentiation PPIN activity used to select the trajectory
    root (highest-activity cells within the origin cluster)
  repo: https://github.com/danielasenraoka/ORIGINS
- name: R
  role: 'Runtime environment: ORIGINS is an open source R package installed via remotes::install_github;
    Slingshot/tradeSeq run in the same R session'
- name: remotes
  role: Used to install the ORIGINS package from GitHub before computing activity
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:5ac426d01d121531a59c0659bf1afaf8c2d3e35fc2450ed014c24063cb1dfffb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - ORIGINS
  - R
  - remotes
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# lineage-bifurcation-analysis

## Summary

Root Slingshot pseudotime trajectory inference at cells with the highest ORIGINS differentiation PPIN activity within a stem/progenitor cluster, then verify that the inferred trajectory bifurcates from that origin into distinct terminal cell types (e.g., basal → L1/L2 in breast epithelium).

## When to use

Use when you have a scRNA-seq count matrix with pre-defined cell clusters and want to reconstruct developmental lineages whose start point is chosen objectively as the most pluripotent cells rather than arbitrarily.

## When NOT to use

- Your goal is only to score pluripotency/annotate cells (use plain ORIGINS activity() or ORIGINS2/PyOrigins) without reconstructing a trajectory
- The dataset has no plausible stem/progenitor cluster or known cell-type labels, so no root cluster exists in which to select highest-activity cells
- Expression matrix is not gene-by-cell normalized data compatible with activity(), e.g., it is already a per-cell summary table

## Inputs

- scRNA-seq count/normalized expression matrix (rows = genes, columns = cells), e.g., GSE113197 breast epithelial cells
- Pre-defined cell cluster labels (e.g., basal, L1, L2)
- differentiation_edges: PPIN edge data frame bundled inside the ORIGINS package

## Outputs

- Per-cell differentiation activity scores (diff_activity vector)
- Slingshot inferred lineages rooted at the highest-activity cells (Lineage 1: origin → L1; Lineage 2: origin → L2)
- Pseudotime values per cell along each lineage
- Trajectory figure (e.g., slingshot_trajectory_breast.png); optionally tradeSeq lineage-specific DE results

## How to apply

Load the scRNA-seq count matrix in R (rows = genes, columns = cells; e.g., GSE113197 breast epithelial cells) and compute per-cell differentiation activity with ORIGINS via diff_activity <- activity(expression_matrix, differentiation_edges), where differentiation_edges ships inside the package. Within the cluster representing the putative origin (e.g., the basal cluster), select the cells with the highest differentiation PPI network activity scores and designate them as the Slingshot root, since ORIGINS was created to identify stem and progenitor cells as trajectory origins. Run Slingshot on the clustered data with this root and inspect the lineage structure: a correct result is a bifurcating trajectory from the origin through a shared intermediate state into the terminal clusters (Lineage 1: basal → L1; Lineage 2: basal → L2). Optionally apply tradeSeq to the two lineages to test lineage-specific differential expression along pseudotime and export the trajectory figure (e.g., slingshot_trajectory_breast.png).

## Related tools

- **ORIGINS** (Computes per-cell differentiation PPIN activity used to select the trajectory root (highest-activity cells within the origin cluster)) — https://github.com/danielasenraoka/ORIGINS
- **R** (Runtime environment: ORIGINS is an open source R package installed via remotes::install_github; Slingshot/tradeSeq run in the same R session)
- **remotes** (Used to install the ORIGINS package from GitHub before computing activity)

## Examples

```
library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- The computed diff_activity vector has one value per cell, normalized by the number of PPIN edges so scores are comparable
- The trajectory root coincides with the cells that have the highest activity scores within the basal/progenitor cluster
- The inferred Slingshot lineage structure is bifurcating: a single origin branching through a shared intermediate state into exactly two terminal lineages (basal → L1 and basal → L2)
- The exported trajectory figure (slingshot_trajectory_breast.png) shows pseudotime increasing from the root to L1/L2 tips
- Optional tradeSeq test identifies genes with lineage-specific differential expression along the two pseudotimes

## Limitations

- Root placement depends entirely on ORIGINS activity ranking; mis-scoring of pluripotency propagates to a wrong trajectory origin
- The demonstrated workflow is validated only on healthy human datasets (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383); behavior on diseased or non-human data is not shown
- ORIGINS is specific to the differentiation PPIN; for other biological processes use the extended ORIGINS2 or PyOrigins implementation

## Evidence

- [readme] This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference.: "This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference."
- [readme] diff_activity <- activity(expression_matrix, differentiation_edges): "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [abstract] We also illustrate a complete workflow from the count matrix to trajectory inference using the breast data set.: "We also illustrate a complete workflow from the count matrix to trajectory inference using the breast data set."
- [methods] remotes::install_github("danielasenraoka/ORIGINS"): "remotes::install_github("danielasenraoka/ORIGINS")"
- [supplementary] GSE113197 for breast epithelial cells: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
