---
name: scalability-benchmarking-of-clustering-methods
description: Use when comparing how clustering/batch-correction methods scale with
  many batches and tens of thousands of cells, or when a candidate method must be
  ruled in/out on computing time or memory (e.g. a 32 GB limit).
when_to_use_negative:
- The question is clustering accuracy (ARI against reference cell-type labels) rather
  than runtime or memory scaling — use the ARI evaluation workflow instead
- The dataset has no batch structure (e.g. a single batch like the paul15 myeloid
  progenitor data), so batch-count scaling cannot be exercised
- Working memory is far below 32 GB and BERMUDA must be run anyway — its recorded
  peak memory exceeds 32 GB, so it cannot complete and will break the benchmark
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3473
tools:
- name: DESC
  role: Method under benchmark; run with autoencoder dims=[n_genes,64,32], tol=0.005,
    GPU if available, timing end-to-end
  repo: https://github.com/eleozzr/desc
- name: Seurat 3.0
  role: Provides the MNN + CCA comparison implementation (MNN) timed on the identical
    30-batch input
- name: scVI
  role: Batch-aware deep-learning comparison method timed end-to-end on the same input
- name: BERMUDA
  role: Batch-aware comparison method; timed and monitored for peak memory exceeding
    32 GB
- name: Scanorama
  role: Pairwise batch-correction comparison method timed end-to-end (reported ~0.4
    h at 30 batches / 30,000 cells)
- name: Scanpy
  role: Single shared preprocessing path (UMI normalization to 10,000, log1p, filter_genes_dispersion
    HVG selection) applied before all methods
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Backend used to train the DESC autoencoder during the benchmarked runs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:46260cbf34bb0670c16d26c07221f3bdddfcda5d14e2eb781bee0835513b56d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - BERMUDA
  - DESC
  - Keras
  - Scanorama
  - Scanpy
  - Seurat 3.0
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 20
---

# scalability-benchmarking-of-clustering-methods

## Summary

Benchmark the wall-clock runtime and peak memory footprint of scRNA-seq clustering and batch-correction methods (DESC, MNN/Seurat 3.0, scVI, BERMUDA, scanorama) on a fixed batch-structured subsample (e.g. 30 batches, 30,000 cells of the macaque retina dataset), to establish which methods scale to large atlas-like studies.

## When to use

Use when comparing how clustering/batch-correction methods scale with many batches and tens of thousands of cells, or when a candidate method must be ruled in/out on computing time or memory (e.g. a 32 GB limit).

## When NOT to use

- The question is clustering accuracy (ARI against reference cell-type labels) rather than runtime or memory scaling — use the ARI evaluation workflow instead
- The dataset has no batch structure (e.g. a single batch like the paul15 myeloid progenitor data), so batch-count scaling cannot be exercised
- Working memory is far below 32 GB and BERMUDA must be run anyway — its recorded peak memory exceeds 32 GB, so it cannot complete and will break the benchmark

## Inputs

- Macaque retina scRNA-seq count matrix subsampled to 30,000 cells with 30 batches defined by sample id
- Scanpy-preprocessed log-normalized expression matrix with highly variable genes selected via filter_genes_dispersion
- DESC autoencoder configuration (dims=[n_genes,64,32], tol=0.005) and installed versions of Seurat 3.0, scVI, BERMUDA, scanorama, Keras

## Outputs

- Per-method wall-clock runtime table for 30 batches / 30,000 cells (runtime_benchmark_macaque.csv)
- Per-method peak memory usage records, including an explicit flag for whether BERMUDA exceeds 32 GB

## How to apply

