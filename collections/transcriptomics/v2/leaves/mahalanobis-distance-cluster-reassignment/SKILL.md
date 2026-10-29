---
name: mahalanobis-distance-cluster-reassignment
description: Use when a marker-count-based annotation leaves cells 'unassigned' by
  a rejection threshold, and you want to recover labels by exploiting clustering structure
  (PCA + GMM clusters) with within-cluster expression similarity.
when_to_use_negative:
- Clusters where 80% or more of cells are unassigned — reassignment is deliberately
  not applied there, since the cluster likely represents an unknown cell type
- Input lacks a prior marker-count assignment with rejection thresholds tm, so there
  is no 'unassigned' population to reassign
- Cell populations are poorly clustered (MarkerCount's discussion notes each population
  should be well clustered for proper identification)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MarkerCount
  role: 'Implements the cluster-basis reassignment: MarkerCount() for marker-based
    mode and MarkerCount_Ref() for reference-based mode; requires Numpy, Pandas, sklearn,
    scipy'
  repo: https://github.com/combio-dku/MarkerCount
- name: SCANPY
  role: Single-cell data handling surrounding the pipeline (preprocessing/visualization);
    the README notes scanpy is required only to show results, not for MarkerCount
    itself
  repo: https://github.com/combio-dku/MarkerCount
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:64bcd4bd6beefb5bb7c34e6f3ea671378df19148370b929a3904dfe282c7dd01
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Mahalanobis-distance cluster reassignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MarkerCount's cluster-basis reassignment step: after a conservative marker-count rejection leaves cells 'unassigned', unassigned cells inside well-populated clusters are relabeled using weighted Mahalanobis distances to the centroids of partially occupying cell types, minimizing unnecessarily unassigned cells.

## When to use

Use when a marker-count-based annotation leaves cells 'unassigned' by a rejection threshold, and you want to recover labels by exploiting clustering structure (PCA + GMM clusters) with within-cluster expression similarity.

## When NOT to use

- Clusters where 80% or more of cells are unassigned — reassignment is deliberately not applied there, since the cluster likely represents an unknown cell type
- Input lacks a prior marker-count assignment with rejection thresholds tm, so there is no 'unassigned' population to reassign
- Cell populations are poorly clustered (MarkerCount's discussion notes each population should be well clustered for proper identification)

## Inputs

- Binarized single-cell RNA-seq expression matrix (bij=1 if xij>0) with marker counts yi,m per cell
- Initial assignments including 'unassigned' labels from the marker-count rejection step
- PCA-reduced test data (15 components) with Gaussian-mixture clusters (Ncell/2 clusters)

## Outputs

- Final cell_assignments.csv with one label per cell drawn from the marker-set cell types plus 'unassigned'
- Per-cluster reassignment decisions (clusters with unassigned portion < 0.8 relabeled)

## How to apply

After initial MarkerCount assignment with rejection at cell-type-specific thresholds tm, reduce the test expression data to 15 PCA components and cluster with a Gaussian mixture model using Ncell/2 clusters (MarkerCount default, analogous to Garnett). For each cluster whose portion of unassigned cells is below 0.8 (chosen as best in the tested 0.1–0.9 range), compute the weighted Mahalanobis distance — with covariance regularization rho=0.1 of the average variances and a size weight 1/pi_m — from each unassigned cell to the centroids of the cell types partially occupying that cluster, and assign the closest cell type. Clusters with an unassigned portion of 0.8 or more are left untouched, preserving the conservative rejection behavior. When marker counts are uneven across cell types, apply the penalty weight w = 1 + 3e^(-(n-1)/2) - 1 based on the number n of markers.

## Related tools

- **MarkerCount** (Implements the cluster-basis reassignment: MarkerCount() for marker-based mode and MarkerCount_Ref() for reference-based mode; requires Numpy, Pandas, sklearn, scipy) — https://github.com/combio-dku/MarkerCount
- **SCANPY** (Single-cell data handling surrounding the pipeline (preprocessing/visualization); the README notes scanpy is required only to show results, not for MarkerCount itself) — https://github.com/combio-dku/MarkerCount

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref, MarkerCount; df_res = MarkerCount_Ref(data, markers, ...)  # via reticulate in R: mkrcnt$MarkerCount_Ref(.. arguments ..)
```

## Evaluation signals

- cell_assignments.csv exists and contains exactly one label per cell, each drawn from the marker-set cell types plus 'unassigned'
- Reassignment was applied only in clusters whose portion of unassigned cells was below 0.8; clusters at or above 0.8 retain 'unassigned' cells
- Mahalanobis distances use covariance regularization rho=0.1 of the average variances and size weight 1/pi_m — verify these parameters in the call
- PCA retains 15 components and the GMM uses Ncell/2 clusters (MarkerCount default)
- Number of 'unassigned' cells decreases relative to the pre-reassignment assignment without misassigning cells in predominantly-unassigned clusters

## Limitations

- Requires well-clustered cell populations; for proper identification each population should be well clustered, which is not always the case
- Not-clearly-separable sub-populations (e.g., monocytes, macrophages, dendritic cells) remain difficult and require further study
- Unclear whether gains come from binarized expression suppressing batch effects, so the mechanism behind improved performance in some datasets is not settled
- Cell types and taxonomy are not standardized; researchers must supply their own marker lists

## Evidence
