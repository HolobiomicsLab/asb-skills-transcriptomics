---
name: dimensionality-reduction-benchmarking
description: Use when comparing dimensionality reduction methods for scRNA-seq embeddings
  on sparse or high-CV data, or when evaluating whether automatic PC/signal-dimension
  selection (e.g., RMT-based) beats fixed-PC baselines.
when_to_use_negative:
- Input is already a reduced embedding or feature table rather than raw gene-by-cell
  count data
- The question is about a single method's embedding quality on one dataset without
  competing baselines or ground-truth labels for ECS/SIL
- Data violate the QC scope assumptions, e.g., non-scRNA-seq counts where the 200-gene/15-cell/5%-mitochondrial
  filters would remove meaningful content
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scLENS
  role: Method under test; provides data-driven signal detection via RMT noise filtering
    and signal robustness test
  repo: https://github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python distribution of scLENS used for setup with PyTorch/CUDA environments
  repo: https://github.com/Mathbiomed/scLENSpy
- name: scDesign2
  role: Simulator (v1.0.0) trained on reference immune cell scRNA-seq data to generate
    benchmark datasets
- name: Seurat
  role: Baseline DR package run with log normalization + 50 PCs
- name: Scanpy
  role: Baseline DR package (50 PCs); its highly_variable_genes function selects 10,000
    genes for scVI
- name: Monocle3
  role: Baseline DR package run with 50 PCs
- name: ParallelPCA
  role: Baseline with automatic PC selection via parallel analysis
- name: Randomly
  role: Baseline with RMT-based automatic signal selection
- name: ACTIONet
  role: Baseline matrix-factorization DR method
- name: SHARP
  role: Baseline random-projection DR method
- name: scDHA
  role: Baseline autoencoder-based DR method
- name: scVI
  role: Baseline autoencoder DR run on 10,000 highly variable genes
- name: ZINB-WaVE
  role: Baseline model-based DR method
- name: scTransform
  role: Baseline model-based normalization/DR method
- name: UMAP
  role: UMAP.jl used to build 2D embeddings (15 neighbors, min_dist 0.01, cosine metric)
    for hierarchical clustering evaluation
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:fa65f7dce22d17ed91d7a5c03f2b1e52faee6e5f09fa37147b315a9f96c51d7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
  tools:
  - ACTIONet
  - CSV
  - CUDA
  - DataFrames
  - Monocle3
  - ParallelPCA
  - Randomly
  - SHARP
  - Scanpy
  - Seurat
  - SparseArrays.jl
  - UMAP
  - ZINB-WaVE
  - scDHA
  - scDesign2
  - scLENS
  - scLENSpy
  - scTransform
  - scVI
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8275862068965517
n_claims: 29
---

# dimensionality-reduction-benchmarking

## Summary

Benchmark scLENS against 11 widely used dimensionality reduction packages (Seurat, Scanpy, Monocle3, ParallelPCA, Randomly, ACTIONet, SHARP, scDHA, scVI, ZINB-WaVE, scTransform) on scRNA-seq datasets using SIL, ECS, and average kNN-overlap scores, with sparsity and CV of TGC as stress axes.

## When to use

Use when comparing dimensionality reduction methods for scRNA-seq embeddings on sparse or high-CV data, or when evaluating whether automatic PC/signal-dimension selection (e.g., RMT-based) beats fixed-PC baselines.

## When NOT to use

- Input is already a reduced embedding or feature table rather than raw gene-by-cell count data
- The question is about a single method's embedding quality on one dataset without competing baselines or ground-truth labels for ECS/SIL
- Data violate the QC scope assumptions, e.g., non-scRNA-seq counts where the 200-gene/15-cell/5%-mitochondrial filters would remove meaningful content

## Inputs

- UMI or read count scRNA-seq matrices (CSV/jld2/10x gz format: matrix.mtx.gz, features.tsv.gz, barcodes.tsv.gz) with ground-truth cell-type labels
- scDesign2 (v1.0.0)-simulated count datasets stratified by sparsity and CV of TGC
- Deeply sequenced datasets with average TGC > 25,000 per cell for downsampling benchmarks

## Outputs

- SIL and ECS score tables per dataset-package pair (e.g., Fig. 4a,c; Fig. 5a)
- Average kNN-overlap scores for all 12 methods on original vs. downsampled data (Fig. 6a)
- Normalized kNN-overlap curves comparing scLENS vs. elbow vs. 95%-variance PC selection across downsampling ratios 0.1-0.5 (Fig. 6c)
- UMAP/PCA embeddings and cluster assignments from hierarchical and Leiden clustering

## How to apply

