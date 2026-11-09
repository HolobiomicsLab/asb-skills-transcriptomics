---
name: scrnaseq-count-matrix-preprocessing
description: Use when a raw scRNA-seq count matrix is sparse and dropout-prone and
  must be filtered, HVG-selected, and normalized to [0,1] before binarization or landmark
  construction.
when_to_use_negative:
- Input is already a filtered/normalized feature or embedding table rather than a
  raw count matrix — re-filtering would drop informative genes and the [0,1] normalization
  assumption no longer holds
- The analysis requires gene-level information beyond the 2000 retained HVGs — the
  article notes that retaining only 2000 HVGs excluded some informative genes
- The downstream tool requires raw integer counts (e.g., NB-model-based methods) rather
  than [0,1]-normalized continuous values
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Provides the pre.R R script and the main binary that consumes the pre-processed
    cells-x-genes matrix
  repo: https://github.com/LD-Lyndee/scQA
- name: Seurat
  role: Source of the highly-variable-gene selection method (references [11], [25])
    used to retain the 2000 HVGs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:283ecb4f3b2d5be85df6d1b405f1d1464d7285745d1a441977552c91a3adafc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - Seurat
  - scQA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# scrnaseq-count-matrix-preprocessing

## Summary

Preprocess raw scRNA-seq count matrices (e.g., Xin GSE81608) as done in scQA: filter genes by expression-frequency extremes, retain 2000 highly variable genes, log-transform, and normalize entries to [0,1] so downstream landmark constructors operate on a dense, bounded matrix.

## When to use

Use when a raw scRNA-seq count matrix is sparse and dropout-prone and must be filtered, HVG-selected, and normalized to [0,1] before binarization or landmark construction.

## When NOT to use

- Input is already a filtered/normalized feature or embedding table rather than a raw count matrix — re-filtering would drop informative genes and the [0,1] normalization assumption no longer holds
- The analysis requires gene-level information beyond the 2000 retained HVGs — the article notes that retaining only 2000 HVGs excluded some informative genes
- The downstream tool requires raw integer counts (e.g., NB-model-based methods) rather than [0,1]-normalized continuous values

## Inputs

- Raw scRNA-seq gene expression count matrix (cells x genes), e.g., Xin GSE81608 human pancreas SMARTer data in CSV/TXT form
- R environment for running the pre.R pre-processing script

## Outputs

- Pre-processed matrix of cells x 2000 highly variable genes, log-transformed and normalized to [0,1], saved as a headerless cells-x-genes text file (e.g., xin_matrix.txt) ready for scQA
- Binarized gene vectors feeding LC1 (0 if expression is zero, 1 otherwise)

## How to apply

Start from the raw gene expression matrix (e.g., Xin GSE81608, 149 cells x 23,985 genes, SMARTer) and filter out genes expressed in fewer than 2.5% of cells or more than 97.5% of cells to address sparsity. Retain the 2000 most highly variable genes using the Seurat-style method referenced by the article's references [11], [25]. Log-transform the data and normalize so every entry falls in the range [0,1]. Perform these steps in R (script provided as pre.R in the scQA repository), and write out a plain matrix of cells x genes without row or column names so the scQA binary can consume it. This bounded, HVG-restricted matrix is the required input for LC1, which binarizes entries (0 if expression is zero, 1 otherwise) to build qualitative landmarks.

## Related tools

- **scQA** (Provides the pre.R R script and the main binary that consumes the pre-processed cells-x-genes matrix) — https://github.com/LD-Lyndee/scQA
- **Seurat** (Source of the highly-variable-gene selection method (references [11], [25]) used to retain the 2000 HVGs)

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git; Rscript pre.R  # produces xin_matrix.txt; ./main xin_matrix.txt o
```

## Evaluation signals

- Gene dimension after filtering equals 2000 highly variable genes, and each retained gene is expressed in between 2.5% and 97.5% of cells
- All matrix entries lie within [0,1] after log-transformation and normalization
- Output file is a headerless cells-x-genes matrix (no row or column names), as required by the scQA input format
- Pre-processing completes via the provided pre.R R script without dropping the expected cell count (e.g., 149 cells for Xin GSE81608)
- Downstream LC1 binarization yields binary vectors (0/1) and gene sorting by decreasing zero counts proceeds on exactly the pre-processed matrix

## Limitations

- Retaining only 2000 HVGs led to the exclusion of some informative genes
- The specific version of R used for pre-processing is not stated
- The source code of the cited HVG method (references [11], [25]) is not explicitly named in the article

## Evidence

- [methods] To address sparsity in raw scRNA-seq data, we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells: "To address sparsity in raw scRNA-seq data, we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells"
- [methods] All data are log-transformed and normalized to ensure that each entry falls in the range [0,1].: "All data are log-transformed and normalized to ensure that each entry falls in the range [0,1]."
- [results] the retention of only 2000 HVGs for subsequent clustering analysis led to the exclusion of some informative genes: "the retention of only 2000 HVGs for subsequent clustering analysis led to the exclusion of some informative genes"
- [readme] The input file must be in the form of a matrix of cells * genes, without names of rows and columns.: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
- [readme] Pre-processing with R provided as pre.R: "Pre-processing with R provided as pre.R"
