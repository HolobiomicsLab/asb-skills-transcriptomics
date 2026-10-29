---
name: runtime-and-memory-profiling
description: Use when comparing computational scalability of scRNA-seq clustering
  or batch-effect-removal methods on a fixed dataset size (e.g. 30,000 cells, 30 batches)
  and you need per-method runtime and peak-memory numbers.
when_to_use_negative:
- The goal is clustering accuracy (ARI) or batch-mixing quality rather than computational
  cost — the runtime skill does not evaluate clustering performance.
- The dataset size is small (e.g. hundreds to a few thousand cells), where the reported
  scaling differences (0.3 h vs ~10 h) do not manifest and benchmarking is uninformative.
- Only one method's output is needed and no cross-method scalability comparison is
  intended; a single timed run without the common 30-batch input and memory monitoring
  is not this benchmark.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DESC
  role: Benchmarked deep learning clustering method; run with dims=[n_genes,64,32],
    tol=0.005, GPU when available
  repo: https://github.com/eleozzr/desc
- name: Seurat 3.0
  role: Provides the MNN implementation (MNN+CCA) used as a comparison method in the
    runtime benchmark
- name: scVI
  role: Benchmarked batch-aware deep learning comparison method
- name: BERMUDA
  role: Benchmarked batch-aware method; peak memory explicitly checked against the
    32 GB limit
- name: Scanorama
  role: Benchmarked pairwise comparison method
- name: Scanpy
  role: Preprocessing pipeline (UMI normalization, log transform, filter_genes_dispersion)
    producing the identical input fed to all benchmarked methods
  repo: https://github.com/theislab/scanpy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:f8b588d5664be5ef3aae5ee5fa42203204a67b59c2737cdee275115904bb4b21
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# runtime-and-memory-profiling

## Summary

Benchmark the wall-clock running time and peak memory of scRNA-seq clustering/batch-correction methods (DESC, MNN, scVI, BERMUDA, Scanorama) on a fixed cell count and batch count, here 30,000 macaque retina cells across 30 sample-defined batches, to quantify computational scalability.

## When to use

Use when comparing computational scalability of scRNA-seq clustering or batch-effect-removal methods on a fixed dataset size (e.g. 30,000 cells, 30 batches) and you need per-method runtime and peak-memory numbers.

## When NOT to use

- The goal is clustering accuracy (ARI) or batch-mixing quality rather than computational cost — the runtime skill does not evaluate clustering performance.
- The dataset size is small (e.g. hundreds to a few thousand cells), where the reported scaling differences (0.3 h vs ~10 h) do not manifest and benchmarking is uninformative.
- Only one method's output is needed and no cross-method scalability comparison is intended; a single timed run without the common 30-batch input and memory monitoring is not this benchmark.

## Inputs

- Macaque retina scRNA-seq count matrix subsampled to 30,000 cells with 30 batches defined by sample id
- Scanpy-preprocessed expression matrix (UMI-normalized, log-transformed, highly variable gene selected)
- Method configurations: DESC autoencoder dims=[n_genes,64,32] with tol=0.005; MNN as implemented in Seurat 3.0; scVI; BERMUDA; Scanorama

## Outputs

- runtime_benchmark_macaque.csv: per-method wall-clock runtime (hours) and peak memory usage
- Explicit flag of whether BERMUDA's peak memory exceeds 32 GB
- Validation report comparing measured runtimes to reported values (DESC 0.3 h, MNN 9.5 h, scVI 10.4 h, BERMUDA 11.2 h, Scanorama 0.4 h)

## How to apply

Load the macaque retina scRNA-seq dataset, subsample 30,000 cells, and assign batches by sample id (30 samples = 30 batches). Preprocess the count matrix identically for all methods with Scanpy: cell-level UMI normalization (divide by total UMIs, multiply by 10,000, natural log transform), highly variable gene selection with filter_genes_dispersion, and gene-level scaling. Run DESC with the benchmarked autoencoder configuration (dims=[n_genes,64,32], tol=0.005, GPU if available) and the comparison methods MNN (Seurat 3.0 implementation), scVI, BERMUDA, and Scanorama end-to-end on the same 30-batch input, timing each with a wall-clock timer. For every method, record peak memory usage, explicitly flagging whether BERMUDA exceeds 32 GB. Compile per-method runtime and peak memory into runtime_benchmark_macaque.csv and validate measured times against the published reference values (DESC 0.3 h, MNN 9.5 h, scVI 10.4 h, BERMUDA 11.2 h, Scanorama 0.4 h) within a reasonable tolerance.

## Related tools

- **DESC** (Benchmarked deep learning clustering method; run with dims=[n_genes,64,32], tol=0.005, GPU when available) — https://github.com/eleozzr/desc
- **Seurat 3.0** (Provides the MNN implementation (MNN+CCA) used as a comparison method in the runtime benchmark)
- **scVI** (Benchmarked batch-aware deep learning comparison method)
- **BERMUDA** (Benchmarked batch-aware method; peak memory explicitly checked against the 32 GB limit)
- **Scanorama** (Benchmarked pairwise comparison method)
- **Scanpy** (Preprocessing pipeline (UMI normalization, log transform, filter_genes_dispersion) producing the identical input fed to all benchmarked methods) — https://github.com/theislab/scanpy

## Examples

```
import scanpy as sc; sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata,n_top_genes=1000); sc.pp.scale(adata,max_value=6)
```

## Evaluation signals

- Measured runtimes fall within a reasonable tolerance of the reported values: DESC 0.3 h, MNN 9.5 h, scVI 10.4 h, BERMUDA 11.2 h, Scanorama 0.4 h for 30 batches / 30,000 cells.
- All five methods receive the identical 30-batch Scanpy-preprocessed input (same normalization, log transform, and highly variable gene selection), so timing differences are attributable to the methods.
- Peak memory is recorded for every method, and BERMUDA's recorded peak memory is confirmed to exceed 32 GB.
- runtime_benchmark_macaque.csv contains one runtime and one peak-memory entry per method with no missing rows.
- DESC timing reflects GPU availability (GPU-enabled run expected to be at the fast end of the reported range).

## Limitations

- BERMUDA's memory footprint exceeds 32 GB at this scale, so it may fail on machines with less RAM.
- Pairwise batch-correction methods (CCA, MNN, Seurat 3.0, Scanorama) are order-dependent, so batch ordering must be held fixed across runs for comparable timings.
- Runtime results depend on hardware and GPU availability; DESC's small memory footprint and GPU utilization give it an advantage that may not transfer to CPU-only systems.
- No software version numbers were stated for Keras or Scanpy, so exact reproducibility of the reference timings is not guaranteed.

## Evidence

- [abstract] DESC has a small memory footprint and can use GPU: "DESC offers a proper balance of clustering accuracy and stability, has a small footprint on memory, does not explicitly require batch information for batch effect removal, and can utilize GPU when"
- [discussion] Scanpy cell-level normalization used as shared preprocessing: "the UMI count for each gene in each cell is divided by the total number of UMIs in the cell, multiplied by 10,000, and then transformed to a natural log scale."
- [discussion] Highly variable gene selection step shared by all benchmarked methods: "Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25"
- [methods] Concrete preprocessing call grounding the example invocation: "sc.pp.filter_genes_dispersion(adata,n_top_genes=1000)"
- [readme] DESC version/dependency constraint relevant to reproducing benchmark timings: "Note: Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