Assemble simulated (scDesign2-trained, e.g., ~60,000 T cells subsampled to ~3000 cells per dataset across sparsity ~0.84 and CV of TGC ~0.3 strata) and real datasets, applying QC that drops cells with <200 expressed genes, genes expressed in <15 cells, and cells with >5% mitochondrial reads. Run each package at default settings (50 PCs for Seurat/Scanpy/Monocle3; 10,000 highly variable genes for scVI; scLENS with log normalization scaling factor 1, L2 normalization, RMT-based noise filtering, and the signal robustness test). Derive two cluster assignments per embedding: hierarchical clustering on the 2D UMAP embedding (15 neighbors, min_dist 0.01, cosine metric) and Leiden clustering on an SNN graph (20 neighbors, edge cutoff 1/15, cosine similarity), sweeping cut heights/resolutions to maximize ECS against ground truth. Score each dataset-package pair with average silhouette (SIL), element-centric similarity (ECS, damping factor 0.975), and average kNN-overlap (k = 5 to 50, normalized by (n x k)/2). For downsampling benchmarks, weight cells by total gene counts to target ~5,000 average TGC and compare each method's kNN graph to the original-data reference. Correct reproduction shows scLENS first in SIL/ECS and highest kNN-overlap, with the smallest degradation as sparsity increases while fixed-50-PC packages decline substantially.

## Related tools

- **scLENS** (Method under test; provides data-driven signal detection via RMT noise filtering and signal robustness test) — https://github.com/Mathbiomed/scLENS
- **scLENSpy** (Python distribution of scLENS used for setup with PyTorch/CUDA environments) — https://github.com/Mathbiomed/scLENSpy
- **scDesign2** (Simulator (v1.0.0) trained on reference immune cell scRNA-seq data to generate benchmark datasets)
- **Seurat** (Baseline DR package run with log normalization + 50 PCs)
- **Scanpy** (Baseline DR package (50 PCs); its highly_variable_genes function selects 10,000 genes for scVI)
- **Monocle3** (Baseline DR package run with 50 PCs)
- **ParallelPCA** (Baseline with automatic PC selection via parallel analysis)
- **Randomly** (Baseline with RMT-based automatic signal selection)
- **ACTIONet** (Baseline matrix-factorization DR method)
- **SHARP** (Baseline random-projection DR method)
- **scDHA** (Baseline autoencoder-based DR method)
- **scVI** (Baseline autoencoder DR run on 10,000 highly variable genes)
- **ZINB-WaVE** (Baseline model-based DR method)
- **scTransform** (Baseline model-based normalization/DR method)
- **UMAP** (UMAP.jl used to build 2D embeddings (15 neighbors, min_dist 0.01, cosine metric) for hierarchical clustering evaluation)

## Examples

```
using scLENS; pre_df = scLENS.preprocess(scLENS.read_file("data/Z8eq.csv.gz")); sclens_embedding = scLENS.sclens(pre_df, device_="gpu")
```

## Evaluation signals

- scLENS attains the highest average SIL and ECS (damping factor 0.975) among the 12 packages, with the smallest performance decline as sparsity increases
- In the downsampling benchmark, scLENS achieves the highest average kNN-overlap score (k = 5 to 50, normalized by (n x k)/2)
- scLENS-selected PC counts yield normalized kNN-overlap scores near the maximum, while the elbow method under-selects and the 95%-variance criterion over-selects PCs across downsampling ratios 0.1-0.5
- Every package yields two cluster assignments (UMAP hierarchical and SNN-graph Leiden) whose cut heights/resolutions were swept to maximize ECS against ground truth
- QC filters are consistently applied: cells <200 expressed genes, genes <15 cells, cells >5% mitochondrial reads removed before all runs

## Limitations

- RMT-based noise filtering requires computing all eigenvalues of the large cell-to-cell similarity matrix, demanding significant memory resources; operation on large data with whole genes requires a computer with high specifications
- Determining optimal dimensions is usually left to the user's discretion in most methods, introducing subjectivity into downstream benchmark results
- Feature selection of highly variable genes can compromise the local structure contained in low variance genes, which affects baselines relying on HVG selection

## Evidence

- [abstract] outperforms 11 DR tools on challenging data: "Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets with high sparsity and variability"
- [abstract] RMT noise filtering and signal robustness test: "we utilize random matrix theory-based noise filtering and a signal robustness test to enable data-driven determination of the threshold for signal dimensions"
- [discussion] data-driven signal/noise threshold: "provides a data-driven threshold to differentiate signals from noise by RMT-based noise filtering"
- [readme] QC filtering thresholds in README: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality"
- [discussion] memory demands of RMT approach: "it necessitates the computation of all eigenvalues of the large cell-to-cell similarity matrix, requiring significant memory resources"
