---
name: cluster-basis-cell-type-reassignment
description: Use when a marker-count-based identification run leaves many cells unassigned
  after rejection, and you want to reassign them cluster-wise before finalizing cell
  type labels.
when_to_use_negative:
- When expression data are not well clustered — the paper notes proper identification
  requires each cell population to be well clustered, which is not always the case
- When the goal is plain marker counting without a rejection/reassignment stage (e.g.,
  marker-based HiCAT-style hierarchical annotation), since this skill specifically
  targets cells left unassigned by the rejection threshold
- When cell types are not clearly separable sub-populations (e.g., monocytes, macrophage,
  dendritic cells), which the discussion flags as requiring further study
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MarkerCount
  role: Implements rejection of uncertain cells followed by cluster-basis cell type
    reassignment/correction; provide MarkerCount_Ref (reference-based) or MarkerCount
    (marker-based) entry points
  repo: https://github.com/combio-dku/MarkerCount
- name: SCANPY
  role: Used for large-scale single-cell gene expression data analysis and for showing
    results alongside the identification pipeline
- name: scikit-learn
  role: Provides PCA (15 components) and Gaussian mixture model clustering and Mahalanobis-distance
    machinery used during cluster-basis reassignment
- name: NumPy
  role: Numerical computations for marker counts, distances, and regularization in
    the reassignment step
- name: Pandas
  role: Assemble and export the per-threshold metric table (e.g., threshold_sensitivity_results.csv)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.06.010
    title: 'MarkerCount: A stable, count-based cell type identifier for single-cell
      RNA-seq experiments'
  source_content_hash: sha256:27ce769e1c056cf12e38be4bcfdc21ad053dfaf0c1976335daa81b62b4b1d7f0
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
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cluster-basis-cell-type-reassignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

MarkerCount's post-rejection step that reassigns 'unassigned' cells to cell types using similarity within clusters, with a reassignment trigger threshold of 0.8 (portion of unassigned cells), chosen as the best performer over a 0.1–0.9 sweep at 0.1 steps.

## When to use

Use when a marker-count-based identification run leaves many cells unassigned after rejection, and you want to reassign them cluster-wise before finalizing cell type labels.

## When NOT to use

- When expression data are not well clustered — the paper notes proper identification requires each cell population to be well clustered, which is not always the case
- When the goal is plain marker counting without a rejection/reassignment stage (e.g., marker-based HiCAT-style hierarchical annotation), since this skill specifically targets cells left unassigned by the rejection threshold
- When cell types are not clearly separable sub-populations (e.g., monocytes, macrophage, dendritic cells), which the discussion flags as requiring further study

## Inputs

- Binarized gene expression matrix (bij = 1 if xij > 0) from single-cell RNA-seq data (e.g., .h5ad)
- Normalized marker counts yi,m computed per cell against selected markers (N=18 markers per cell type, occurrence frequency cutoff fth=0.9)
- Cell-type-specific rejection thresholds tm applied to normalized marker counts
- Initial per-cell assignments plus the set of unassigned cells

## Outputs

- Final cell type labels including reassigned cells (previously unassigned) and cluster-wise corrections
- Per-threshold sensitivity metrics table (e.g., threshold_sensitivity_results.csv) with C, E, EA, CUA, EUA measures across 0.1–0.9

## How to apply

After computing normalized marker counts and applying cell-type-specific rejection thresholds tm, cells that fail rejection become 'unassigned'. Reduce dimensionality of the expression data with PCA using 15 components and cluster with a Gaussian mixture model at Ncell/2 clusters. If the portion of unassigned cells falls below a threshold (fixed at 0.8, which exhibited the best performance in the range 0.1–0.9 at 0.1 steps), reassign unassigned cells cluster-basis using the weighted Mahalanobis distance with regularization rho=0.1 of average variances; apply the penalty weight w=1+3e-(n-1)/2-1 per cell type according to its number of markers n to resolve uneven marker counts. Score the outcome with the five measures correct (C), error (E), erroneously assigned (EA), correctly unassigned (CUA), and erroneously unassigned (EUA) against manual annotations; 0.8 corresponded to the best-performing threshold in the sweep.

## Related tools

- **MarkerCount** (Implements rejection of uncertain cells followed by cluster-basis cell type reassignment/correction; provide MarkerCount_Ref (reference-based) or MarkerCount (marker-based) entry points) — https://github.com/combio-dku/MarkerCount
- **SCANPY** (Used for large-scale single-cell gene expression data analysis and for showing results alongside the identification pipeline)
- **scikit-learn** (Provides PCA (15 components) and Gaussian mixture model clustering and Mahalanobis-distance machinery used during cluster-basis reassignment)
- **NumPy** (Numerical computations for marker counts, distances, and regularization in the reassignment step)
- **Pandas** (Assemble and export the per-threshold metric table (e.g., threshold_sensitivity_results.csv))

## Examples

```
from MarkerCount.marker_count import MarkerCount_Ref; df_res = MarkerCount_Ref(.. arguments ..)
```

## Evaluation signals

- The sweep table spans the full 0.1–0.9 range at 0.1 steps and the 0.8 threshold corresponds to the best identification performance
- Reassignment is only triggered when the portion of unassigned cells is below the 0.8 threshold
- PCA retains 15 components and the Gaussian mixture model uses Ncell/2 clusters, matching the paper's parameters
- Mahalanobis distance uses regularization rho=0.1 of average variances, and the penalty weight w=1+3e-(n-1)/2-1 is applied per cell type by marker count n
- Results are scored against manual annotations with the five measures C, E, EA, CUA, EUA, and reassigned cells reduce erroneously unassigned (EUA) without inflating erroneously assigned (EA)

## Limitations

- For proper identification, each cell population should be well clustered, even though this is not always the case
- Handling not-clearly-separable sub-populations, such as monocytes, macrophages, and dendritic cells, is a greater issue that requires further studies
- Cell types and taxonomy are not standardized, so researchers must prepare their own marker lists for the specific study and objective
- Although binarized gene expression was employed to suppress batch effect, it is unclear whether superior performance in some datasets stems from this binarization

## Evidence
