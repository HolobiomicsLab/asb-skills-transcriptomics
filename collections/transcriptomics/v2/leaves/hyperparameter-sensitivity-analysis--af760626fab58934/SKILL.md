---
name: hyperparameter-sensitivity-analysis
description: Use when you must quantify how sensitive integration performance (e.g.
  scMoMaT's NMI/ARI/GC on the human PBMC dataset) is to choices like latent dimension
  d, lambda, KNN neighborhood size k, or radius r.
when_to_use_negative:
- The analysis goal is a single default-parameter integration run rather than robustness
  characterization (README recommends K = 30, n_neighbors = 30, resolution = 0.9 as
  defaults).
- The dataset has no reference cell type labels, since NMI/ARI cannot be computed
  without them.
- The aim is cross-method benchmarking against LIGER, Seurat, MultiVI, etc. — that
  requires a comparison harness, not a within-method sensitivity sweep.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scMoMaT
  role: 'Provides the model being swept: scmomat_model first-stage training, extract_cell_factors,
    post_process (KNN graph), and leiden_cluster used to produce metrics per grid
    point'
  repo: https://github.com/PeterZZQ/scMoMaT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36066-2
    title: scMoMaT jointly performs single cell mosaic integration and multi-modal
      bio-marker detection
  source_content_hash: sha256:3ed2dc20ef9d248c5a3105a4e7e98298d2e535ac2f53b39ad66a22b2f4e2e669
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PeterZZQ/scMoMaT
  tools:
  - scMoMaT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# hyperparameter-sensitivity-analysis

## Summary

Systematic grid sweep of an integration pipeline's key hyper-parameters (latent dimension d, regularization lambda, neighborhood size k, radius r) with clustering metrics (NMI, ARI) computed per setting, to establish robustness or plateau behavior of the method.

## When to use

Use when you must quantify how sensitive integration performance (e.g. scMoMaT's NMI/ARI/GC on the human PBMC dataset) is to choices like latent dimension d, lambda, KNN neighborhood size k, or radius r.

## When NOT to use

- The analysis goal is a single default-parameter integration run rather than robustness characterization (README recommends K = 30, n_neighbors = 30, resolution = 0.9 as defaults).
- The dataset has no reference cell type labels, since NMI/ARI cannot be computed without them.
- The aim is cross-method benchmarking against LIGER, Seurat, MultiVI, etc. — that requires a comparison harness, not a within-method sensitivity sweep.

## Inputs

- PBMC (ASAP-PBMC) count matrices shipped with the scMoMaT repository (test_pbmc.py)
- Reference cell type labels for the PBMC dataset
- Hyper-parameter grids: d in {10,20,30,50,80}, lambda in {1e-4,1e-3,1e-2}, k in {15,30,50}, r in {0.7,0.9,1}

## Outputs

- Aggregated sweep table hyperparameter_sweep_pbmc.csv (NMI/ARI per d, lambda, k, r)
- Per-grid-point Leiden cluster labels (resolution 0.9) from scmomat.leiden_cluster
- Post-processed KNN graphs (s_pair_dist, knn_indices, knn_dists) per parameter combination

## How to apply

Load the target dataset (for scMoMaT, the provided ASAP-PBMC data via test_pbmc.py) and preprocess the count matrices with utils.preprocess(). Train the first-stage model for each latent dimension d in {10, 20, 30, 50, 80} and lambda in {1e-4, 1e-3, 1e-2}, holding T = 4000 iterations, then extract cell factors. For each of these factor sets, run scmomat.post_process across neighborhood size k in {15, 30, 50} and radius r in {0.7, 0.9, 1} to build the KNN graph. Leiden-cluster every post-processed graph at resolution 0.9 and compute NMI and ARI against the reference cell-type labels. Aggregate the full (d, lambda, k, r) metric grid into a single table (hyperparameter_sweep_pbmc.csv) and inspect it for stability in lambda, k, r and a plateau in NMI/ARI once d exceeds the threshold needed for the dataset.

## Related tools

- **scMoMaT** (Provides the model being swept: scmomat_model first-stage training, extract_cell_factors, post_process (KNN graph), and leiden_cluster used to produce metrics per grid point) — https://github.com/PeterZZQ/scMoMaT

## Examples

```
model = scmomat.scmomat_model(counts = counts, K = 30, device = device); losses = model.train_func(T = 4000); zs = model.extract_cell_factors()
```

## Evaluation signals

- The sweep table covers the full 5 x 3 x 3 x 3 (d, lambda, k, r) grid with a NMI and ARI value per cell.
- NMI and ARI vary negligibly across lambda, k, and r values, matching the reported robustness finding.
- NMI and ARI plateau (stabilize) once d exceeds the latent-dimension threshold sufficient for PBMC (d at or above ~30 given the README's 'number of latent dimensions (20~30)' guidance).
- All clusterings were produced with the same Leiden resolution (0.9) so metric differences are attributable to swept parameters only.
- The output CSV exists, is named hyperparameter_sweep_pbmc.csv, and parses with consistent per-grid-point rows.

## Limitations

- Robustness conclusions are dataset-specific: the finding that scMoMaT is robust to lambda, k, r and plateaus in d was established on the human PBMC dataset and may not transfer to other datasets (e.g. BMMC healthy, Spleen, MOp_5batches).
- The scMoMaT post-processing step had a memory overflow issue that required a fix in v0.2.2, which can affect large grid sweeps.
- The paper defers learning cross-modality feature relationships to future work, so the sweep does not cover feature-representation hyper-parameters.

## Evidence

- [methods] post_process signature: "s_pair_dist, knn_indices, knn_dists = scmomat.post_process(zs, n_neighbors, njobs = 8, r = r)"
- [methods] leiden clustering call: "labels_leiden = scmomat.leiden_cluster(X = None, knn_indices = knn_indices, knn_dists = knn_dists, resolution = resolution)"
- [readme] preprocessing requirement: "The data matrix is of the shape `(ncells, nfeatures)`, and preprocessed with `utils.preprocess()` function."
- [readme] PBMC data availability: "Necessary data for PBMC (the first real dataset in the manuscript, `ASAP-PBMC`) and MOp (the second real dataset in the manuscript, `MOp_5batches`) are provided"
- [readme] first-stage training loop: [evidence span withheld — anchor status: not_found]
