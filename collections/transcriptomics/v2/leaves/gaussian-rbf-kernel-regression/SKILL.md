---
name: gaussian-rbf-kernel-regression
description: Use when noisy per-cell scVelo RNA velocity must be aggregated to a lower-resolution
  representation (meta-cells) and smoothed via regression on averaged spliced expression
  before trajectory graph construction.
when_to_use_negative:
- Input AnnData lacks either the spliced count matrix or a computed velocity matrix
  and upstream scVelo velocity inference is not intended as part of this skill
- Goal is downstream trajectory/pseudotime extraction (kNN graph construction, path
  finding, first-order pseudotime reconstruction) rather than the upstream velocity-smoothing
  step
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellPath
  role: Provides meta_cell_construction() and the regression machinery that fits the
    Gaussian-RBF kernel model on meta-cells
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Computes the per-cell RNA velocity matrix that the regression smooths at meta-cell
    level
- name: scikit-learn
  role: Supplies the KMeans clustering (flavor='k-means') and the kernel regression
    backend used for RBF smoothing
- name: numpy
  role: Handles matrix averaging of spliced counts/velocity across meta-cells and
    saves smoothed_velocity.npy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:19719480ed29ecd6f71ae2f9cb419a2088267cb6efb3cf96585acf673f8c109f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Gaussian-RBF-kernel regression for smoothed RNA velocity

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Gaussian-RBF kernel regression (scikit-learn kernel backend, bandwidth delta=1) to regress meta-cell-aggregated RNA velocity on averaged spliced gene expression, producing smoothed per-meta-cell velocity vectors as the first stage of CellPath's trajectory inference.

## When to use

Use when noisy per-cell scVelo RNA velocity must be aggregated to a lower-resolution representation (meta-cells) and smoothed via regression on averaged spliced expression before trajectory graph construction.

## When NOT to use

- Input AnnData lacks either the spliced count matrix or a computed velocity matrix and upstream scVelo velocity inference is not intended as part of this skill
- Goal is downstream trajectory/pseudotime extraction (kNN graph construction, path finding, first-order pseudotime reconstruction) rather than the upstream velocity-smoothing step

## Inputs

- AnnData object (.h5ad) containing the spliced count matrix
- RNA velocity matrix computed by scVelo and stored in the AnnData object
- User-specified num_metacells (number of clusters)

## Outputs

- Meta-cell-level averaged spliced expression matrix
- Smoothed per-meta-cell RNA velocity vectors (saved as smoothed_velocity.npy)

## How to apply

Initialize a cp.CellPath object with an AnnData object holding the spliced count matrix and scVelo-computed RNA velocity (set preprocess=True, or let scvelo compute velocity during initialization if absent). Cluster single cells into meta-cells with cellpath.meta_cell_construction() using flavor='k-means' (scikit-learn KMeans) or flavor='leiden', specifying num_metacells as the number of clusters — the rationale, per the paper, is to reduce single-cell noise and computational complexity. Average spliced gene expression across cells within each meta-cell to build the meta-cell-level expression matrix. Fit a Gaussian-RBF kernel regression with bandwidth delta=1 using the scikit-learn kernel backend, regressing meta-cell-aggregated RNA velocity on the averaged spliced expression per gene, then apply the fitted regression to output smoothed per-meta-cell velocity vectors (saved as smoothed_velocity.npy). Validate by running the fitted regression in round-trip mode on a small test matrix and checking that predicted velocities are finite and reproduce input velocity values within numerical tolerance.

## Related tools

- **CellPath** (Provides meta_cell_construction() and the regression machinery that fits the Gaussian-RBF kernel model on meta-cells) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Computes the per-cell RNA velocity matrix that the regression smooths at meta-cell level)
- **scikit-learn** (Supplies the KMeans clustering (flavor='k-means') and the kernel regression backend used for RBF smoothing)
- **numpy** (Handles matrix averaging of spliced counts/velocity across meta-cells and saves smoothed_velocity.npy)

## Examples

```
cellpath_obj = cp.CellPath(adata = adata, preprocess = True); cellpath_obj.meta_cell_construction(num_metacells = num_metacells, flavor = 'k-means')
```

## Evaluation signals

- Round-trip test: applying the fitted regression to a small test matrix yields finite values that reproduce input velocities within numerical tolerance (value_in_range check)
- smoothed_velocity.npy is written and contains finite values with one velocity vector per meta-cell
- Number of meta-cells produced equals the user-specified num_metacells under the chosen flavor ('k-means' or 'leiden')
- Meta-cell expression matrix dimensions match: averaged spliced expression and smoothed velocity share the same gene axis and meta-cell axis
- Pipeline runs to completion from cp.CellPath(adata, preprocess=True) initialization through smoothed velocity output

## Limitations

- Performance of all RNA velocity-based methods, including this smoothing step, is limited by the accuracy of upstream RNA velocity inference
- The bandwidth is fixed at delta=1 in this configuration; deviations from that setting are not validated by the source workflow

## Evidence
