---
name: pca-gmm-clustering-of-scrnaseq
description: Use when a marker-count-based cell type identifier leaves many cells
  unassigned and you need to reassign them by within-cluster expression similarity,
  or when reproducing the 0.1-0.9 rejection/reassignment threshold sweep.
when_to_use_negative:
- Cell populations are not well clustered — the article notes that for proper identification
  each cell population should be well clustered, even though this is not always the
  case
- You need to distinguish not-clearly-separable sub-populations such as monocytes,
  macrophage, and dendritic cells, which the authors flag as requiring further study
- All cells are already confidently assigned by the marker-count step and the portion
  of unassigned cells is not below the reassignment threshold
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MarkerCount
  role: Hosts the cluster-basis cell type reassignment pipeline that consumes the
    PCA+GMM clusters; reassignment triggered when unassigned portion is below threshold
    (0.8)
  repo: https://github.com/combio-dku/MarkerCount
- name: scikit-learn
  role: Provides PCA (15 components) and the Gaussian mixture model used for clustering
    prior to reassignment
- name: SCANPY
  role: Used for large-scale single-cell gene expression data analysis and reading
    .h5ad inputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:54d8a6ca782dce207fd4e8d77621816bfa63b9dc7bc6141190e41d1149061496
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
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# PCA + Gaussian mixture model clustering for cluster-basis cell reassignment in scRNA-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reduces single-cell RNA-seq expression data to 15 principal components and fits a Gaussian mixture model with Ncell/2 clusters to group cells by gene-expression similarity, enabling MarkerCount's cluster-basis reassignment of unassigned cells using a weighted Mahalanobis distance.

## When to use

Use when a marker-count-based cell type identifier leaves many cells unassigned and you need to reassign them by within-cluster expression similarity, or when reproducing the 0.1-0.9 rejection/reassignment threshold sweep.

## When NOT to use

- Cell populations are not well clustered — the article notes that for proper identification each cell population should be well clustered, even though this is not always the case
- You need to distinguish not-clearly-separable sub-populations such as monocytes, macrophage, and dendritic cells, which the authors flag as requiring further study
- All cells are already confidently assigned by the marker-count step and the portion of unassigned cells is not below the reassignment threshold

## Inputs

- Gene expression matrix of single-cell RNA-seq data (e.g., .h5ad from SCANPY)
- Initial per-cell assignments and set of unassigned cells from marker-count identification with cell-type-specific rejection thresholds tm
- Normalized marker counts yi,m per cell and cell type
- Number of cells Ncell (to set GMM component count to Ncell/2)

## Outputs

- Cluster labels per cell from the Gaussian mixture model (Ncell/2 clusters on 15 PCs)
- Reassigned cell types for previously unassigned cells via weighted Mahalanobis distance (rho=0.1)
- For the threshold sweep: per-threshold metric table (correct C, error E, erroneously assigned EA, correctly unassigned CUA, erroneously unassigned EUA) written to threshold_sensitivity_results.csv

## How to apply

After initial marker-count-based identification with cell-type-specific rejection thresholds on normalized marker counts, run PCA with 15 components on the gene expression data and cluster with a Gaussian mixture model set to Ncell/2 clusters. Use the resulting clusters to reassign unassigned cells via a weighted Mahalanobis distance that is regularized with rho=0.1 of the average variances, and a penalty weight w=1+3e-(n-1)/2-1 applied per cell type according to marker number n. Trigger reassignment only if the portion of unassigned cells is below the threshold (fixed at 0.8, chosen because it exhibited the best performance in the range 0.1-0.9 at 0.1 steps). Tools: scikit-learn for PCA and GMM, SCANPY/NumPy/Pandas for data handling, within the MarkerCount package.

## Related tools

- **MarkerCount** (Hosts the cluster-basis cell type reassignment pipeline that consumes the PCA+GMM clusters; reassignment triggered when unassigned portion is below threshold (0.8)) — https://github.com/combio-dku/MarkerCount
- **scikit-learn** (Provides PCA (15 components) and the Gaussian mixture model used for clustering prior to reassignment)
- **SCANPY** (Used for large-scale single-cell gene expression data analysis and reading .h5ad inputs)

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref; df_res = MarkerCount_Ref(..arguments..)
```

## Evaluation signals

- The clustering uses exactly 15 PCA components and Ncell/2 GMM clusters; deviations from these settings are inconsistent with the method
- The Mahalanobis distance regularization uses rho=0.1 of the average variances
- For the threshold sweep, the resulting table spans the full 0.1-0.9 range at 0.1 steps and the 0.8 threshold corresponds to the best-performing value
- Reassignment fires only when the portion of unassigned cells is below the threshold; above it, no reassignment occurs
- Per-threshold metrics (C, E, EA, CUA, EUA) can be compared against manual annotations to confirm correct reassignment

## Limitations

- For proper identification, each cell population should be well clustered, even though this is not always the case
- Certain cell types (beta/delta/epsilon/gamma cells in pancreas, dendritic cells in blood, B cells in tumors, T cells in lung) are not clearly separable by normalized marker counts alone and would yield considerable errors
- Handling not-clearly-separable sub-populations, such as monocytes, macrophage, and dendritic cells, requires further studies
- The 0.8 reassignment threshold is a heuristic chosen for best performance on the datasets used in this work

## Evidence
