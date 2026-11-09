---
name: highly-variable-gene-preprocessing
description: Use when preparing raw multi-batch single-cell RNA-seq count matrices
  (e.g., the two-batch Zilionis mouse lung data with B-cell and T-cell clusters) as
  input to batch-correction/integration methods that require a reduced gene set.
when_to_use_negative:
- The input is already a reduced/normalized feature matrix from a prior HVG selection
  step — re-filtering would discard signal needed for batch-correction benchmarks
- The dataset is small enough that JIVE's original bulk implementation runs feasibly
  without gene filtering, and full-gene-resolution results are desired
- The analysis goal does not involve batch effects or multi-source integration (scJIVE
  is designed to extract shared biological patterns while adjusting for batch effect)
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scJIVE (jive function)
  role: Downstream consumer of the HVG-filtered matrix; the jive() function estimates
    the joint/individual decomposition used for batch correction
  repo: https://github.com/oconnell-statistics-lab/scJIVE
- name: R
  role: Environment for loading data (scRNAseq package), performing HVG selection,
    and running the integration and evaluation pipeline
- name: Seurat v5
  role: Benchmark integration method run on the same HVG-filtered matrix for comparison
    against JIVE and Harmony
- name: Harmony
  role: Benchmark integration method run on the same HVG-filtered matrix for comparison
    against JIVE and Seurat v5
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:f734e033ecb472a30375f7ef96587ea0a9badfa69097bdbeca00bbf9dac6708c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oconnell-statistics-lab/scJIVE
  tools:
  - Harmony
  - R
  - Seurat v5
  - scJIVE (jive function)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# highly-variable-gene-preprocessing

## Summary

Selects the top highly variable genes from multi-batch scRNA-seq data to produce a compact expression matrix (e.g., 2000 genes x 2533 cells in the Zilionis mouse lung benchmark) with batch labels and cell-type annotations retained, so that batch-correction methods like scJIVE, Seurat v5, and Harmony can run efficiently and comparably.

## When to use

Use when preparing raw multi-batch single-cell RNA-seq count matrices (e.g., the two-batch Zilionis mouse lung data with B-cell and T-cell clusters) as input to batch-correction/integration methods that require a reduced gene set.

## When NOT to use

- The input is already a reduced/normalized feature matrix from a prior HVG selection step — re-filtering would discard signal needed for batch-correction benchmarks
- The dataset is small enough that JIVE's original bulk implementation runs feasibly without gene filtering, and full-gene-resolution results are desired
- The analysis goal does not involve batch effects or multi-source integration (scJIVE is designed to extract shared biological patterns while adjusting for batch effect)

## Inputs

- Raw scRNA-seq expression matrix (genes x cells) from multi-batch data, e.g., the Zilionis mouse lung dataset loaded via the scRNAseq R package
- Per-cell batch labels (e.g., two batches with B-cell and T-cell clusters)
- Per-cell cell-type annotations

## Outputs

- Filtered expression matrix of the top 2000 highly variable genes x 2533 cells
- Batch labels and cell-type annotations aligned to the retained cells
- Preprocessed input ready for JIVE (scJIVE), Seurat v5, Harmony, and raw-data baseline comparisons

## How to apply

Load the raw scRNA-seq dataset (e.g., via the scRNAseq R package for the Zilionis mouse lung data) and subset to the batches of interest — here the two batches with B-cell and T-cell clusters. Rank genes by variability and keep only the top 2000 variable genes, yielding a 2000 genes x 2533 cells expression matrix; crucially, carry the batch labels and cell-type annotations alongside the filtered matrix so downstream methods can use them. This dimensionality reduction is essential because JIVE's original bulk-oriented implementation is computationally infeasible for large-scale single-cell data, motivating the efficient scJIVE implementation. Feed the resulting matrix (with annotations) into the integration methods under comparison — scJIVE's jive() function, Seurat v5, and Harmony — plus keep the raw uncorrected data as a baseline for kBET, ASW, and LISI evaluation.

## Related tools

- **scJIVE (jive function)** (Downstream consumer of the HVG-filtered matrix; the jive() function estimates the joint/individual decomposition used for batch correction) — https://github.com/oconnell-statistics-lab/scJIVE
- **R** (Environment for loading data (scRNAseq package), performing HVG selection, and running the integration and evaluation pipeline)
- **Seurat v5** (Benchmark integration method run on the same HVG-filtered matrix for comparison against JIVE and Harmony)
- **Harmony** (Benchmark integration method run on the same HVG-filtered matrix for comparison against JIVE and Seurat v5)

## Examples

```
devtools::install_github("oconnell-statistics-lab/scJIVE"); jive.out <- jive(hvg_matrix, batch_labels)
```

## Evaluation signals

- The filtered matrix has the expected dimensions: 2000 genes x 2533 cells after selecting the top 2000 variable genes
- Batch labels and cell-type annotations are present and aligned one-to-one with the retained cells
- Downstream kBET acceptance rates for raw and integrated embeddings all fall within 0-1, and the metric CSV contains kBET, ASW, and LISI fields
- Raw (uncorrected) data is retained as a baseline so LISI cell-type values of corrected methods (e.g., JIVE and Seurat v5, which were considerably worse than raw on Zilionis mouse lung data) can be judged against it
- The same filtered matrix is passed to all methods (JIVE, Seurat v5, Harmony, raw baseline) so metric comparisons are apples-to-apples

## Limitations

- JIVE's original implementation was designed for bulk sequencing data and is computationally infeasible for large-scale single-cell datasets, so gene filtering alone may not suffice for very large datasets — the efficient scJIVE implementation is required
- JIVE performed best in scenarios where batch sizes are balanced; unbalanced batch sizes may degrade results even with proper HVG preprocessing
- Preprocessing quality does not guarantee integration quality: on the Zilionis mouse lung data, JIVE and Seurat v5 failed to preserve cell type distinctions (LISI cell-type values worse than raw data) despite the same preprocessing

## Evidence

- [other] Preprocess by selecting the top 2000 variable genes, yielding a 2000 genes x 2533 cells expression matrix with batch labels and cell-type annotations.: [evidence span withheld — anchor status: not_found]
- [intro] scJIVE can be used to extract shared biological patterns from multi-source sequencing data while adjusting for individual non-biological variations (i.e., batch effect).: "scJIVE can be used to extract shared biological patterns from multi-source sequencing data while adjusting for individual non-biological variations (i.e., batch effect)."
- [abstract] However, its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets.: "However, its current implementation is originally designed for bulk sequencing data, making it computationally infeasible for large-scale single-cell sequencing datasets."
- [methods] The default function to estimate the JIVE decomposition is `jive`: "The default function to estimate the JIVE decomposition is `jive`"
