---
name: large-scale-scrnaseq-preprocessing
description: Use when raw UMI count matrices from multi-batch scRNA-seq datasets (thousands
  to tens of thousands of cells) must be normalized, log-transformed, scaled, and
  filtered to highly variable genes before deep-learning clustering such as DESC.
when_to_use_negative:
- Input is already normalized and log-transformed — reapplying UMI-per-10k normalization
  and log1p would distort the data.
- The goal is full-gene analysis requiring all genes, since HVG selection with filter_genes_dispersion
  (n_top_genes=1000) discards the remainder.
- The dataset contains fewer than a handful of batches where batch effect removal
  is not a concern; DESC's iterative self-learning assumes technical differences across
  batches are smaller than true biological variations.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Performs cell-level UMI normalization, natural log transform (sc.pp.log1p),
    highly variable gene selection (filter_genes_dispersion), and gene scaling (sc.pp.scale,
    max_value=6) to prepare the input matrix.
  repo: https://github.com/theislab/scanpy
- name: DESC
  role: Downstream deep embedding clustering on the preprocessed matrix, with autoencoder
    dims=[n_genes,64,32] and tol=0.005, using Keras/TensorFlow and GPU when available.
  repo: https://github.com/eleozzr/desc
- name: desc 2.0.3
  role: TensorFlow 1-compatible release of DESC that reproduces the paper's benchmark
    results.
  repo: https://github.com/eleozzr/desc
- name: Keras
  role: Backend used to train the DESC autoencoder on the preprocessed gene expression
    matrix.
- name: Seurat 3.0
  role: Comparison method implementing MNN run on the identical preprocessed 30-batch
    input for runtime benchmarking.
- name: scVI
  role: Comparison deep-learning method consuming the same preprocessed input during
    runtime benchmarking.
- name: BERMUDA
  role: Comparison method whose runtime (~11.2 h for 30 batches/30,000 cells) and
    peak memory (>32 GB) are recorded against DESC.
- name: Scanorama
  role: Pairwise comparison method timed end-to-end on the same preprocessed input
    (about 0.4 h for 30 batches).
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:44ddbbe76bb1567b011452974372b78ce3621c937c5535ac0a6dfa11710fc2ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scanpy
  tools:
  - BERMUDA
  - DESC
  - Keras
  - Scanorama
  - Scanpy
  - Seurat 3.0
  - desc 2.0.3
  - scVI
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# large-scale-scrnaseq-preprocessing

## Summary

Preprocess large scRNA-seq count matrices (e.g., 30,000 macaque retina cells across 30 sample-id batches) with Scanpy cell-level UMI normalization, natural log transform, gene-level scaling, and highly variable gene selection via filter_genes_dispersion, producing input suitable for DESC and comparison methods.

## When to use

Use when raw UMI count matrices from multi-batch scRNA-seq datasets (thousands to tens of thousands of cells) must be normalized, log-transformed, scaled, and filtered to highly variable genes before deep-learning clustering such as DESC.

## When NOT to use

- Input is already normalized and log-transformed — reapplying UMI-per-10k normalization and log1p would distort the data.
- The goal is full-gene analysis requiring all genes, since HVG selection with filter_genes_dispersion (n_top_genes=1000) discards the remainder.
- The dataset contains fewer than a handful of batches where batch effect removal is not a concern; DESC's iterative self-learning assumes technical differences across batches are smaller than true biological variations.

## Inputs

- Raw UMI count matrix from a multi-batch scRNA-seq dataset (e.g., macaque retina, 30,000 cells, 30 batches defined by sample id)
- Batch/sample-id annotation per cell
- Target number of highly variable genes (e.g., n_top_genes=1000)

## Outputs

- Cell-level normalized, natural-log-transformed expression matrix
- Filtered matrix restricted to highly variable genes (filter_genes_dispersion)
- Gene-standardized matrix (scaled with max_value=6) ready for DESC / MNN / scVI / BERMUDA / scanorama
- Per-method runtime and peak-memory table (e.g., runtime_benchmark_macaque.csv) when used for the 30-batch benchmark

## How to apply

