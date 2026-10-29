---
name: sparse-expression-matrix-preprocessing
description: Use when a raw scRNA-seq count matrix in .csv format must be converted
  into the pre-processed <dataset_name>.{mat, csv} format expected by AutoImpute.py
  before dropout imputation.
when_to_use_negative:
- The data is not a sparse scRNA-seq gene expression count matrix (e.g. bulk RNA-seq
  or already-normalized/feature-engineered tables)
- The matrix has already been pre-processed into the <dataset_name>.{mat, csv} format
  — re-running pre-process.R would double-normalize and re-filter
- More than 1000 genes are needed downstream, since the pipeline hard-truncates to
  the top 1000 dispersion-ranked genes
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutoImpute
  role: Imputation model consuming the pre-processed 1000-gene matrix; the preprocessing
    pipeline exists to produce its required input format
  repo: github.com/divyanshu-talwar/AutoImpute
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:209aaed49f90e7955cceea76d2b8309d7c25495988ef013155cdc420a3733fb9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sparse-expression-matrix-preprocessing

## Summary

Preprocess a raw scRNA-seq count matrix (cells as rows, genes as columns) via gene filtering, median-of-totals normalization, top-1000 highly-dispersed gene selection, and log2(x+1) transformation so it can be consumed by the AutoImpute autoencoder imputation model.

## When to use

Use when a raw scRNA-seq count matrix in .csv format must be converted into the pre-processed <dataset_name>.{mat, csv} format expected by AutoImpute.py before dropout imputation.

## When NOT to use

- The data is not a sparse scRNA-seq gene expression count matrix (e.g. bulk RNA-seq or already-normalized/feature-engineered tables)
- The matrix has already been pre-processed into the <dataset_name>.{mat, csv} format — re-running pre-process.R would double-normalize and re-filter
- More than 1000 genes are needed downstream, since the pipeline hard-truncates to the top 1000 dispersion-ranked genes

## Inputs

- Raw scRNA-seq count matrix as .csv (cells as rows, genes as columns), placed in AutoImpute\ Model/data/raw/
- Input file name and target dataset name passed as command-line arguments to pre-process.R

## Outputs

- Pre-processed expression matrix in <dataset_name>.{mat, csv} format with exactly 1000 highly-dispersed gene columns
- Imputed count matrix produced downstream by AutoImpute.py (same dimensions as input, e.g. saved via --imputed_save)

## How to apply

Place the raw .csv data into 'AutoImpute\ Model/data/raw/' and run the R script 'pre-process.R <input-file-name> <dataset-name>'. The script filters genes, retaining only those with >=3 reads in at least three cells; performs median normalization by dividing each count by its cell's total and multiplying by the median of total read counts across cells; selects the top 1000 genes ranked by dispersion (coefficient of variance); and applies log2(x+1) transformation with a 1 pseudo-count. The output matrix must have exactly 1000 gene columns and be saved in the <dataset_name>.{mat, csv} format consumed by AutoImpute.py, after which imputation is run with 'python AutoImpute.py --data <dataset_name>.{mat, csv}'.

## Related tools

- **AutoImpute** (Imputation model consuming the pre-processed 1000-gene matrix; the preprocessing pipeline exists to produce its required input format) — github.com/divyanshu-talwar/AutoImpute

## Examples

```
Rscript pre-process.R Blakeley_raw_data.csv blakeley && python AutoImpute.py --data blakeley.csv
```

## Evaluation signals

- Output matrix contains exactly 1000 gene columns corresponding to the highest-coefficient-of-variance genes
- Output is saved in the <dataset_name>.{mat, csv} format that AutoImpute.py accepts via --data
- Values are log2-transformed after adding a 1 pseudo-count (non-positive values indicate a missed transformation step)
- Counts were median-normalized: each cell's counts divided by its total and scaled by the median of total read counts across cells
- Genes with fewer than 3 reads in fewer than 3 cells have been discarded from the output

## Limitations

- Pipeline hard-selects only the top 1000 dispersion-ranked genes, discarding the rest of the transcriptome
- Gene filtering (>=3 reads in at least three cells) may remove lowly expressed but biologically relevant genes
- Requires the specific R dependency stack (R.matlab, Matrix, svd, irlba, etc.) and Python 2.7 with numpy, scikit-learn, tensorflow, matplotlib for the downstream model

## Evidence

- [readme] Raw data filtering and format: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [readme] Preprocessing invocation: "Rscript pre-process.R <input-file-name> <dataset-name>"
- [readme] Input format for model: "run AutoImpute as `python AutoImpute.py --data <dataset_name>.{mat, csv}`"
- [methods] Pipeline steps from source card: [evidence span withheld — anchor status: not_found]
- [methods] Normalization and gene selection: [evidence span withheld — anchor status: not_found]
- [intro] Rationale for autoencoder imputation: "AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values."
