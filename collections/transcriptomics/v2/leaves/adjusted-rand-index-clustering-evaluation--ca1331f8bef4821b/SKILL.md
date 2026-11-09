---
name: adjusted-rand-index-clustering-evaluation
description: Use when comparing clusterings (e.g., Leiden community detection on a
  q-diffused, Gaussian, or scVI-derived graph) against expert manual annotations across
  cell subtypes or organ tissues, and you need an ARI-based accuracy metric for benchmarking.
when_to_use_negative:
- When ground-truth labels are unavailable or the clustering cannot be compared to
  manually annotated phenotypes — ARI cannot be computed and the Tabula Sapiens/PBMC
  evaluation design does not apply
- When the tissue sample has 10,000 or more sampled cells or contains PBMCs that were
  not screened out — outside the article's Tabula Sapiens benchmark scope
- When evaluating embedding quality by visualization or trajectory inference rather
  than clustering agreement — the article's metric here is ARI against manual annotations,
  not downstream qualitative structure
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: q-Diffusion.jl
  role: Produces the q-diffused affinity matrix (preprocess_cell_counts, estimate_inner_bandwidth,
    AdaptiveKernel/DeformedKernel) whose Leiden clusters are scored by ARI; the accuracy
    benchmark an accelerated variant must preserve is the Tabula Sapiens ARI
  repo: github.com/marmarelis/QDiffusion.jl
- name: Leiden
  role: Community detection algorithm whose cluster assignments (optimizing modularity
    on the symmetrized weighted graph) are evaluated by ARI against manual annotations
- name: scVI
  role: Atlas-supplied dimensionality reduction serving as a competing embedding whose
    clustering ARIs are compared against q-diffusion on Tabula Sapiens
- name: Seurat
  role: Baseline neighborhood-graph method providing comparison clusterings scored
    by ARI in the PBMC IFN-γ benchmark
- name: Scanpy
  role: Baseline neighborhood-graph method providing comparison clusterings scored
    by ARI in the PBMC and Tabula Sapiens benchmarks
- name: PHATE
  role: Baseline method (and qPHATE variant) whose clusterings are scored by ARI for
    comparison
- name: SC3
  role: Baseline clustering method providing comparison clusters for ARI evaluation
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_004
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:75f96655a6ca8e7010a0a9599d7226a70f5264f8d7b03b74d1bf1793e72fe0d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leiden
  - Monocle
  - PHATE
  - SC3
  - Scanpy
  - Seurat
  - q-Diffusion.jl
  - q-diffusion
  - scVI
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# adjusted-rand-index-clustering-evaluation

## Summary

Evaluates unsupervised cell clustering quality by computing adjusted Rand indices (ARI) of Leiden community detection assignments against manually annotated cell phenotypes, as used in the q-diffusion benchmarks on the PBMC IFN-γ dataset and the Tabula Sapiens human atlas.

## When to use

Use when comparing clusterings (e.g., Leiden community detection on a q-diffused, Gaussian, or scVI-derived graph) against expert manual annotations across cell subtypes or organ tissues, and you need an ARI-based accuracy metric for benchmarking.

## When NOT to use

- When ground-truth labels are unavailable or the clustering cannot be compared to manually annotated phenotypes — ARI cannot be computed and the Tabula Sapiens/PBMC evaluation design does not apply
- When the tissue sample has 10,000 or more sampled cells or contains PBMCs that were not screened out — outside the article's Tabula Sapiens benchmark scope
- When evaluating embedding quality by visualization or trajectory inference rather than clustering agreement — the article's metric here is ARI against manual annotations, not downstream qualitative structure

## Inputs

- Leiden cluster assignments per cell (from a q-diffused kernel graph, symmetrized as A=(Ã+Ãᵀ)/2, or from baseline embeddings scVI/Seurat/Monocle/Scanpy/SC3/PHATE)
- Manually annotated cell phenotypes from the Tabula Sapiens atlas (per organ: skin, trachea, uterus, liver) or known IFN-γ stimulation labels within the eight PBMC subtypes
- Log-normalized (genes x cells) matrix from preprocess_cell_counts, with no feature selection or dimensionality reduction

## Outputs

- Adjusted Rand index (ARI) per organ tissue or PBMC subtype, comparing clustering to manual annotations
- Per-organ / per-subtype ARI comparison table (e.g., tabula_sapiens_ari_comparison.csv, cond_qd_pbmc_ari.csv, ann_benchmark.csv)

## How to apply

