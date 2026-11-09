---
name: computational-resource-profiling-runtime-memory
description: Use when benchmarking clustering methods on large-scale scRNA-seq data
  (e.g., tens of thousands of cells) and you must report not only NMI accuracy but
  also wall-clock time and peak memory to justify scalability claims.
when_to_use_negative:
- Dataset is small enough that runtime and memory differences between methods are
  negligible, so only accuracy (NMI/ARI) matters
- The goal is a single method's parameter tuning rather than a head-to-head scalability
  comparison across methods
- Input is not a large-scale scRNA-seq clustering problem (the paper's motivation
  is that existing algorithms' high computational complexity impedes extension to
  large-scale datasets)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: sscClust (SSCC)
  role: R package providing the SSCC framework (Spearman subsampling-clustering-classification)
    whose reduced subsample size (200 cells vs SC3's 5000) yields the 50-fold runtime
    and >33% memory savings being profiled
  repo: https://github.com/Japrin/sscClust
- name: SC3
  role: Baseline clustering method run with default parameters to measure the reference
    runtime, memory, and NMI against which SSCC acceleration is compared
- name: dropClust
  role: Third clustering method included in the same benchmark to extend the runtime/memory/NMI
    profiling beyond SC3 variants
- name: R
  role: Implementation environment for all methods; an R build linked to optimized
    BLAS (ATLAS, MKL, or Microsoft R Open) is recommended for large datasets
  repo: https://cran.r-project.org/doc/manuals/r-release/R-admin.html#BLAS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:153b21ef480409da20585585239c6d96ac9746fe0add457b8dec0bb6ff109821
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SC3
  - SSCC
  - dropClust
  - sscClust (SSCC)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# computational-resource-profiling-runtime-memory

## Summary

Profiles wall-clock runtime and peak memory usage of scRNA-seq clustering methods alongside clustering accuracy (NMI), so that accuracy gains can be weighed against computational cost. This skill operationalizes the SSCC paper's benchmark practice of comparing SC3+SSCC, SC3 default, and dropClust on PBMC 68k with recorded time, memory, and NMI per method.

## When to use

Use when benchmarking clustering methods on large-scale scRNA-seq data (e.g., tens of thousands of cells) and you must report not only NMI accuracy but also wall-clock time and peak memory to justify scalability claims.

## When NOT to use

- Dataset is small enough that runtime and memory differences between methods are negligible, so only accuracy (NMI/ARI) matters
- The goal is a single method's parameter tuning rather than a head-to-head scalability comparison across methods
- Input is not a large-scale scRNA-seq clustering problem (the paper's motivation is that existing algorithms' high computational complexity impedes extension to large-scale datasets)

## Inputs

- scRNA-seq count matrix (e.g., PBMC 68k dataset, 68,578 human blood cells) loadable in R via sscClust utilities
- Reference cell labels for NMI computation
- Clustering method configurations to compare (SC3 default, SC3+SSCC with 200-cell subsample, dropClust)

## Outputs

- CSV of per-method benchmark results (pbmc68k_benchmark_results.csv) with NMI, wall-clock runtime, and maximum memory per method
- Comparative resource ratios (e.g., fold speedup, percentage memory savings) between methods

## How to apply

Run each competing method (e.g., SC3 with its default subsampling-clustering-classification strategy using a 5000-cell subsample size, SC3 wrapped in the SSCC framework with a 200-cell subsample size, and dropClust) on the same dataset, such as the 68,578-cell PBMC 68k count matrix loaded in R via sscClust utilities. Instrument each run to record wall-clock runtime and peak memory usage during execution, not as post-hoc estimates. Compute normalized mutual information (NMI) of each method's clustering against reference cell labels to pair accuracy with resource cost. Write per-method NMI, runtime, and maximum memory to a structured output such as pbmc68k_benchmark_results.csv so acceleration and memory ratios (e.g., 50-fold speedup, memory savings >33%) can be computed directly. Use an R build linked to an optimized BLAS library (ATLAS, MKL, or Microsoft R Open) since the sscClust README warns computation becomes expensive on large datasets otherwise.

## Related tools

- **sscClust (SSCC)** (R package providing the SSCC framework (Spearman subsampling-clustering-classification) whose reduced subsample size (200 cells vs SC3's 5000) yields the 50-fold runtime and >33% memory savings being profiled) — https://github.com/Japrin/sscClust
- **SC3** (Baseline clustering method run with default parameters to measure the reference runtime, memory, and NMI against which SSCC acceleration is compared)
- **dropClust** (Third clustering method included in the same benchmark to extend the runtime/memory/NMI profiling beyond SC3 variants)
- **R** (Implementation environment for all methods; an R build linked to optimized BLAS (ATLAS, MKL, or Microsoft R Open) is recommended for large datasets) — https://cran.r-project.org/doc/manuals/r-release/R-admin.html#BLAS

## Examples

```
sce.all <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- pbmc68k_benchmark_results.csv contains one row per method with non-missing NMI, runtime, and maximum memory values
- SC3+SSCC runtime is ~50x faster than SC3 default (2.2 min vs ~110 min) on PBMC 68k, consistent with the paper's reported acceleration
- SC3+SSCC peak memory is at least one-third lower than SC3 default (3.7 GB, saving >33%, i.e., ~66% of SC3's usage)
- SC3+SSCC NMI is ~21% higher than SC3 default (0.59 vs 0.48) on the same reference labels
- All methods were run on the identical count matrix with the same reference labels so NMI and resource metrics are directly comparable

## Limitations

- Runtime and memory measurements are hardware- and BLAS-dependent; the sscClust README notes computation becomes expensive without an optimized BLAS library, so absolute values may not reproduce without a similar environment
- The subsampling strategy reduces computational complexity from O(n^2) to O(n) but introduces subsampling biases that can decrease clustering accuracy and robustness, so resource savings may come at an accuracy cost on some datasets
- No version numbers were stated for the compared tools (SC3, Seurat, dropClust) and no dataset accessions were provided in the article, complicating exact reproduction

## Evidence

- [abstract] 50-fold acceleration with 66% memory usage: "SSCC achieved 20% improvement for clustering accuracy and 50-fold acceleration, but only consumed 66% memory usage, compared to the widelyused software package SC3"
- [intro] reduce complexity O(n^2) to O(n): "Benchmarking on various scRNA-seq datasets demonstrates that compared to the current solutions, SSCC can reduce the computational complexity from O(n2) to O(n) while maintaining high clustering"
- [discussion] subsampling bias caveat: "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness, although it can reduce the computational complexity from O(n2) to O(n)"
- [readme] BLAS recommendation: "Calculation will become expensive when the dataset become large, so we stongly recommand use R which linked to optimized BLAS library, such as ATLAS, MKL etc."
