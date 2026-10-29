---
name: sparse-matrix-memory-optimization
description: Use when running RMT-based dimensionality reduction (e.g. scLENS) on
  whole-gene scRNA-seq matrices where eigenvalue computation exhausts CPU/GPU memory,
  and median-scaling with sparse storage is an acceptable trade-off.
when_to_use_negative:
- When the input matrix is already dense/small enough to fit in memory — the ~1.7x
  speedup and memory savings do not justify modifying the pipeline
- When downstream analysis requires exact z-score-normalized gene values, since median
  scaling changes the scaling statistic
- When feature selection of highly variable genes has already been applied (gene subsetting),
  rather than whole-gene matrices where memory is the bottleneck
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scLENS
  role: Base dimensionality reduction pipeline (Julia) whose gene-scaling step is
    modified for median scaling; provides preprocess QC, sclens embedding, and device
    selection
  repo: https://github.com/Mathbiomed/scLENS
- name: SparseArrays.jl
  role: Julia module used to allocate only non-zero values and their indices, preserving
    matrix sparsity through median scaling
- name: CUDA
  role: GPU backend; device selected via CUDA.has_cuda() with fallback to CPU
- name: CSV / DataFrames.jl
  role: Reading count files and writing the benchmark results table
- name: scLENSpy
  role: Python wrapper of scLENS; requires PyTorch installed with CUDA support separately
  repo: https://github.com/Mathbiomed/scLENSpy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:2aff99e0d6cf7ec83f2a29a9564e77dc78befe2fc9679d1fc7b3756dbabc82a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
  tools:
  - CSV
  - CSV / DataFrames.jl
  - CUDA
  - DataFrames
  - SparseArrays.jl
  - scLENS
  - scLENSpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# sparse-matrix-memory-optimization

## Summary

Reduces the memory footprint and runtime of scLENS-style single-cell pipelines by replacing z-score gene scaling with median scaling during log normalization and preserving sparsity via SparseArrays.jl, so large scRNA-seq count matrices fit on commodity CPU/GPU hardware.

## When to use

Use when running RMT-based dimensionality reduction (e.g. scLENS) on whole-gene scRNA-seq matrices where eigenvalue computation exhausts CPU/GPU memory, and median-scaling with sparse storage is an acceptable trade-off.

## When NOT to use

- When the input matrix is already dense/small enough to fit in memory — the ~1.7x speedup and memory savings do not justify modifying the pipeline
- When downstream analysis requires exact z-score-normalized gene values, since median scaling changes the scaling statistic
- When feature selection of highly variable genes has already been applied (gene subsetting), rather than whole-gene matrices where memory is the bottleneck

## Inputs

- scRNA-seq count matrix as CSV (rows=cells, columns=genes, first column cell IDs) or JLD2 with a 'data' DataFrame having a :cell column
- 10x triplet files (matrix.mtx.gz, features.tsv.gz, barcodes.tsv.gz) convertible via scLENS.tenx2jld2
- Optional gene dictionary CSV with 'gene' and 'gene_ID' columns

## Outputs

- Median-scaled sparse embedding from scLENS.sclens (PCA and UMAP coordinates)
- Benchmark table sclens_med_benchmark.csv with per-dataset CPU/GPU memory, runtime, ECS, and kNN-overlap for both variants
- Saved results as CSV or AnnData (.h5ad) via scLENS.save_anndata

## How to apply

Modify the gene-scaling step of the log-normalization pipeline so the value subtracted from each gene's log-transformed expression is the gene's median instead of the mean (replacing conventional z-score scaling), which the article identifies as the change that 'considerably reduced the memory demand of scLENS'. Store the count matrix in a sparse format, allocating only non-zero values and their indices via the SparseArrays.jl module so sparsity (often >90% zeros) is preserved through scaling. Run the pipeline with device selection driven by CUDA.has_cuda() — 'gpu' when available, 'cpu' otherwise — applying the same QC as the baseline (scLENS.preprocess filtering cells with <200 expressed genes and genes expressed in <15 cells). Benchmark peak CPU memory, GPU memory, and wall-clock time against original scLENS on the same datasets (39 datasets for memory, 58 for quality), and validate that embedding quality (ECS, kNN-overlap k=5–50) stays similar: expect ~2.4x lower CPU memory, ~1.6x lower GPU memory, and ~1.7x faster runtime.

## Related tools

- **scLENS** (Base dimensionality reduction pipeline (Julia) whose gene-scaling step is modified for median scaling; provides preprocess QC, sclens embedding, and device selection) — https://github.com/Mathbiomed/scLENS
- **SparseArrays.jl** (Julia module used to allocate only non-zero values and their indices, preserving matrix sparsity through median scaling)
- **CUDA** (GPU backend; device selected via CUDA.has_cuda() with fallback to CPU)
- **CSV / DataFrames.jl** (Reading count files and writing the benchmark results table)
- **scLENSpy** (Python wrapper of scLENS; requires PyTorch installed with CUDA support separately) — https://github.com/Mathbiomed/scLENSpy

## Examples

```
using scLENS, CUDA: has_cuda; cur_dev = has_cuda() ? "gpu" : "cpu"; pre_df = scLENS.preprocess(scLENS.read_file("data/Z8eq.csv.gz")); emb = scLENS.sclens(pre_df, device_=cur_dev)
```

## Evaluation signals

- scLENS-med shows ~2.4x lower CPU memory and ~1.6x lower GPU memory than original scLENS on the 39 memory-analysis datasets
- Runtime improves ~1.7x versus the original pipeline on identical hardware
- ECS from hierarchical/graph-based clustering against ground-truth labels remains similar to original scLENS, with slightly lower average kNN-overlap (k varied 5–50) across the 58 benchmarking datasets
- On ~20,000-gene datasets, CPU RAM scales from ~23 GB at 25,000 cells to ~93 GB at 100,000 cells
- Sparse storage retains only non-zero values and their indices — verify the matrix type/type-stability through the scaling step

## Limitations

- RMT-based noise filtering requires computing all eigenvalues of the large cell-to-cell similarity matrix, which demands significant memory resources even after median scaling
- Embedding quality is slightly lower (kNN-overlap) than the original mean-scaled scLENS
- Even optimized, scLENS operation on large data with whole genes still requires a computer with high specifications
- GPU support requires an NVIDIA GPU with CUDA capability and appropriate drivers; otherwise execution falls back to CPU

## Evidence

- [discussion] we considerably reduced the memory demand of scLENS by replacing conventional gene scaling (z-score scaling) with median scaling: "we considerably reduced the memory demand of scLENS by replacing conventional gene scaling (z-score scaling) with median scaling"
- [discussion] RMT requires computing all eigenvalues of the similarity matrix: "it necessitates the computation of all eigenvalues of the large cell-to-cell similarity matrix, requiring significant memory resources"
- [discussion] its operation on large data with whole genes still requires a computer with high specifications: "its operation on large data with whole genes still requires a computer with high specifications"
- [readme] README: device selection and QC thresholds: "cur_dev = if has_cuda()
    "gpu"
else
    "cpu"
end"
- [readme] README: preprocess QC filtering criteria: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality."
- [readme] README: CUDA requirements and CPU fallback: "If CUDA is not available, the package will automatically use the CPU."
