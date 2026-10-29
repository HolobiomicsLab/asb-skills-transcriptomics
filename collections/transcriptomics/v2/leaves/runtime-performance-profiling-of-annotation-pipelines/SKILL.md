---
name: runtime-performance-profiling-of-annotation-pipelines
description: Use when benchmarking how quickly automated cell-type annotation methods
  run on the same clustered scRNA-seq data, or when justifying a method choice by
  both speed and accuracy across multiple datasets.
when_to_use_negative:
- The annotation method cannot accept the shared marker inputs — e.g., scCATCH was
  excluded from simulated-data comparisons because its implementation does not allow
  custom markers
- Datasets have not been clustered with the same Seurat parameters, making runtime
  and accuracy comparisons across methods non-equivalent
- The goal is benchmarking reference-based supervised classifiers (e.g., CaSTLe, ACTINN),
  which require labeled training data rather than marker gene sets
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ScType
  role: Method under benchmark; fully-automated and ultra-fast cell-type identification
    run via sctype_score() on the scaled expression matrix
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: Seurat
  role: Seurat v4.0 used for identical preprocessing (LogNormalize, vst variable features,
    ScaleData, PCA, clustering) of every dataset before timing annotation methods
- name: scSorter
  role: Prior knowledge-based baseline method timed and accuracy-compared against
    ScType on the same clustered datasets
- name: SCINA
  role: Prior knowledge-based baseline method timed and accuracy-compared against
    ScType
- name: scCATCH
  role: Prior knowledge-based baseline method timed and accuracy-compared against
    ScType on clustered datasets (excluded from simulated-data comparisons)
- name: R
  role: Implementation language for running all methods and recording wall-clock runtimes
  repo: https://github.com/IanevskiAleksandr/sc-type
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:8f2ac9ff2cc03b2876370140b7cb23ad0b45b551a5de75782f28e107f3932316
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IanevskiAleksandr/sc-type
  tools:
  - R
  - SCINA
  - ScType
  - Seurat
  - dplyr
  - scCATCH
  - scSorter
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Runtime performance profiling of annotation pipelines

## Summary

Systematic wall-clock runtime benchmarking of cell-type annotation methods (ScType vs scSorter, SCINA, scCATCH) on identically clustered scRNA-seq datasets, demonstrating ScType's ultra-fast execution (>30x faster than scSorter) alongside its 98.6% annotation accuracy.

## When to use

Use when benchmarking how quickly automated cell-type annotation methods run on the same clustered scRNA-seq data, or when justifying a method choice by both speed and accuracy across multiple datasets.

## When NOT to use

- The annotation method cannot accept the shared marker inputs — e.g., scCATCH was excluded from simulated-data comparisons because its implementation does not allow custom markers
- Datasets have not been clustered with the same Seurat parameters, making runtime and accuracy comparisons across methods non-equivalent
- The goal is benchmarking reference-based supervised classifiers (e.g., CaSTLe, ACTINN), which require labeled training data rather than marker gene sets

## Inputs

- Seurat-processed scRNA-seq objects from six datasets: GSE124395 (human liver), GSE67835 (human brain), GSE85241 (human pancreas), GSE63269 (mouse lung), GSE63473 (mouse retina), and 10x PBMC3k filtered gene-barcode matrices
- Tissue-specific positive and negative marker gene sets prepared from ScTypeDB_full.xlsx via gene_sets_prepare()
- Reference cell-type labels per dataset for accuracy comparison

## Outputs

- Per-dataset, per-method wall-clock runtime measurements
- Cluster-level and per-cell annotation accuracy counts (e.g., 72/73 cell types correct, 98.6% accuracy)
- benchmark_results.csv comparison table of runtime and accuracy across ScType, scSorter, SCINA, and scCATCH

## How to apply

First process every dataset identically in Seurat v4.0 (LogNormalize with scale.factor=10000, 2000 variable features via vst, ScaleData, RunPCA, FindNeighbors/FindClusters at resolution=0.8 for PBMC3k) so that runtime differences reflect the annotation algorithms rather than preprocessing. Run the method under test (ScType's sctype_score with scaled=TRUE, gs=gs_positive, gs2=gs_negative from gene_sets_prepare on ScTypeDB_full.xlsx) and each baseline (scSorter, SCINA, scCATCH) on the same clustered object with the same marker inputs, excluding human-brain-derived markers when annotating brain clusters to avoid circularity. Record wall-clock runtime per method per dataset alongside the cluster annotations. Correctness is judged by comparing cluster labels to reference cell types (ScType: 72/73 cell types correct, >94% per-cell accuracy on every dataset) and by confirming the >30x runtime advantage over scSorter; write results to a comparison table such as benchmark_results.csv.

## Related tools

- **ScType** (Method under benchmark; fully-automated and ultra-fast cell-type identification run via sctype_score() on the scaled expression matrix) — https://github.com/IanevskiAleksandr/sc-type
- **Seurat** (Seurat v4.0 used for identical preprocessing (LogNormalize, vst variable features, ScaleData, PCA, clustering) of every dataset before timing annotation methods)
- **scSorter** (Prior knowledge-based baseline method timed and accuracy-compared against ScType on the same clustered datasets)
- **SCINA** (Prior knowledge-based baseline method timed and accuracy-compared against ScType)
- **scCATCH** (Prior knowledge-based baseline method timed and accuracy-compared against ScType on clustered datasets (excluded from simulated-data comparisons))
- **R** (Implementation language for running all methods and recording wall-clock runtimes) — https://github.com/IanevskiAleksandr/sc-type

## Examples

```
gs_list <- gene_sets_prepare("ScTypeDB_full.xlsx", "Liver"); es.max <- sctype_score(scRNAseqData = es, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)
```

## Evaluation signals

- ScType correctly annotates 72 of 73 cell types (98.6% accuracy) across the six human and mouse datasets
- ScType annotates >94% of cells correctly on every dataset
- ScType wall-clock runtime is >30x faster than scSorter on the same clustered inputs
- Identical Seurat preprocessing parameters (LogNormalize scale.factor=10000, 2000 vst features, resolution=0.8 for PBMC3k) are used for all methods so runtime comparisons are fair
- Comparison table (benchmark_results.csv) contains per-dataset runtime and correctly-assigned cell-type counts for all four methods

## Limitations

- Human-brain-derived markers must be excluded when annotating brain clusters in the comparative study to avoid bias from marker circularity
- scCATCH cannot use custom markers and was therefore not used in the simulated-data comparisons
- Accuracy claims depend on marker database coverage; ScType's z-score transformation may be suboptimal for single-cell transcriptomic data, and negative markers remain relatively few compared to positive markers

## Evidence

- [discussion] ScType outperformed baselines in accuracy and speed: "Our unsupervised approach outperformed the prior knowledge-based methods scSorter30, SCINA33, and scCATCH34"
- [discussion] Exclusion of brain markers to avoid bias: "we excluded the markers identified from the human brain data while annotating the cell cluster in the comparative study"
- [discussion] Identical Seurat preprocessing for benchmarking: "We utilized Seurat v4.042 for data processing and normalization."
- [readme] ScType core scoring invocation used in the benchmark: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
- [discussion] Five GEO benchmark datasets: "Five datasets were downloaded from Gene Express Omnibus (GEO): Human Liver (GSE124395)"
