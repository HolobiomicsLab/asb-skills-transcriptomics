---
name: scalability-analysis-of-clustering-pipelines
description: Use when quantifying how a scRNA-seq feature selection or clustering
  method's runtime and peak memory grow with cell count — e.g. comparing a correlation-based
  method against correlation-free baselines at 10k/100k/1M-cell scales.
when_to_use_negative:
- The question concerns clustering accuracy (e.g. silhouette index, ARI against cell-type
  annotations), not computational scaling — use the accuracy benchmarking skill instead
- The dataset is small (e.g. the 10X pbmc_1k_v2 PBMC dataset, ~1k cells), where memory/time
  scaling cannot be meaningfully profiled across orders of magnitude
- Only one feature selection method is of interest and no correlation-free baselines
  will be run, since the benchmark's value is the comparative scaling contrast
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DUBStepR
  role: Correlation-based feature selection method whose runtime and peak memory are
    profiled across the 10k/100k/1M-cell scale points
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Preprocessing (quality control, log-normalization) of the GSE119945 organogenesis
    counts and generation of downsampled Seurat objects at each scale
  repo: https://satijalab.org/seurat/
- name: hdf5r
  role: Loads the large GSE119945 count matrix into the Seurat object without exhausting
    memory
- name: R
  role: Execution environment (>= 3.5.0) providing system timing and memory profiling
    utilities to record runtime and peak memory per fresh session
- name: M3Drop
  role: Correlation-free baseline method (M3DropDANB) whose memory scaling is compared
    against DUBStepR
- name: GiniClust
  role: Baseline feature selection method that fails at 100k cells, a check that must
    be reproduced in the benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:981465881e6cf362d19014c9153e5ff12b5f8a4ca8a0b927b46c1b7a097c98ba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - GiniClust
  - M3Drop
  - R
  - Seurat
  - hdf5r
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# scalability-analysis-of-clustering-pipelines

## Summary

Benchmark how a single-cell feature selection/clustering pipeline's wall-clock runtime and peak memory scale with cell count, as done for DUBStepR on the Cao et al. mouse organogenesis scRNA-seq dataset (GSE119945), where memory grew 6.4x from 100k to 1M cells.

## When to use

Use when quantifying how a scRNA-seq feature selection or clustering method's runtime and peak memory grow with cell count — e.g. comparing a correlation-based method against correlation-free baselines at 10k/100k/1M-cell scales.

## When NOT to use

- The question concerns clustering accuracy (e.g. silhouette index, ARI against cell-type annotations), not computational scaling — use the accuracy benchmarking skill instead
- The dataset is small (e.g. the 10X pbmc_1k_v2 PBMC dataset, ~1k cells), where memory/time scaling cannot be meaningfully profiled across orders of magnitude
- Only one feature selection method is of interest and no correlation-free baselines will be run, since the benchmark's value is the comparative scaling contrast

## Inputs

- Cao et al. mouse organogenesis scRNA-seq count matrix from GEO accession GSE119945 (HDF5-backed, loaded with hdf5r)
- Random downsamplings of 10,000 and 100,000 cells
- Full million-plus-cell preprocessed Seurat object (QC-filtered, log-normalized)

## Outputs

- runtime_memory_benchmark.csv — per-method wall-clock runtime and peak memory at each cell-count scale
- Comparison table reproducing Supp. Fig. S4 (time and memory versus cell count for all eight feature selection methods)

## How to apply

Download the Cao et al. mouse organogenesis count matrix from GEO accession GSE119945, load it into a Seurat object via hdf5r, and perform quality control and log-normalization (R >= 3.5.0). Generate random 10,000-cell and 100,000-cell downsamplings and keep the full million-plus-cell dataset as the largest scale point. Run each feature selection method — DUBStepR, HVGDisp, HVGVST, trendVar, devianceFS, M3DropDANB, GiniClust, and HLG — in a fresh R session per run to isolate memory footprints. Record per-method wall-clock runtime and peak memory at each scale with R's system timing and memory profiling utilities, then assemble a time-vs-memory comparison table and verify the scaling pattern against Supp. Fig. S4.

## Related tools

- **DUBStepR** (Correlation-based feature selection method whose runtime and peak memory are profiled across the 10k/100k/1M-cell scale points) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Preprocessing (quality control, log-normalization) of the GSE119945 organogenesis counts and generation of downsampled Seurat objects at each scale) — https://satijalab.org/seurat/
- **hdf5r** (Loads the large GSE119945 count matrix into the Seurat object without exhausting memory)
- **R** (Execution environment (>= 3.5.0) providing system timing and memory profiling utilities to record runtime and peak memory per fresh session)
- **M3Drop** (Correlation-free baseline method (M3DropDANB) whose memory scaling is compared against DUBStepR)
- **GiniClust** (Baseline feature selection method that fails at 100k cells, a check that must be reproduced in the benchmark)

## Examples

```
if (!require(DUBStepR)) install.packages("DUBStepR")
```

## Evaluation signals

- DUBStepR's peak memory on the 1M-cell organogenesis dataset is ~6.4x its peak memory on the 100k downsampling (as quantified in the source task)
- DUBStepR shows no sharp increase in time or memory with scale, while the five correlation-free baselines exhibit markedly larger memory increases from 100k to 1M cells
- GiniClust and HLG fail (crash/error) at the 100,000-cell scale, reproducing Supp. Fig. S4 behavior
- runtime_memory_benchmark.csv contains a row for every method (8 methods) at every scale point (10k, 100k, 1M) with non-null runtime and peak-memory values
- Each method was run in a fresh R session so recorded peak memory is attributable to a single method, not carried over between runs

## Limitations

- GiniClust and HLG fail at 100k cells, so their larger-scale points cannot be measured and their scaling curves are incomplete
- Key quantitative results are deferred to supplementary material (Supp. Fig. S3, S4 and Supp. Note 1B), not the main text; the article itself reports no quantitative thresholds or effect sizes for the scalability claim
- Benchmark datasets in the paper are identified only by reference numbers (e.g. reference 28), so dataset provenance must be re-established from the source accession (GSE119945)

## Evidence

- [discussion] Scalability claim with supplementary figure pointer: "DUBStepR scales effectively to datasets of over a million cells without sharp increases in time or memory consumption (Supp. Fig. S4)."
- [abstract] Abstract-level scalability statement: "DUBStepR is scalable to over a million cells, and can be straightforwardly applied to other data types such as single-cell ATAC-seq."
- [methods] Required R version for reproducible runs: "DUBStepR requires your R version to be >= 3.5.0"
- [methods] Loading large count matrices into Seurat: "load the data in to your Seurat object using the **hdf5r** package"
- [methods] Installing the method under benchmark: "if (!require(DUBStepR))
  install.packages("DUBStepR")"