Load the macaque retina scRNA-seq dataset (or equivalent multi-batch scRNA-seq data) and, for runtime benchmarks, subsample 30,000 cells with batches defined by sample id. Apply cell-level normalization by dividing UMI counts per gene by total UMIs in the cell, multiplying by 10,000, and transforming to a natural log scale (sc.pp.log1p after Scanpy normalization). Select highly variable genes using the filter_genes_dispersion function from Scanpy (sc.pp.filter_genes_dispersion with n_top_genes=1000), then standardize each gene by subtracting its mean and dividing by its standard deviation across cells (sc.pp.scale with max_value=6). This preprocessing produces the normalized, HVG-reduced expression matrix that DESC consumes with an autoencoder of dims=[n_genes,64,32] and tol=0.005; the identical preprocessed input must be fed to comparison methods (MNN/Seurat 3.0, scVI, BERMUDA, Scanorama) so runtime comparisons are fair.

## Related tools

- **Scanpy** (Performs cell-level UMI normalization, natural log transform (sc.pp.log1p), highly variable gene selection (filter_genes_dispersion), and gene scaling (sc.pp.scale, max_value=6) to prepare the input matrix.) — https://github.com/theislab/scanpy
- **DESC** (Downstream deep embedding clustering on the preprocessed matrix, with autoencoder dims=[n_genes,64,32] and tol=0.005, using Keras/TensorFlow and GPU when available.) — https://github.com/eleozzr/desc
- **desc 2.0.3** (TensorFlow 1-compatible release of DESC that reproduces the paper's benchmark results.) — https://github.com/eleozzr/desc
- **Keras** (Backend used to train the DESC autoencoder on the preprocessed gene expression matrix.)
- **Seurat 3.0** (Comparison method implementing MNN run on the identical preprocessed 30-batch input for runtime benchmarking.)
- **scVI** (Comparison deep-learning method consuming the same preprocessed input during runtime benchmarking.)
- **BERMUDA** (Comparison method whose runtime (~11.2 h for 30 batches/30,000 cells) and peak memory (>32 GB) are recorded against DESC.)
- **Scanorama** (Pairwise comparison method timed end-to-end on the same preprocessed input (about 0.4 h for 30 batches).)

## Examples

```
import scanpy as sc; adata = sc.datasets.paul15(); sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); sc.pp.scale(adata, max_value=6)
```

## Evaluation signals

- Measured DESC runtime for 30,000 cells / 30 batches falls within a reasonable tolerance of the reported ~0.3 h (scanorama ~0.4 h, MNN ~9.5 h, scVI ~10.4 h, BERMUDA ~11.2 h).
- BERMUDA's recorded peak memory exceeds 32 GB, confirming the benchmark captured the memory contrast with DESC's small memory footprint.
- Preprocessing chain matches the paper exactly: UMI-per-10k normalization → natural log transform → filter_genes_dispersion HVG selection → gene-level standardization (sc.pp.scale max_value=6).
- All comparison methods receive the identical preprocessed 30-batch input matrix, so runtimes are directly comparable.
- The compiled runtime_benchmark_macaque.csv contains per-method runtime and peak-memory entries for all five methods.

## Limitations

- DESC's iterative batch-effect removal only works when technical differences across batches are smaller than true biological variations.
- Reproducing the paper's results requires TensorFlow < 2.0 (desc 2.0.3) and Python 3.5.x or 3.6.x, per the DESC README.
- Pairwise comparison methods (MNN, Seurat 3.0, Scanorama) are order-dependent: the order in which batches are corrected affects the final results.
- BERMUDA may require more than 32 GB of memory at this scale, making it infeasible on typical workstations.
- No software version numbers are stated in the article for Keras or Scanpy, so exact reproducibility of preprocessing may vary with versions.

## Evidence

- [discussion] Cell-level normalization with UMI scaling: "the UMI count for each gene in each cell is divided by the total number of UMIs in the cell, multiplied by 10,000, and then transformed to a natural log scale."
- [discussion] HVG selection with Scanpy: "Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25"
- [methods] Gene-level scaling: "sc.pp.scale(adata,max_value=6)"
- [results] Runtime benchmark results at 30 batches / 30,000 cells: [evidence span withheld — anchor status: not_found]
- [readme] DESC version constraints from README: "you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper"
