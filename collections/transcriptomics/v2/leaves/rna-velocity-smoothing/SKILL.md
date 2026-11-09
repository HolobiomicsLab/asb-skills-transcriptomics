---
name: rna-velocity-smoothing
description: Use when an scRNA-seq AnnData object with spliced counts and scVelo-derived
  RNA velocity must be denoised into meta-cell-level smoothed velocities as input
  for trajectory inference.
when_to_use_negative:
- The goal is cell-level rather than meta-cell-level velocities, or no RNA velocity
  (spliced/unspliced) information is available for the dataset
- The task is downstream of smoothing (kNN graph construction, path finding, or pseudotime
  assignment) — use the later CellPath steps such as all_in_one instead
- Single-cell counts are extremely sparse for direct velocity estimation and no unspliced
  reads exist; scVelo velocity inference itself may be unreliable
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellPath
  role: Provides CellPath class, meta_cell_construction() (flavor='k-means' or 'leiden'),
    and the RBF-kernel regression that outputs smoothed velocity per meta-cell
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Computes the raw per-cell RNA velocity that feeds the regression; used during
    CellPath initialization when velocity is absent from the AnnData object
- name: scikit-learn
  role: Supplies KMeans for k-means flavor meta-cell clustering and the kernel-regression
    backend for the Gaussian-RBF (delta=1) velocity smoothing
- name: numpy
  role: Persists the smoothed per-meta-cell velocity matrix as smoothed_velocity.npy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:a19fed14d7b5f1b89171962ea28f9e6039ba35b14fb1278fd472c4559b499fd0
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
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# rna-velocity-smoothing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

CellPath's first pipeline step: cluster single cells into meta-cells and fit a Gaussian-RBF kernel regression regressing meta-cell-aggregated RNA velocity on averaged spliced expression, yielding noise-reduced per-meta-cell smoothed velocity vectors.

## When to use

Use when an scRNA-seq AnnData object with spliced counts and scVelo-derived RNA velocity must be denoised into meta-cell-level smoothed velocities as input for trajectory inference.

## When NOT to use

- The goal is cell-level rather than meta-cell-level velocities, or no RNA velocity (spliced/unspliced) information is available for the dataset
- The task is downstream of smoothing (kNN graph construction, path finding, or pseudotime assignment) — use the later CellPath steps such as all_in_one instead
- Single-cell counts are extremely sparse for direct velocity estimation and no unspliced reads exist; scVelo velocity inference itself may be unreliable

## Inputs

- AnnData (.h5ad) object containing the spliced count matrix and scVelo-computed RNA velocity (or spliced/unspliced counts from which scVelo can compute velocity during initialization)
- User-specified num_metacells (number of meta-cell clusters)

## Outputs

- Meta-cell-level averaged spliced expression matrix
- Smoothed per-meta-cell RNA velocity matrix (smoothed_velocity.npy)

## How to apply

Initialize cp.CellPath with the AnnData object (preprocess=True if velocity is already stored in adata; otherwise scVelo computes it during initialization). Cluster cells into meta-cells with cellpath.meta_cell_construction() using flavor='k-means' (scikit-learn KMeans) or flavor='leiden' (better for intricate trajectories), specifying num_metacells as the cluster count. Average spliced gene expression across cells within each meta-cell to build the meta-cell-level expression matrix, then fit a Gaussian-RBF kernel regression (scikit-learn kernel backend, bandwidth delta=1) regressing meta-cell-aggregated velocity on the averaged spliced expression per gene. Apply the fitted regression to output smoothed per-meta-cell velocity vectors and save the result as smoothed_velocity.npy.

## Related tools

- **CellPath** (Provides CellPath class, meta_cell_construction() (flavor='k-means' or 'leiden'), and the RBF-kernel regression that outputs smoothed velocity per meta-cell) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Computes the raw per-cell RNA velocity that feeds the regression; used during CellPath initialization when velocity is absent from the AnnData object)
- **scikit-learn** (Supplies KMeans for k-means flavor meta-cell clustering and the kernel-regression backend for the Gaussian-RBF (delta=1) velocity smoothing)
- **numpy** (Persists the smoothed per-meta-cell velocity matrix as smoothed_velocity.npy)

## Examples

```
cellpath_obj = cp.CellPath(adata = adata, preprocess = True); cellpath_obj.meta_cell_construction(num_metacells = num_metacells, flavor = "leiden")
```

## Evaluation signals

- Round-trip check: run the fitted RBF regression on a small test matrix and confirm the script executes without error
- All predicted smoothed velocity entries are finite (no NaN/Inf)
- Smoothed velocities on the training meta-cells reproduce the input velocity values within numerical tolerance (value_in_range check)
- Number of meta-cells produced equals the user-specified num_metacells for either flavor='k-means' or 'leiden'
- smoothed_velocity.npy is written and its shape matches (num_metacells, n_velocity_genes)

## Limitations

- Performance of downstream trajectories is limited by RNA velocity inference accuracy: 'the performance of all RNA velocity-based methods are affected by accuracy in RNA velocity inference'
- Smoothing operates at meta-cell granularity to reduce single-cell noise, trading per-cell resolution for stability
- Choice of num_metacells and flavor affects the smoothing: leiden is recommended for datasets with intricate trajectories

## Evidence
