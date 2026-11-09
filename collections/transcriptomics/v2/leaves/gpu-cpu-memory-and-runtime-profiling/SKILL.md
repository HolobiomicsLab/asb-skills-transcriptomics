---
name: gpu-cpu-memory-and-runtime-profiling
description: Use when benchmarking scLENS or similar single-cell pipelines for memory
  footprint and speed on CPU vs. GPU, or when evaluating whether median scaling can
  replace z-score scaling to cut resource cost.
when_to_use_negative:
- The goal is dimensionality reduction quality alone (e.g. evaluating scLENS against
  11 other DR tools) rather than computational cost measurement
- Input data has already undergone feature selection of highly variable genes, which
  compromises low-variance local structure and changes the memory profile being measured
- No CUDA-capable GPU is present and GPU memory savings are the target metric — the
  package will silently fall back to CPU
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: scLENS
  role: Julia package whose sclens() run is being profiled on gpu/cpu devices; preprocess()
    applies QC thresholds before profiling
  repo: github.com/Mathbiomed/scLENS
- name: SparseArrays.jl
  role: Julia module used to allocate only non-zero values and their indices, preserving
    sparsity in the median-scaling implementation that reduces memory demand
- name: CUDA
  role: Device selection via CUDA.has_cuda() chooses 'gpu' when available, else 'cpu';
    GPU memory usage is one of the recorded metrics
- name: CSV
  role: Writes the aggregated sclens_med_benchmark.csv results table
- name: DataFrames
  role: Structures per-dataset measurement rows for the aggregated benchmark table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:4e5f3c4f7eba9c83594778f69f2c9a05b0ffe025f5949253182088eb19d710b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CSV
  - CUDA
  - DataFrames
  - SparseArrays.jl
  - scLENS
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# GPU/CPU memory and runtime profiling

## Summary

Profiles peak CPU memory, GPU memory, and wall-clock runtime of a Julia-based dimensionality reduction pipeline (scLENS vs. its median-scaling variant scLENS-med) across datasets of varying cell counts, to quantify the computational savings of replacing z-score gene scaling with median scaling.

## When to use

Use when benchmarking scLENS or similar single-cell pipelines for memory footprint and speed on CPU vs. GPU, or when evaluating whether median scaling can replace z-score scaling to cut resource cost.

## When NOT to use

- The goal is dimensionality reduction quality alone (e.g. evaluating scLENS against 11 other DR tools) rather than computational cost measurement
- Input data has already undergone feature selection of highly variable genes, which compromises low-variance local structure and changes the memory profile being measured
- No CUDA-capable GPU is present and GPU memory savings are the target metric — the package will silently fall back to CPU

## Inputs

- Preprocessed single-cell count matrices (cells x genes) after scLENS.preprocess QC: cells with <200 expressed genes filtered, genes expressed in <15 cells filtered, cells with >5% mitochondrial reads filtered
- 39 datasets used in the memory analysis and 58 benchmarking datasets
- Four large datasets of ~20,000 genes with 25,000 to 100,000 cells
- NVIDIA GPU with CUDA capability and appropriate drivers (optional)

## Outputs

- sclens_med_benchmark.csv with one row per dataset and columns for CPU memory, GPU memory, runtime, ECS, and kNN-overlap for both variants
- CPU RAM and runtime measurements at each cell-count size for the large datasets
- Peak GPU memory and wall-clock time per scLENS run per device

## How to apply

Run both the original scLENS and a median-scaled variant (scLENS-med, where the subtraction value in the gene-scaling step of log normalization is the per-gene median instead of the mean, with only non-zero values and indices allocated via SparseArrays.jl to preserve sparsity) on the same 39 memory-analysis datasets. Select the device via CUDA.has_cuda(): use 'gpu' when CUDA is available and 'cpu' otherwise, recording peak CPU memory, GPU memory, and wall-clock time per run. Compute embedding-quality metrics (ECS from hierarchical/graph-based clustering against ground-truth labels, and average kNN-overlap with k varied from 5 to 50) on the 58 benchmarking datasets to confirm performance is preserved. Additionally run scLENS-med on four large datasets of roughly 20,000 genes with 25,000–100,000 cells to characterize how CPU RAM scales with cell count. Aggregate all measurements into sclens_med_benchmark.csv with one row per dataset.

## Related tools

- **scLENS** (Julia package whose sclens() run is being profiled on gpu/cpu devices; preprocess() applies QC thresholds before profiling) — github.com/Mathbiomed/scLENS
- **SparseArrays.jl** (Julia module used to allocate only non-zero values and their indices, preserving sparsity in the median-scaling implementation that reduces memory demand)
- **CUDA** (Device selection via CUDA.has_cuda() chooses 'gpu' when available, else 'cpu'; GPU memory usage is one of the recorded metrics)
- **CSV** (Writes the aggregated sclens_med_benchmark.csv results table)
- **DataFrames** (Structures per-dataset measurement rows for the aggregated benchmark table)

## Examples

```
using CUDA: has_cuda; cur_dev = has_cuda() ? "gpu" : "cpu"; pre_df = scLENS.preprocess(ndf); sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev)
```

## Evaluation signals

- scLENS-med shows ~2.4x lower CPU memory and ~1.6x lower GPU memory than original scLENS on average
- scLENS-med runtime is ~1.7x faster than original scLENS
- ECS values are similar between variants and average kNN-overlap is only slightly lower for scLENS-med
- CPU RAM grows from ~23 GB at 25,000 cells to ~93 GB at 100,000 cells for the large ~20,000-gene datasets
- Output CSV has one row per dataset with columns for CPU/GPU memory, runtime, ECS, and kNN-overlap for both variants

## Limitations

- RMT-based noise filtering requires computing all eigenvalues of the large cell-to-cell similarity matrix, so scLENS on large data with whole genes still requires a computer with high specifications
- Median scaling yields slightly lower average kNN-overlap than the original scLENS, a modest performance trade-off for the memory/speed savings
- CUDA is required for GPU profiling; without an NVIDIA GPU with appropriate drivers the package automatically falls back to CPU

## Evidence

- [discussion] Median scaling used in place of conventional z-score gene scaling: "we considerably reduced the memory demand of scLENS by replacing conventional gene scaling (z-score scaling) with median scaling"
- [discussion] RMT eigenvalue computation drives memory demand: "it necessitates the computation of all eigenvalues of the large cell-to-cell similarity matrix, requiring significant memory resources"
- [readme] Automatic GPU/CPU device selection: "scLENS supports both GPU and CPU execution, and if CUDA is not available, the package will automatically use the CPU."
- [readme] GPU hardware requirement for CUDA runs: "To use CUDA, you must have an NVIDIA GPU with CUDA capability, and the appropriate NVIDIA drivers must be installed on your system."
- [readme] QC thresholds applied before profiling: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality."