First obtain the clustering to be scored: run Leiden community detection optimizing modularity on the symmetrized weighted undirected graph, where the affinity matrix comes from a q-diffused kernel (AdaptiveKernel wrapping a DeformedKernel with q=1.2, inner bandwidth from estimate_inner_bandwidth, symmetrized as A=(Ã+Ãᵀ)/2) or from a baseline embedding (Seurat, Monocle, Scanpy, SC3, PHATE, scVI). For the Tabula Sapiens benchmark, restrict to organ tissues with fewer than 10,000 sampled cells and screen out PBMCs before clustering; optimize clustering parameters per organ by a grid search over silhouette scores. Then compute the ARI of the cluster assignments against the manually annotated cell phenotypes — within each of the eight PBMC subtypes for the IFN-γ classification benchmark, or per organ (skin, trachea, uterus, liver) for the unsupervised Tabula Sapiens benchmark. Judge success by comparing ARIs across methods: q-diffusion should exceed baseline embeddings in three of four organs, with the liver as the known degraded case, and CD8 T cells as the known PBMC subtype where existing methods perform better. Write per-comparison ARI tables to CSV (e.g., tabula_sapiens_ari_comparison.csv, cond_qd_pbmc_ari.csv, ann_benchmark.csv).

## Related tools

- **q-Diffusion.jl** (Produces the q-diffused affinity matrix (preprocess_cell_counts, estimate_inner_bandwidth, AdaptiveKernel/DeformedKernel) whose Leiden clusters are scored by ARI; the accuracy benchmark an accelerated variant must preserve is the Tabula Sapiens ARI) — github.com/marmarelis/QDiffusion.jl
- **Leiden** (Community detection algorithm whose cluster assignments (optimizing modularity on the symmetrized weighted graph) are evaluated by ARI against manual annotations)
- **scVI** (Atlas-supplied dimensionality reduction serving as a competing embedding whose clustering ARIs are compared against q-diffusion on Tabula Sapiens)
- **Seurat** (Baseline neighborhood-graph method providing comparison clusterings scored by ARI in the PBMC IFN-γ benchmark)
- **Scanpy** (Baseline neighborhood-graph method providing comparison clusterings scored by ARI in the PBMC and Tabula Sapiens benchmarks)
- **PHATE** (Baseline method (and qPHATE variant) whose clusterings are scored by ARI for comparison)
- **SC3** (Baseline clustering method providing comparison clusters for ARI evaluation)

## Examples

```
kernel = AdaptiveKernel(16, DeformedKernel(1.2f0, data, inner_bandwidth=estimate_inner_bandwidth(data)), data, in_log=true)
```

## Evaluation signals

- ARI is computed against the expert manual annotations of the Tabula Sapiens atlas (or known IFN-γ labels within the eight PBMC subtypes), not against unlabeled metrics
- Expected q-diffusion result: higher ARI than atlas-supplied embeddings (including scVI) in skin, trachea, and uterus, but degraded clusters in the liver — deviations should be investigated, not silently accepted
- Clustering parameters must have been optimized per case by a grid search over silhouette scores before ARI evaluation
- The affinity matrix must be the arithmetic symmetrization A=(Ã+Ãᵀ)/2 of the anisotropic kernel matrix with the diagonal subtracted before Leiden modularity optimization
- An approximate-kernel variant is accepted only if its Leiden ARI on Tabula Sapiens annotations remains close to the exact kernel's ARI while runtime is reduced (exact evaluation reported at 20 minutes to 3 hours per affinity matrix)

## Limitations

- No algorithm is provided for selecting the q-diffusion parameters q and φ (deformation parameter and inner bandwidth); q=1.2 was used in the benchmarks and the README example uses q=1.5
- The article did not consider batch-effect correction or alignment of disparate datasets, so ARI comparisons assume comparable preprocessing
- Runtime of the full all-pairs q-diffused kernel evaluation is suboptimal (20 min to 3 h for one affinity matrix), which constrains how many clustering configurations can be ARI-scored
- Known failure cases: liver clusters degrade under q-diffusion in the Tabula Sapiens benchmark, and CD8 T cells favor existing methods in the PBMC benchmark

## Evidence

- [results] Adjusted Rand indices (ARIs)31 of Leiden community detection42 in the PBMC dataset.: "Adjusted Rand indices (ARIs)31 of Leiden community detection42 in the PBMC dataset."
- [results] grid search silhouette optimization: "Clustering parameters were optimized by a grid search over the silhouette scores44 for each case."
- [results] Tabula Sapiens selection filter: "we selected those organ tissues for which fewer than 10,000 cells were sampled. We also screened out the PBMCs"
- [readme] kernel construction parameters: "kernel = AdaptiveKernel(16, # scale to the 16th nearest neighbor
  DeformedKernel(1.5f0, # set deformation parameter q=1.5"
- [methods] inner bandwidth heuristic: "inner_bandwidth = estimate_inner_bandwidth(data)"
- [readme] preprocessing without feature selection: "There is also a convenience mathod called `preprocess_cell_counts(data)` that returns a named tuple containing different versions of the matrix"
