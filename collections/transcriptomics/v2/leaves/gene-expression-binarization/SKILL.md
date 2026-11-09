---
name: gene-expression-binarization
description: Use when preprocessing sparse single-cell RNA-seq matrices (e.g. Xin
  GSE81608) to build a binary low-dimensional representation of cells, where dropouts
  must be handled as signals rather than imputed away.
when_to_use_negative:
- Input is already a binary or imputed expression matrix — binarization and dropout-strengthening
  are designed for sparse, dropout-prone count data
- The analysis goal is the quantitative (LC2) representation based on quasi-trend-preserved
  expression patterns, which uses the raw normalized values rather than binary encoding
- Retaining only 2000 HVGs would discard known informative genes for the study, since
  the article notes this exclusion as a limitation
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Implements the LC1 binarization step and downstream gene amalgamation into
    binary qualitative landmarks
  repo: https://github.com/LD-Lyndee/scQA
- name: Seurat
  role: Style of highly variable gene selection (method of references [11],[25]) used
    during preprocessing before binarization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:b75cc40c2a738c8ba7394c97c0447388d72b52d9bd1079fdaf7506d97919baa1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - Seurat
  - scQA
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gene-expression-binarization

## Summary

Binarizes dropout-prone scRNA-seq expression values (0 if zero, 1 otherwise) as the first step of scQA's qualitative landmark constructor (LC1), treating dropouts as weak signals that are later strengthened by clustering genes with similar binary expression patterns.

## When to use

Use when preprocessing sparse single-cell RNA-seq matrices (e.g. Xin GSE81608) to build a binary low-dimensional representation of cells, where dropouts must be handled as signals rather than imputed away.

## When NOT to use

- Input is already a binary or imputed expression matrix — binarization and dropout-strengthening are designed for sparse, dropout-prone count data
- The analysis goal is the quantitative (LC2) representation based on quasi-trend-preserved expression patterns, which uses the raw normalized values rather than binary encoding
- Retaining only 2000 HVGs would discard known informative genes for the study, since the article notes this exclusion as a limitation

## Inputs

- Raw scRNA-seq expression matrix (cells × genes, e.g. Xin GSE81608, 1492 cells × 239,851 genes) without row/column names
- Preprocessed matrix: genes filtered at 2.5%/97.5% cell-expression cutoffs, top 2000 highly variable genes, log-transformed and normalized to [0,1]

## Outputs

- Binarized gene expression matrix (0/1 entries per gene per cell) with genes sorted by decreasing zero count
- Binary gene-gene similarity graph (S = 1 − h/n) feeding LC1 gene clustering
- Qualitative cell-landmark matrix Q1 (cells as rows, consensus binary landmark columns per retained gene cluster, saved as q1_landmark_matrix.csv)

## How to apply

First preprocess the raw expression matrix in R (pre.R): filter genes expressed in fewer than 2.5% or more than 97.5% of cells, retain 2000 highly variable genes (Seurat-style method of references [11],[25]), then log-transform and normalize all entries to [0,1]. Then binarize each gene vector within scQA's LC1: set entries to 0 where the normalized expression value is zero and 1 otherwise, and sort genes by decreasing number of zeros. This binarization intentionally preserves dropout zeros because subsequent steps compute binary gene-gene similarity S = 1 − h/n (normalized Hamming distance) between neighboring genes in the sorted sequence, so treating dropouts as weak signals allows gene amalgamation into consensus binary landmarks (ρ = 0.7 similarity threshold, clusters of ≥3 genes retained after pruning). The output feeds the qualitative cell-landmark matrix Q1 with one binary landmark column per retained gene cluster.

## Related tools

- **scQA** (Implements the LC1 binarization step and downstream gene amalgamation into binary qualitative landmarks) — https://github.com/LD-Lyndee/scQA
- **Seurat** (Style of highly variable gene selection (method of references [11],[25]) used during preprocessing before binarization)

## Examples

```
./main xin_matrix.txt o
```

## Evaluation signals

- Binarized matrix contains only 0/1 entries, with 0 exactly where the normalized expression value was zero
- Genes are sorted by decreasing number of zeros before similarity computation (S = 1 − h/n between 0.05×m neighbors on each side)
- Only the top 1999 gene pairs (one thousandth of all pairs) are retained as edges in the gene similarity graph
- Gene clusters retained after pruning contain at least 3 genes; cluster template vector assigns 1 where ≥0.5 (fraction) of cluster genes are expressed
- Q1 landmark matrix has cells as rows and roughly a dozen (≈ number of retained clusters) binary landmark columns, saved as q1_landmark_matrix.csv

## Limitations

- Retaining only 2000 HVGs excluded some informative genes
- Genes may be misclassified in LC1 due to the influence of high dimensionality
- Version of R used for pre-processing is not stated
- The input matrix must be cells × genes without row or column names

## Evidence

- [methods] Binarize each gene vector in scQA (LC1): set the entry to 0 if the expression value is zero and 1 otherwise: "Qualitative landmark constructor (LC1) initially binarizes the matrix and persists to cluster genes with qualitative quasi-trend-preserved expression patterns to form landmarks."
- [intro] Binarization treats dropouts as weak signals: "we regard dropouts as weak signals, strengthened by amalgamation of genes sharing analogous binary expression patterns"
- [methods] Preprocessing thresholds before binarization: "we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5"
- [methods] HVG retention before binarization: "we retain 2000 highly variable genes, employing the method outlined in references [11], [25]"
- [methods] Normalization range before binarization: "All data are log-transformed and normalized to ensure that each entry falls in the range [0,1]."
- [readme] Input matrix format requirement: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
- [results] Limitation: HVG exclusion: "the retention of only 2000 HVGs for subsequent clustering analysis led to the exclusion of some informative genes"