Load the macaque retina scRNA-seq dataset, subsample 30,000 cells, and define 30 batches by sample id so every method receives an identical batch-structured input. Preprocess once with Scanpy: cell-level UMI normalization to 10,000, natural log transform (sc.pp.log1p), and highly variable gene selection with sc.pp.filter_genes_dispersion (n_top_genes=1000). Run DESC with the benchmarked autoencoder configuration (dims=[n_genes,64,32], tol=0.005, GPU available) and the comparison methods MNN as implemented in Seurat 3.0, scVI, BERMUDA, and scanorama end-to-end on the same matrix, timing each run and recording peak memory per method (explicitly checking whether BERMUDA exceeds 32 GB). The rationale is that DESC claims a small memory footprint and GPU acceleration, so runtime/memory — not only ARI — is a decision criterion; compile per-method runtime and peak memory into a table (runtime_benchmark_macaque.csv) and validate against reported values (DESC 0.3 h, MNN 9.5 h, scVI 10.4 h, BERMUDA 11.2 h, scanorama 0.4 h).

## Related tools

- **DESC** (Method under benchmark; run with autoencoder dims=[n_genes,64,32], tol=0.005, GPU if available, timing end-to-end) — https://github.com/eleozzr/desc
- **Seurat 3.0** (Provides the MNN + CCA comparison implementation (MNN) timed on the identical 30-batch input)
- **scVI** (Batch-aware deep-learning comparison method timed end-to-end on the same input)
- **BERMUDA** (Batch-aware comparison method; timed and monitored for peak memory exceeding 32 GB)
- **Scanorama** (Pairwise batch-correction comparison method timed end-to-end (reported ~0.4 h at 30 batches / 30,000 cells))
- **Scanpy** (Single shared preprocessing path (UMI normalization to 10,000, log1p, filter_genes_dispersion HVG selection) applied before all methods) — https://github.com/theislab/scanpy
- **Keras** (Backend used to train the DESC autoencoder during the benchmarked runs)

## Examples

```
import scanpy as sc; sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata,n_top_genes=1000)
```

## Evaluation signals

- Measured runtimes fall within a reasonable tolerance of the reported values: DESC 0.3 h, MNN 9.5 h, scVI 10.4 h, BERMUDA 11.2 h, scanorama 0.4 h for 30 batches / 30,000 cells
- BERMUDA's recorded peak memory exceeds 32 GB (explicit monitoring step in the benchmark)
- runtime_benchmark_macaque.csv contains one runtime and one peak-memory value per method (DESC, MNN, scVI, BERMUDA, scanorama)
- All five methods were run on the identical input: 30,000 cells, 30 batches defined by sample id, same Scanpy preprocessing (log1p, n_top_genes=1000 HVGs)
- DESC run uses the benchmarked configuration dims=[n_genes,64,32], tol=0.005, and timing covers the full end-to-end analysis

## Limitations

- BERMUDA's peak memory exceeds 32 GB at this scale, so it cannot be benchmarked on memory-constrained machines
- DESC's exact reproducibility depends on the TensorFlow version: the README states results matching the paper require tensorflow < 2.0 (desc 2.0.3); no version numbers are stated for Keras or Scanpy
- Pairwise comparison methods (CCA, MNN, Seurat 3.0, Scanorama) are order-dependent on batches, which can affect results and complicates fair timing comparisons

## Evidence

- [results] Runtime benchmark at 30 batches and 30,000 cells: "the computing time for analysis of 30,000 cells is about 0.3 h for DESC, 9.5 h for MNN, 10.4 h for scVI, 11.2 h for BERMUDA, and 0.4 h for scanorama"
- [discussion] Scanpy HVG preprocessing: "Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25 (https://github.com/theislab/scanpy)."
- [methods] Cell-level normalization scheme shared across methods: "the UMI count for each gene in each cell is divided by the total number of UMIs in the cell, multiplied by 10,000, and then transformed to a natural log scale."
- [abstract] DESC small memory footprint and GPU claim motivating runtime benchmark: "DESC offers a proper balance of clustering accuracy and stability, has a small footprint on memory, does not explicitly require batch information for batch effect removal, and can utilize GPU when"
- [readme] DESC reproducibility depends on TensorFlow version: "you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
