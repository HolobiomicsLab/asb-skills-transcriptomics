---
name: pca-dimensionality-reduction-single-cell
description: Use when a single-cell RNA-seq annotation pipeline has cells rejected
  as 'unassigned' after marker counting and needs a low-dimensional embedding (PCA)
  plus clustering to reassign them by cluster similarity.
when_to_use_negative:
- When cells were clearly assigned by marker counts alone and no cluster-basis correction
  is needed
- When cell populations are not well clustered — the paper notes proper identification
  requires well-clustered populations, which is not always the case
- When handling not-clearly-separable sub-populations (e.g., monocytes, macrophage,
  dendritic cells), where the article says further studies are needed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MarkerCount
  role: Owns the pipeline step in which PCA (15 components) and GMM clustering (Ncell/2
    clusters) feed the cluster-basis cell type reassignment of 'unassigned' cells
  repo: https://github.com/combio-dku/MarkerCount
- name: SCANPY
  role: Used for large-scale single-cell gene expression data analysis around the
    MarkerCount workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:11869d8c41ed67adbb2a0c4cee9dd7a3f49e7644c0eee430a0bd49642592ee62
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# pca-dimensionality-reduction-single-cell

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

PCA-based dimensionality reduction of a single-cell expression matrix followed by Gaussian mixture model clustering, used inside the MarkerCount test-phase pipeline to enable cluster-basis reassignment of cells initially rejected as 'unassigned'.

## When to use

Use when a single-cell RNA-seq annotation pipeline has cells rejected as 'unassigned' after marker counting and needs a low-dimensional embedding (PCA) plus clustering to reassign them by cluster similarity.

## When NOT to use

- When cells were clearly assigned by marker counts alone and no cluster-basis correction is needed
- When cell populations are not well clustered — the paper notes proper identification requires well-clustered populations, which is not always the case
- When handling not-clearly-separable sub-populations (e.g., monocytes, macrophage, dendritic cells), where the article says further studies are needed

## Inputs

- Binarized or raw single-cell RNA-seq expression matrix (e.g., .h5ad file) with initial per-cell 'unassigned' flags from marker-count assignment
- Per-cell initial marker-count labels from MarkerCount()

## Outputs

- PCA-reduced embedding of cells (15 components)
- Gaussian mixture model cluster assignments (Ncell/2 clusters)
- Updated cell_assignments.csv with one label per cell drawn from the marker-set cell types plus 'unassigned'

## How to apply

After initial cell-type assignment by normalized marker counts (fraction of the cell type's marker set, N=18 markers per type, expressed in the cell), reduce the test expression matrix with PCA to 15 components. Cluster the reduced space with a Gaussian mixture model using Ncell/2 clusters, the MarkerCount default (analogous to Garnett). For clusters whose portion of 'unassigned' cells is below 0.8 (chosen as best-performing in the 0.1–0.9 range at 0.1 steps), reassign unassigned cells by comparing the weighted Mahalanobis distance — with covariance regularization rho=0.1 of the average variances and size weight 1/pi_m — from centroids of partially occupying cell types, assigning the closest. The rationale is that conservative rejection thresholds yield many 'unassigned' cells, and cluster-basis reassignment using gene expression pattern similarities recovers labels that marker counts alone misidentify (e.g., beta/delta/epsilon/gamma pancreas cells, dendritic cells in blood).

## Related tools

- **MarkerCount** (Owns the pipeline step in which PCA (15 components) and GMM clustering (Ncell/2 clusters) feed the cluster-basis cell type reassignment of 'unassigned' cells) — https://github.com/combio-dku/MarkerCount
- **SCANPY** (Used for large-scale single-cell gene expression data analysis around the MarkerCount workflow)

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref, MarkerCount; df_res = mkrcnt$MarkerCount_Ref( .. arguments .. )  # R via reticulate: mkrcnt <- import("MarkerCount.marker_count")
```

## Evaluation signals

- cell_assignments.csv exists and contains exactly one label per cell, drawn from the marker-set cell types plus 'unassigned'
- Reassignment was applied only in clusters whose portion of unassigned cells is below the 0.8 threshold
- PCA retains exactly 15 components and the GMM uses Ncell/2 clusters
- Mahalanobis-distance reassignment uses covariance regularization rho=0.1 and size weight 1/pi_m
- Final labels no longer mark cells that fall inside clusters with <0.8 unassigned portion as 'unassigned'

## Limitations

- Reassignment assumes well-clustered cell populations, which is not always the case
- Not-clearly-separable sub-populations such as monocytes, macrophage, and dendritic cells remain problematic
- Cluster reassignment threshold 0.8 was chosen empirically (best in 0.1–0.9 at 0.1 steps) and may not generalize to all datasets
- Binarized gene expression is used to suppress batch effect, but whether MarkerCount's superior performance stems from this is unclear

## Evidence
