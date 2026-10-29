---
name: benchmark-comparison-of-cell-annotation-methods
description: Use when you need to quantitatively compare automated cell-type annotation
  methods on the same clustered scRNA-seq datasets, with controlled marker inputs,
  accuracy counts, and runtime measurements.
when_to_use_negative:
- When the cell-type labels being evaluated were themselves derived from the annotation
  method's marker database for that tissue (e.g., brain markers derived from the brain
  test dataset), since this biases the comparison — the protocol explicitly excludes
  such self-derived markers
- When scCATCH must consume custom user-supplied marker sets in the simulated-data
  comparison, since its implementation does not allow custom markers
- When the goal is tumor/malignant-cell discrimination rather than healthy tissue
  annotation — ScType's SNV-calling capability via vartrix is a separate capability
  outside this benchmark
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ScType
  role: 'Method under test: fully-automated, unsupervised marker-based cell-type annotation
    using positive and negative marker specificity scores'
  repo: https://github.com/IanevskiAleksandr/sc-type
- name: Seurat
  role: 'Preprocessing backbone: LogNormalize, vst variable feature selection, scaling,
    PCA, and Louvain clustering shared by all compared methods'
- name: scSorter
  role: Prior-knowledge-based baseline method benchmarked on identical clustered datasets
    and marker inputs
- name: SCINA
  role: Prior-knowledge-based baseline method benchmarked on identical clustered datasets
    and marker inputs
- name: scCATCH
  role: Prior-knowledge-based baseline method benchmarked on identical clustered datasets
    and marker inputs (excluded from simulated-data comparison due to no custom-marker
    support)
- name: R
  role: Implementation language for all benchmark scripts, including runtime recording
    and results aggregation
- name: dplyr
  role: Data manipulation for assembling the comparison table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28803-w
    title: Fully-automated and ultra-fast cell-type identification using specific
      marker combinations from single-cell transcriptomic data
  source_content_hash: sha256:606e3e9c7c1c60f66e5bce11725bb78cbc5ee9ab82aa71aed0d1ddc1c86581e6
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# benchmark-comparison-of-cell-annotation-methods

## Summary

A systematic benchmarking protocol comparing the fully-automated marker-based method ScType against prior-knowledge methods scSorter, SCINA, and scCATCH for cell-type annotation across six human and mouse scRNA-seq datasets, recording both annotation accuracy and wall-clock runtime.

## When to use

Use when you need to quantitatively compare automated cell-type annotation methods on the same clustered scRNA-seq datasets, with controlled marker inputs, accuracy counts, and runtime measurements.

## When NOT to use

- When the cell-type labels being evaluated were themselves derived from the annotation method's marker database for that tissue (e.g., brain markers derived from the brain test dataset), since this biases the comparison — the protocol explicitly excludes such self-derived markers
- When scCATCH must consume custom user-supplied marker sets in the simulated-data comparison, since its implementation does not allow custom markers
- When the goal is tumor/malignant-cell discrimination rather than healthy tissue annotation — ScType's SNV-calling capability via vartrix is a separate capability outside this benchmark

## Inputs

- GEO scRNA-seq count matrices: GSE124395 (human liver), GSE67835 (human brain), GSE85241 (human pancreas), GSE63269 (mouse lung), GSE63473 (mouse retina)
- 10x Genomics PBMC3k filtered gene-barcode matrices (pbmc3k_filtered_gene_bc_matrices.tar.gz)
- ScTypeDB_full.xlsx tissue-specific marker database (positive and negative markers)
- Reference cell-type labels per dataset for scoring annotations

## Outputs

- Per-cluster cell-type annotations from ScType, scSorter, SCINA, and scCATCH
- benchmark_results.csv comparison table (correct cell-type counts, per-cell accuracy per dataset, runtime per method)
- Aggregate metrics: 72/73 cell types correct (98.6%) for ScType, >94% per-cell accuracy per dataset, >30x speedup vs scSorter

## How to apply

