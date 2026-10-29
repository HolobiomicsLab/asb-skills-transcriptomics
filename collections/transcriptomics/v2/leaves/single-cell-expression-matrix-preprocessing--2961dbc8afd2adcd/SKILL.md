---
name: single-cell-expression-matrix-preprocessing
description: Use when a large scRNA-seq dataset (e.g., 254,941-cell umbilical cord
  blood or 665,858-cell mouse brain data) must be converted into the CSV cell-by-feature/PC
  matrix format that Sphetcher requires before sketching.
when_to_use_negative:
- Input is raw sequencing output (FASTQ, raw reads) rather than a cell-by-feature
  matrix or PCs — quantification/dimensionality reduction must happen first
- The analysis goal is clustering or trajectory inference on the full dataset rather
  than producing a sketch-ready matrix; run Louvain or Monocle 2 directly instead
- A sketch indicator already exists and only downstream subsetting/clustering of the
  20,000 sketched cells is needed
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Sphetcher
  role: Downstream consumer of the preprocessed CSV expression matrix; it performs
    thresholding-based spherical sketching (Pearson correlation distance) to output
    a sketch indicator vector
  repo: https://github.com/canzarlab/Sphetcher
- name: Louvain
  role: Downstream community-detection clustering applied to cells selected from the
    preprocessed matrix via the sketch indicator
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:40e89f1626f6a9ec2418c037a3d38e064d89ae315003a0939ff407c0388e6f6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Louvain
  - Sphetcher
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-expression-matrix-preprocessing

## Summary

Prepare a single-cell RNA-seq dataset as a cell-by-feature CSV expression matrix (rows are cells, columns are genes, transcripts, or principal components) so it can be consumed by Sphetcher's spherical-thresholding sketching, which evenly covers the transcriptomic space and highlights rare cell populations such as inflammatory macrophages.

## When to use

Use when a large scRNA-seq dataset (e.g., 254,941-cell umbilical cord blood or 665,858-cell mouse brain data) must be converted into the CSV cell-by-feature/PC matrix format that Sphetcher requires before sketching.

## When NOT to use

- Input is raw sequencing output (FASTQ, raw reads) rather than a cell-by-feature matrix or PCs — quantification/dimensionality reduction must happen first
- The analysis goal is clustering or trajectory inference on the full dataset rather than producing a sketch-ready matrix; run Louvain or Monocle 2 directly instead
- A sketch indicator already exists and only downstream subsetting/clustering of the 20,000 sketched cells is needed

## Inputs

- Raw or dimensionality-reduced single-cell RNA-seq expression data (e.g., umbilical cord blood matrix with 254,941 cells, Zeisel mouse nervous system, MOCA)
- Optional prior class labels as a column vector of integers 1..K (cell type, collection time point) plus a minimum representatives-per-class value l_min

## Outputs

- expression_matrix.csv: expression matrix in CSV format with rows as cells and columns as features/genes/PCs (e.g., zeisel_pca.csv style)
- class_labels.csv: integer class-label column vector aligned to rows of the expression matrix (when fairness constraints are used)

## How to apply

First normalize the scRNA-seq dataset into a matrix where rows are samples (cells) and columns are features (genes, transcripts, or principal components PCs), and store it in comma-separated values (CSV) format as required by Sphetcher. For very large datasets such as the 254,941-cell umbilical cord blood matrix, reduce to PCs so columns are tractable, mirroring the zeisel_pca.csv example input provided in the /data directory. If prior categorical knowledge such as cell type or collection time point should be preserved, also prepare a class_labels.csv column vector with integer class labels 1..K and a minimum representative count l_min per class. Rationale: Sphetcher's thresholding-based spherical sketching operates on this metric feature matrix using Pearson correlation as distance metric, so a well-formed cell-by-PC matrix directly determines sketch quality and downstream recovery of rare populations (e.g., macrophage cluster with CD74, HLA-DRA, B2M, JUNB marker AUROC > 0.88 after Louvain clustering of a 20,000-cell sketch).

## Related tools

- **Sphetcher** (Downstream consumer of the preprocessed CSV expression matrix; it performs thresholding-based spherical sketching (Pearson correlation distance) to output a sketch indicator vector) — https://github.com/canzarlab/Sphetcher
- **Louvain** (Downstream community-detection clustering applied to cells selected from the preprocessed matrix via the sketch indicator)

## Examples

```
sphetcher expression_matrix.csv 20000 sketch_indicator_output.csv
```

## Evaluation signals

- Output CSV has rows = number of cells (e.g., 254,941) and columns = features or PCs, matching the Sphetcher input contract
- Class-labels vector (if used) contains only integers between 1 and K and has one entry per row of the expression matrix
- Matrix loads in CSV readers without header/index ambiguity so the Sphetcher binary can parse it
- Feeding the matrix to Sphetcher with sketch_size 20000 yields sketch_indicator_output.csv containing exactly 20,000 ones
- Subsetting the original matrix by the indicator preserves expression values so downstream Louvain clustering can recover marker-gene AUROC > 0.88 for the macrophage cluster

## Limitations

- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study
- Sphetcher's underlying model might be less compatible with assumptions made by algorithms other than Louvain and Monocle 2, and this was not tested
- No software version numbers are stated for Sphetcher or the downstream tools used

## Evidence

- [methods] First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs).: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
- [readme] sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv: "sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv"
- [other] Load the umbilical cord blood expression matrix (254,941 cells as rows, features/PCs as columns) in CSV format as required by Sphetcher.: [evidence span withheld — anchor status: not_found]
- [readme] sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv: "sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv"
