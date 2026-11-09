---
name: single-cell-spliced-expression-aggregation
description: Use when noisy per-cell scVelo RNA velocity needs smoothing before trajectory
  graph construction, i.e., an AnnData object with a spliced count matrix and velocity
  is available and meta-cell-level outputs are required.
when_to_use_negative:
- Input lacks scVelo-computed RNA velocity and computing it is out of scope — the
  regression requires velocity as the regression target
- Single-cell-level (not meta-cell-level) resolution is required — this step deliberately
  aggregates cells into meta-cells to reduce noise
- The goal is downstream trajectory extraction (kNN graph, path finding, pseudotime)
  rather than velocity smoothing — those are separate later CellPath steps
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellPath
  role: Provides meta_cell_construction() for k-means or leiden meta-cell clustering
    and the RBF kernel regression that yields smoothed meta-cell velocity
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Computes the per-cell RNA velocity used as the regression target when it is
    absent from the AnnData object
- name: scikit-learn
  role: Supplies the KMeans clustering flavor and the kernel-based regression backend
    (Gaussian-RBF, delta=1)
- name: anndata
  role: Holds the spliced count matrix and velocity matrix read from .h5ad as CellPath
    input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:8ab2493762f32b306c6a83cc6161ca08acf9c7e5b46f5d084ae2ead58fdcc998
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PeterZZQ/CellPath
  repo_url: https://github.com/PeterZZQ/CellPaths.git
  tools:
  - CellPath
  - anndata
  - numpy
  - scVelo
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2935
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

# single-cell-spliced-expression-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-cell spliced gene expression to meta-cell level averages and fits a Gaussian-RBF kernel regression to produce smoothed per-meta-cell RNA velocity vectors, the noise-reduction first step of the CellPath trajectory inference pipeline.

## When to use

Use when noisy per-cell scVelo RNA velocity needs smoothing before trajectory graph construction, i.e., an AnnData object with a spliced count matrix and velocity is available and meta-cell-level outputs are required.

## When NOT to use

- Input lacks scVelo-computed RNA velocity and computing it is out of scope — the regression requires velocity as the regression target
- Single-cell-level (not meta-cell-level) resolution is required — this step deliberately aggregates cells into meta-cells to reduce noise
- The goal is downstream trajectory extraction (kNN graph, path finding, pseudotime) rather than velocity smoothing — those are separate later CellPath steps

## Inputs

- AnnData object (.h5ad) containing a spliced count matrix
- RNA velocity matrix computed by scVelo (stored in adata, or computed during initialization when preprocess=False)
- User-specified number of meta-cells (num_metacells)

## Outputs

- Meta-cell-level averaged spliced expression matrix
- Smoothed per-meta-cell RNA velocity vectors saved as smoothed_velocity.npy

## How to apply

Initialize a cp.CellPath object from the AnnData object (with spliced counts and scVelo velocity; set preprocess=False so scvelo computes velocity during initialization if it is absent). Cluster single cells into meta-cells with cellpath.meta_cell_construction() using flavor='k-means' (scikit-learn KMeans) or flavor='leiden', specifying num_metacells as the number of clusters — leiden is recommended for datasets with intricate trajectories. Average spliced gene expression across all cells within each meta-cell to build the meta-cell-level expression matrix. Fit a Gaussian-RBF kernel regression (scikit-learn kernel backend, bandwidth delta=1) regressing meta-cell-aggregated RNA velocity on the averaged spliced expression per gene, then apply the fitted regression to output smoothed per-meta-cell velocity vectors saved as smoothed_velocity.npy. The rationale is that meta-cell aggregation reduces single-cell noise and computational complexity before downstream kNN graph construction.

## Related tools

- **CellPath** (Provides meta_cell_construction() for k-means or leiden meta-cell clustering and the RBF kernel regression that yields smoothed meta-cell velocity) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Computes the per-cell RNA velocity used as the regression target when it is absent from the AnnData object)
- **scikit-learn** (Supplies the KMeans clustering flavor and the kernel-based regression backend (Gaussian-RBF, delta=1))
- **anndata** (Holds the spliced count matrix and velocity matrix read from .h5ad as CellPath input)

## Examples

```
cellpath_obj = cp.CellPath(adata=adata, preprocess=True); cellpath_obj.meta_cell_construction(num_metacells=100, flavor='leiden')
```

## Evaluation signals

- Run the fitted regression in round-trip mode on a small test matrix and verify predicted smoothed velocities are finite
- Check smoothed velocities reproduce the input velocity values within numerical tolerance (value_in_range check)
- Confirm smoothed_velocity.npy is written with shape (num_metacells, n_genes) matching the meta-cell count requested
- Verify each single cell is assigned to exactly one of the num_metacells clusters for both flavor='k-means' and flavor='leiden'

## Limitations

- Performance of the downstream trajectory inference is limited by the accuracy of the upstream RNA velocity inference, which still has unresolved challenges
- The bandwidth delta=1 and meta-cell number are fixed choices; inappropriate num_metacells for the dataset may over- or under-smooth
- Meta-cell aggregation sacrifices single-cell resolution by design to reduce noise and computational complexity

## Evidence