Process each dataset (GSE124395 human liver, GSE67835 human brain, GSE85241 human pancreas, GSE63269 mouse lung, GSE63473 mouse retina, and 10x PBMC3k) in Seurat v4.0 with LogNormalize (scale.factor=10000), 2000 vst variable features, ScaleData, RunPCA, then FindNeighbors/FindClusters (resolution=0.8 for PBMC3k). Prepare tissue-specific positive and negative marker gene sets with gene_sets_prepare() from ScTypeDB_full.xlsx and run sctype_score() on the scaled matrix (scaled=TRUE, gs=gs_positive, gs2=gs_negative), labeling clusters scoring < ncells/4 as 'Unknown'. Run scSorter, SCINA, and scCATCH on the same clustered datasets with identical marker inputs; critically, exclude markers derived from the human brain dataset itself when annotating brain clusters to prevent circularity. Record wall-clock runtime for each method, then count correctly assigned cell types and per-cell annotation accuracy per dataset against reference labels.

## Related tools

- **ScType** (Method under test: fully-automated, unsupervised marker-based cell-type annotation using positive and negative marker specificity scores) — https://github.com/IanevskiAleksandr/sc-type
- **Seurat** (Preprocessing backbone: LogNormalize, vst variable feature selection, scaling, PCA, and Louvain clustering shared by all compared methods)
- **scSorter** (Prior-knowledge-based baseline method benchmarked on identical clustered datasets and marker inputs)
- **SCINA** (Prior-knowledge-based baseline method benchmarked on identical clustered datasets and marker inputs)
- **scCATCH** (Prior-knowledge-based baseline method benchmarked on identical clustered datasets and marker inputs (excluded from simulated-data comparison due to no custom-marker support))
- **R** (Implementation language for all benchmark scripts, including runtime recording and results aggregation)
- **dplyr** (Data manipulation for assembling the comparison table)

## Examples

```
source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/gene_sets_prepare.R"); source("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/R/sctype_score_.R"); gs <- gene_sets_prepare("ScTypeDB_full.xlsx", "Liver"); es.max <- sctype_score(scRNAseqData = expr, scaled = TRUE, gs = gs$gs_positive, gs2 = gs$gs_negative)
```

## Evaluation signals

- ScType correctly annotates 72 of 73 cell types (98.6%) across the six datasets
- Per-cell annotation accuracy exceeds 94% on every dataset
- ScType wall-clock runtime is >30x faster than scSorter on the same inputs
- 8 originally incorrectly or non-specifically annotated cell types are correctly reannotated
- Comparison table is written to benchmark_results.csv with per-method counts and runtimes

## Limitations

- CellMarker-sourced markers did not list source publications, so marker overlap with benchmark datasets could not be confirmed (reproducibility gap)
- Negative markers remain relatively few compared to positive markers, potentially limiting annotation power
- z-score transformation of expression may be suboptimal for single-cell transcriptomic data, and drop-out events are not corrected in the current version
- scCATCH could not be included in the simulated-data comparison because it cannot use custom markers

## Evidence

- [discussion] ScType outperformed prior-knowledge baselines on accuracy: "Our unsupervised approach outperformed the prior knowledge-based methods scSorter30, SCINA33, and scCATCH34"
- [discussion] Brain markers excluded from brain annotation to avoid bias: "we excluded the markers identified from the human brain data while annotating the cell cluster in the comparative study"
- [discussion] scCATCH excluded from simulated-data comparison: "the scCATCH method was not used in these comparisons since its implementation does not allow for the use custom markers"
- [discussion] Benchmark datasets enumerated: "Five datasets were downloaded from Gene Express Omnibus (GEO): Human Liver (GSE124395)"
- [readme] ScType core scoring call with positive and negative gene sets: "es.max <- sctype_score(scRNAseqData = scRNAseqData, scaled = TRUE, gs = gs_list$gs_positive, gs2 = gs_list$gs_negative)"
- [methods] Marker gene set preparation from tissue-specific DB: "gs_list <- gene_sets_prepare("https://raw.githubusercontent.com/IanevskiAleksandr/sc-type/master/ScTypeDB_short.xlsx", "Immune system")"
