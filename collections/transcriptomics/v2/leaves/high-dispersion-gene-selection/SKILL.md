---
name: high-dispersion-gene-selection
description: Use when a raw single-cell count matrix must be reduced to a manageable,
  informative gene set (highest-dispersion genes) before autoencoder-based dropout
  imputation with AutoImpute.
when_to_use_negative:
- The input matrix is already pre-processed and stored in the <dataset_name>.{mat,
  csv} format expected by AutoImpute.py
- The analysis target is not gene-expression imputation and does not require restricting
  to a high-dispersion gene subset (e.g., full-transcriptome differential expression)
- The dataset has fewer than 1000 genes passing the >=3 reads-in-at-least-3-cells
  filter, making the top-1000 selection ill-posed
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: AutoImpute
  role: R pre-processing scripts (pre-process.R) implement gene filtering, median
    normalization, top-1000 dispersion gene selection, and log2 transformation; the
    output feeds the AutoImpute.py autoencoder model
  repo: github.com/divyanshu-talwar/AutoImpute
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:d3efcff1e7d4a442ff7b21bb282f12c30bd96cbe2f9c23265bd5ec4dfa31be8c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# High-Dispersion Gene Selection

## Summary

Reduces a scRNA-seq count matrix to the top 1000 genes ranked by dispersion (coefficient of variance) as the pre-processing step required before feeding data to the AutoImpute autoencoder imputation model.

## When to use

Use when a raw single-cell count matrix must be reduced to a manageable, informative gene set (highest-dispersion genes) before autoencoder-based dropout imputation with AutoImpute.

## When NOT to use

- The input matrix is already pre-processed and stored in the <dataset_name>.{mat, csv} format expected by AutoImpute.py
- The analysis target is not gene-expression imputation and does not require restricting to a high-dispersion gene subset (e.g., full-transcriptome differential expression)
- The dataset has fewer than 1000 genes passing the >=3 reads-in-at-least-3-cells filter, making the top-1000 selection ill-posed

## Inputs

- Raw scRNA-seq count matrix in .csv format (cells as rows, genes as columns), placed in AutoImpute Model/data/raw/

## Outputs

- Pre-processed expression matrix limited to the 1000 highest-dispersion genes, log2-transformed with pseudo-count 1, stored as <dataset_name>.{mat, csv} in AutoImpute Model/data/

## How to apply

Load the raw scRNA-seq count matrix (cells as rows, genes as columns) from the raw data directory as a .csv file. Apply gene filtering to retain only genes detected with >=3 reads in at least three cells, discarding un-expressed genes. Perform median normalization by dividing each read count by the total counts in its cell and multiplying by the median of the total read counts across cells. Rank genes by dispersion (coefficient of variance) and select the top 1000 genes for downstream analysis. Log2 transform the selected expression matrix after adding 1 as a pseudo-count, and save the result in the <dataset_name>.{mat, csv} format consumed by AutoImpute.py.

## Related tools

- **AutoImpute** (R pre-processing scripts (pre-process.R) implement gene filtering, median normalization, top-1000 dispersion gene selection, and log2 transformation; the output feeds the AutoImpute.py autoencoder model) — github.com/divyanshu-talwar/AutoImpute

## Examples

```
Rscript pre-process.R Blakeley_raw_data.csv blakeley
```

## Evaluation signals

- The output matrix must contain exactly 1000 gene columns (the highest-dispersion genes)
- The output must be stored in the pre-processed <dataset_name>.{mat, csv} format that AutoImpute.py consumes via --data <dataset_name>.{mat, csv}
- Genes not detected with >=3 reads in at least three cells must be absent from the output matrix
- Expression values must be log2(x+1)-transformed, so all entries are non-negative and zero counts map to 0
- The processed matrix must preserve the original cells (rows) with columns representing genes and rows representing cells as AutoImpute.py expects

## Limitations

- Selecting the top 1000 genes by coefficient of variance discards the rest of the transcriptome; results apply only to that reduced gene space
- The >=3 reads in at least three cells filter discards un-expressed genes, which may remove biologically relevant but very lowly expressed genes
- The pre-processing pipeline is R-based with specific R dependencies (R.matlab, Matrix, dplyr, irlba, dynamicTreeCut, GenSA, etc.); missing packages will break the workflow
- For datasets other than the provided sample (e.g., blakeley.csv), the necessary pre-processing must be run before invoking AutoImpute

## Evidence

- [methods] workflow gene filter: [evidence span withheld — anchor status: not_found]
- [methods] workflow gene selection: [evidence span withheld — anchor status: not_found]
- [readme] README input format: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [readme] README pre-process invocation: "Run the R-script `pre-process.R` using the following command:
bash
Rscript pre-process.R <input-file-name> <dataset-name>
"
- [intro] article rationale: "AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values."
