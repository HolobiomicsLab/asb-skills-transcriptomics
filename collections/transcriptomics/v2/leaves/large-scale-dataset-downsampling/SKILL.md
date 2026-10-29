---
name: large-scale-dataset-downsampling
description: Use when benchmarking a scRNA-seq feature selection or clustering method's
  runtime and memory footprint across dataset sizes (e.g., 10k → 100k → 1M cells)
  on a single large source dataset.
when_to_use_negative:
- The dataset is already small (e.g., the 10X Genomics pbmc_1k_v2 PBMC dataset used
  for the clustering-accuracy benchmarks) — scaling behavior cannot be assessed without
  a large source dataset
- The analysis goal is clustering accuracy or silhouette-index comparison rather than
  runtime/memory scalability
- Only a downsampled subset exists and the full-scale point (over a million cells)
  cannot be run, invalidating the scaling comparison
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DUBStepR
  role: Feature selection method whose memory/runtime scaling from 100k to 1M cells
    is being quantified
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Preprocessing (quality control, log-normalization) and loading of the mouse
    organogenesis dataset at each scale
- name: hdf5r
  role: Reading the large GSE119945 count matrix into the Seurat object
- name: M3Drop
  role: Baseline method (M3DropDANB) run at each scale for time/memory comparison
- name: GiniClust
  role: Baseline method known to fail at 100k cells; recorded as a failure point in
    the benchmark
- name: R
  role: Runtime environment (>= 3.5.0); fresh sessions per method with system timing
    and memory profiling
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:f39a985e516b5158384d3778b96fe6b601719753a33c96e5178e7102bc20631a
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# large-scale-dataset-downsampling

## Summary

Generates multi-scale (10k, 100k, ~1M cell) downsamplings of the Cao et al. mouse organogenesis scRNA-seq dataset (GEO GSE119945) to profile how a feature selection method's runtime and peak memory scale with cell count, as in the DUBStepR scalability benchmark.

## When to use

Use when benchmarking a scRNA-seq feature selection or clustering method's runtime and memory footprint across dataset sizes (e.g., 10k → 100k → 1M cells) on a single large source dataset.

## When NOT to use

- The dataset is already small (e.g., the 10X Genomics pbmc_1k_v2 PBMC dataset used for the clustering-accuracy benchmarks) — scaling behavior cannot be assessed without a large source dataset
- The analysis goal is clustering accuracy or silhouette-index comparison rather than runtime/memory scalability
- Only a downsampled subset exists and the full-scale point (over a million cells) cannot be run, invalidating the scaling comparison

## Inputs

- Cao et al. mouse organogenesis scRNA-seq count matrix (GEO GSE119945, ~1M+ cells, HDF5-backed)
- Seurat object after quality control and log-normalization
- Random downsamplings of 10,000 and 100,000 cells retaining the full gene set

## Outputs

- runtime_memory_benchmark.csv comparing per-method wall-clock runtime and peak memory at each cell-count scale
- Comparison table reproducing Supp. Fig. S4 (memory ratio, e.g., DUBStepR 6.4x from 100k to 1M cells)
- Record of methods that fail at scale (GiniClust, HLG fail at 100k cells)

## How to apply

Download the Cao et al. mouse organogenesis count matrix from GEO accession GSE119945 and preprocess it in R (>= 3.5.0) with Seurat: quality control, log-normalization, and loading via the hdf5r package. Create random 10,000-cell and 100,000-cell downsamplings while retaining the full million-plus-cell dataset as the largest scale point, so that all points share the same gene universe and preprocessing. Run each method (e.g., DUBStepR, HVGDisp, HVGVST, trendVar, devianceFS, M3DropDANB, GiniClust, HLG) on every size in a fresh R session and record wall-clock runtime and peak memory with R's system timing and memory profiling utilities. Assemble a time-vs-memory-vs-cell-count table, then check that the target method's memory increase from 100k to full scale is markedly smaller than that of correlation-free baselines, and that methods known to fail (GiniClust, HLG) crash at 100k cells, reproducing Supp. Fig. S4.

## Related tools

- **DUBStepR** (Feature selection method whose memory/runtime scaling from 100k to 1M cells is being quantified) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Preprocessing (quality control, log-normalization) and loading of the mouse organogenesis dataset at each scale)
- **hdf5r** (Reading the large GSE119945 count matrix into the Seurat object)
- **M3Drop** (Baseline method (M3DropDANB) run at each scale for time/memory comparison)
- **GiniClust** (Baseline method known to fail at 100k cells; recorded as a failure point in the benchmark)
- **R** (Runtime environment (>= 3.5.0); fresh sessions per method with system timing and memory profiling)

## Examples

```
Rscript -e 'library(Seurat); library(hdf5r); obj <- CreateSeuratObject(Read10X_h5("GSE119945.h5")); obj <- logNormalize(GetAssayData(obj)); DUBStepR(input.data=obj, min.cells=3, species="mouse", optimise.features=TRUE)'
```

## Evaluation signals

- The 6.4x memory ratio for DUBStepR between the 100k-cell and ~1M-cell runs is reproduced, with the increase markedly smaller than that of the five correlation-free baselines
- GiniClust and HLG fail (crash or run out of memory) at the 100,000-cell scale, matching Supp. Fig. S4
- Downsampling invariants: 10k and 100k subsets are random cell subsamples retaining the full gene universe and identical QC/normalization steps
- Each method is run in a fresh R session and peak memory plus wall-clock runtime are recorded for every (method, scale) pair before writing runtime_memory_benchmark.csv

## Limitations

- No quantitative thresholds or effect sizes for scalability claims appear in the main text; the time/memory curves are deferred to supplementary Supp. Fig. S4 and Supp. Note 1B
- GiniClust and HLG cannot be profiled at large scales because they fail at 100k cells, so their full scaling profile is not measurable
- Benchmark dataset collection in the paper is identified only by reference number 28, not named or accessioned; scalability numbers depend on the specific hardware and R version (>= 3.5.0)

## Evidence

- [abstract] Scalability to a million cells: "DUBStepR is scalable to over a million cells, and can be straightforwardly applied to other data types such as single-cell ATAC-seq."
- [discussion] Supp. Fig. S4 scalability result: "DUBStepR scales effectively to datasets of over a million cells without sharp increases in time or memory consumption (Supp. Fig. S4)."
- [methods] R version requirement for benchmarking: "DUBStepR requires your R version to be >= 3.5.0"
- [methods] Loading large data into Seurat: "load the data in to your Seurat object using the **hdf5r** package"
- [methods] DUBStepR function signature: "`DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)`"
