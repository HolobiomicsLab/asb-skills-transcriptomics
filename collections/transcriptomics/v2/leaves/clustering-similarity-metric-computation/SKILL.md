---
name: clustering-similarity-metric-computation
description: Use when benchmarking a dimensionality reduction or clustering variant
  on scRNA-seq embeddings and you need quantitative agreement scores (ECS, kNN-overlap)
  against ground-truth labels or a reference method's embedding.
when_to_use_negative:
- Input is raw count data that has not yet been preprocessed and embedded — QC via
  scLENS.preprocess and the scLENS/scLENS-med runs must precede metric computation
- No ground-truth labels and no reference embedding exist, since both ECS and kNN-overlap
  are defined relative to a ground truth or a reference method
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scLENS
  role: Reference dimensionality reduction method whose embeddings and clusterings
    provide the baseline for ECS and kNN-overlap comparison
  repo: github.com/Mathbiomed/scLENS
- name: scLENSpy
  role: Python wrapper for running scLENS on gpu/cpu devices (gpu selected when CUDA.has_cuda())
    to generate embeddings for both variants
  repo: https://github.com/Mathbiomed/scLENSpy
- name: CSV
  role: Julia package used to write the aggregated benchmark table sclens_med_benchmark.csv
- name: DataFrames
  role: Julia package used to assemble per-dataset metric rows into the single benchmark
    table
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:7c3a590ce58eb1d613b019c07d7fd33e678af470384880e88f222b3ecf0cbf05
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CSV
  - CUDA
  - DataFrames
  - SparseArrays.jl
  - scLENS
  - scLENSpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# clustering-similarity-metric-computation

## Summary

Computes embedding-quality metrics — element-centric similarity (ECS) against ground-truth labels and average kNN-overlap across k from 5 to 50 — to quantify whether a modified scLENS variant (e.g., median scaling) preserves dimensionality reduction performance.

## When to use

Use when benchmarking a dimensionality reduction or clustering variant on scRNA-seq embeddings and you need quantitative agreement scores (ECS, kNN-overlap) against ground-truth labels or a reference method's embedding.

## When NOT to use

- Input is raw count data that has not yet been preprocessed and embedded — QC via scLENS.preprocess and the scLENS/scLENS-med runs must precede metric computation
- No ground-truth labels and no reference embedding exist, since both ECS and kNN-overlap are defined relative to a ground truth or a reference method

## Inputs

- scLENS embeddings (PCA/low-dimension representations) for reference and variant methods across the 58 benchmarking scRNA-seq datasets
- Ground-truth cell-type labels for each benchmarking dataset
- Runtime and memory measurements (peak CPU memory, GPU memory, wall-clock time) recorded per dataset and device ('gpu' if CUDA.has_cuda(), else 'cpu')

## Outputs

- Per-dataset ECS scores from hierarchical/graph-based clustering vs. ground-truth labels for both scLENS and scLENS-med
- Average kNN-overlap scores with k varied from 5 to 50 for both variants
- Aggregated comparison table sclens_med_benchmark.csv with one row per dataset and columns for CPU/GPU memory, runtime, ECS, and kNN-overlap for both variants

## How to apply

For each benchmarking dataset, run both the reference method (original scLENS with z-score gene scaling) and the variant (scLENS-med with median scaling replacing the mean in the log-normalization gene-scaling step, preserving sparsity via SparseArrays.jl). Derive clusterings from the resulting embeddings using hierarchical/graph-based clustering and compare them to ground-truth labels using element-centric similarity (ECS). Additionally compute the average kNN-overlap score, varying k from 5 to 50, to measure local-neighborhood preservation relative to the reference embedding. Aggregate metrics per dataset — alongside CPU/GPU peak memory and runtime — into a single table (e.g., sclens_med_benchmark.csv) with one row per dataset. Accept the variant if ECS is similar to the reference and average kNN-overlap is only slightly lower, alongside the expected efficiency gains (~1.7x faster, ~2.4x lower CPU memory).

## Related tools

- **scLENS** (Reference dimensionality reduction method whose embeddings and clusterings provide the baseline for ECS and kNN-overlap comparison) — github.com/Mathbiomed/scLENS
- **scLENSpy** (Python wrapper for running scLENS on gpu/cpu devices (gpu selected when CUDA.has_cuda()) to generate embeddings for both variants) — https://github.com/Mathbiomed/scLENSpy
- **CSV** (Julia package used to write the aggregated benchmark table sclens_med_benchmark.csv)
- **DataFrames** (Julia package used to assemble per-dataset metric rows into the single benchmark table)

## Examples

```
cur_dev = has_cuda() ? "gpu" : "cpu"; sclens_embedding = scLENS.sclens(pre_df, device_=cur_dev); CSV.write("sclens_med_benchmark.csv", metrics_df)
```

## Evaluation signals

- scLENS-med shows ECS similar to (slightly lower than) the original scLENS across the 58 benchmarking datasets
- Average kNN-overlap of scLENS-med is slightly lower than scLENS on average, with k swept from 5 to 50
- The output sclens_med_benchmark.csv has exactly one row per dataset with columns for CPU/GPU memory, runtime, ECS, and kNN-overlap for both variants
- Efficiency expectations hold alongside metric similarity: ~1.7x faster runtime, ~2.4x lower CPU memory, ~1.6x lower GPU memory for scLENS-med
- Device selection recorded consistently: 'gpu' when CUDA is available via CUDA.has_cuda(), 'cpu' otherwise

## Limitations

- Metrics are relative: ECS requires ground-truth labels and kNN-overlap requires a reference embedding, so quality cannot be assessed without them
- scLENS-med trades a small amount of embedding quality (similar ECS, slightly lower average kNN-overlap) for its memory and speed gains
- RMT-based signal detection in scLENS requires computing all eigenvalues of the large cell-to-cell similarity matrix, which demands significant memory resources and limits scalability

## Evidence

- [other] ECS and kNN-overlap used to compare variants: [evidence span withheld — anchor status: not_found]
- [discussion] Median scaling replacing z-score scaling: "we considerably reduced the memory demand of scLENS by replacing conventional gene scaling (z-score scaling) with median scaling"
- [abstract] scLENS outperforms competing dimensionality reduction tools: "Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets with high sparsity and variability"
- [readme] GPU/CPU device selection in scLENS: "scLENS supports both GPU and CPU execution, and if CUDA is not available, the package will automatically use the CPU."
- [discussion] RMT eigenvalue computation is memory intensive: "it necessitates the computation of all eigenvalues of the large cell-to-cell similarity matrix, requiring significant memory resources"
