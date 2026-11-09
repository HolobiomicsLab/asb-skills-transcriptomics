---
name: execution-time-memory-profiling
description: 'Use when scaling claims about a feature selection or clustering method
  must be quantified: run the method on downsamplings (10k, 100k cells) and the full
  dataset (1M+ cells) and record per-method runtime and peak memory.'
when_to_use_negative:
- The goal is clustering accuracy comparison (silhouette index, number of feature
  genes) rather than computational scalability profiling
- The input dataset is small enough that runtime and memory differences between methods
  are not informative
- The method under test is a correlation-free baseline whose memory behavior is already
  established and does not need re-benchmarking
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DUBStepR
  role: Primary correlation-based feature selection method whose runtime and peak
    memory are profiled across dataset scales
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Preprocesses the count matrix (quality control, log-normalization) and holds
    the data object before feature selection runs
- name: hdf5r
  role: Loads the large HDF5-format mouse organogenesis count matrix into the Seurat
    object
- name: M3Drop
  role: Baseline feature selection method (M3DropDANB) benchmarked for runtime and
    memory at each scale
- name: GiniClust
  role: Baseline feature selection method benchmarked; known to fail at 100k cells
- name: R
  role: Execution environment (>= 3.5.0); each method run in a fresh R session with
    system timing and memory profiling utilities
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:c041d2738c5cbb9bdde796bc45c56bd4731b70f269cf4a8ce00282f5e4d3bacf
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# execution-time-memory-profiling

## Summary

Benchmark wall-clock runtime and peak memory consumption of scRNA-seq feature selection methods (DUBStepR, HVGDisp, HVGVST, trendVar, devianceFS, M3DropDANB, GiniClust, HLG) across dataset scales on the mouse organogenesis scRNA-seq data, to quantify scalability claims such as DUBStepR's 6.4x memory increase from 100k to 1M cells.

## When to use

Use when scaling claims about a feature selection or clustering method must be quantified: run the method on downsamplings (10k, 100k cells) and the full dataset (1M+ cells) and record per-method runtime and peak memory.

## When NOT to use

- The goal is clustering accuracy comparison (silhouette index, number of feature genes) rather than computational scalability profiling
- The input dataset is small enough that runtime and memory differences between methods are not informative
- The method under test is a correlation-free baseline whose memory behavior is already established and does not need re-benchmarking

## Inputs

- Cao et al. mouse organogenesis scRNA-seq count matrix (GEO accession GSE119945, HDF5/10X format)
- Random downsamplings of the dataset at 10,000 and 100,000 cells plus the full 1M+ cell dataset
- Feature selection methods to compare: DUBStepR, HVGDisp, HVGVST, trendVar, devianceFS, M3DropDANB, GiniClust, HLG

## Outputs

- runtime_memory_benchmark.csv — comparison table of wall-clock runtime and peak memory versus cell count for each method
- Reproduction of Supp. Fig. S4 (scalability profile across dataset sizes)

## How to apply

Download the Cao et al. mouse organogenesis scRNA-seq count matrix (GEO accession GSE119945), apply quality control and log-normalization with Seurat (R >= 3.5.0), and load the data into the Seurat object using the hdf5r package. Generate random 10,000-cell and 100,000-cell downsamplings and retain the full million-plus-cell dataset as the largest scale point. Run each feature selection method — DUBStepR, HVGDisp, HVGVST, trendVar, devianceFS, M3DropDANB, GiniClust, and HLG — on every dataset size in a fresh R session, recording wall-clock runtime and peak memory via R's system timing and memory profiling utilities. Assemble a time-vs-cell-count and memory-vs-cell-count comparison table, verifying that DUBStepR's memory increase from 100k to full scale (6.4x) is markedly smaller than that of the five correlation-free baselines, and that GiniClust and HLG fail at 100k cells, reproducing Supp. Fig. S4.

## Related tools

- **DUBStepR** (Primary correlation-based feature selection method whose runtime and peak memory are profiled across dataset scales) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Preprocesses the count matrix (quality control, log-normalization) and holds the data object before feature selection runs)
- **hdf5r** (Loads the large HDF5-format mouse organogenesis count matrix into the Seurat object)
- **M3Drop** (Baseline feature selection method (M3DropDANB) benchmarked for runtime and memory at each scale)
- **GiniClust** (Baseline feature selection method benchmarked; known to fail at 100k cells)
- **R** (Execution environment (>= 3.5.0); each method run in a fresh R session with system timing and memory profiling utilities)

## Examples

```
Rscript -e 'out <- DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)'
```

## Evaluation signals

- runtime_memory_benchmark.csv contains one runtime and peak-memory row per method (8 methods) per scale point (10k, 100k, full 1M+ cells)
- DUBStepR's memory consumption at the full 1M-cell scale is ~6.4x its memory at the 100k-cell scale — markedly smaller than the increase of the five correlation-free baselines
- GiniClust and HLG fail (crash or do not complete) at the 100,000-cell downsampling, matching Supp. Fig. S4
- Every method run was executed in a fresh R session so that recorded peak memory is not contaminated by prior runs
- The scaling profile shows no sharp increase in DUBStepR time or memory beyond a million cells, consistent with the paper's scalability claim

## Limitations

- GiniClust and HLG cannot be profiled beyond ~100k cells because they fail at that scale
- Key scalability results are deferred to supplementary material (Supp. Fig. S4, Supp. Note 1B) and the main text reports no quantitative thresholds or effect sizes beyond the measured 6.4x memory increase
- DUBStepR requires R >= 3.5.0 and its recommended Seurat/hdf5r loading path; profiling on other toolchains may not be comparable

## Evidence

- [discussion] Scalability claim on million-cell data: "DUBStepR scales effectively to datasets of over a million cells without sharp increases in time or memory consumption (Supp. Fig. S4)."
- [abstract] Scalability and general applicability: "DUBStepR is scalable to over a million cells, and can be straightforwardly applied to other data types such as single-cell ATAC-seq."
- [methods] Minimum R version for the profiling environment: "DUBStepR requires your R version to be >= 3.5.0"
- [methods] Recommended data loading path used before profiling: "load the data in to your Seurat object using the **hdf5r** package"
- [readme] Role of feature selection in the pipeline being scaled: "Feature selection, i.e. determining the optimal subset of genes to cluster cells into cell types, is a critical step in the unsupervised clustering of scRNA-seq data."
