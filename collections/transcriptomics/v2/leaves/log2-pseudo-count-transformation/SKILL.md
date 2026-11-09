---
name: log2-pseudo-count-transformation
description: Use when a raw scRNA-seq count matrix (cells x genes .csv) must be transformed
  into the normalized, log2-transformed <dataset_name>.{mat, csv} format expected
  by AutoImpute.py before imputation.
when_to_use_negative:
- The matrix has already been pre-processed into <dataset_name>.{mat, csv} format
  and is ready to be passed directly to AutoImpute.py
- The downstream method expects raw counts or a normalization/transform other than
  median normalization plus log2(x+1) (e.g. size-factor normalization for DE testing)
- Fewer than 1000 genes pass the >=3-reads-in->=3-cells filter, making the top-1000
  dispersion selection inapplicable
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: AutoImpute
  role: Downstream model whose required input format (<dataset_name>.{mat, csv} log2-transformed
    matrix) defines the output of this transformation; the pipeline ends by producing
    its input
  repo: https://github.com/divyanshu-talwar/AutoImpute
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:69d022d1fddf3237ac56dfbea18845f5f5446e17bec42954bd948b0af4488952
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/divyanshu-talwar/AutoImpute
  tools:
  - AutoImpute
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# log2-pseudo-count-transformation

## Summary

Log2 transformation with a +1 pseudo-count of the top-1000 dispersion-selected scRNA-seq gene expression matrix, the final step of the AutoImpute pre-processing pipeline (pre-process.R) that conditions sparse count data for autoencoder-based imputation.

## When to use

Use when a raw scRNA-seq count matrix (cells x genes .csv) must be transformed into the normalized, log2-transformed <dataset_name>.{mat, csv} format expected by AutoImpute.py before imputation.

## When NOT to use

- The matrix has already been pre-processed into <dataset_name>.{mat, csv} format and is ready to be passed directly to AutoImpute.py
- The downstream method expects raw counts or a normalization/transform other than median normalization plus log2(x+1) (e.g. size-factor normalization for DE testing)
- Fewer than 1000 genes pass the >=3-reads-in->=3-cells filter, making the top-1000 dispersion selection inapplicable

## Inputs

- Raw scRNA-seq count matrix in .csv format (cells as rows, genes as columns) placed in AutoImpute\ Model/data/raw/

## Outputs

- Pre-processed log2-transformed expression matrix with exactly 1000 highest-dispersion gene columns, saved in <dataset_name>.{mat, csv} format

## How to apply

Load the raw scRNA-seq count matrix (cells as rows, genes as columns) from AutoImpute\ Model/data/raw/ as a .csv file. Filter genes to those with >=3 reads in at least three cells, then apply median normalization: divide each count by its cell total and multiply by the median of total counts across cells. Select the top 1000 genes ranked by dispersion (coefficient of variance), then log2 transform after adding a pseudo-count of 1. Save the result as <dataset_name>.{mat, csv}, which is the input format consumed by AutoImpute.py. In practice this is executed via the repository's R script rather than reimplemented.

## Related tools

- **AutoImpute** (Downstream model whose required input format (<dataset_name>.{mat, csv} log2-transformed matrix) defines the output of this transformation; the pipeline ends by producing its input) — https://github.com/divyanshu-talwar/AutoImpute

## Examples

```
Rscript pre-process.R Blakeley_raw_data.csv blakeley
```

## Evaluation signals

- Output matrix contains exactly 1000 gene columns corresponding to the highest-dispersion genes
- All matrix values are non-negative log2-scale values (log2(x+1) of median-normalized counts), so minimum value is 0
- File is written in the <dataset_name>.{mat, csv} format in AutoImpute\ Model/data/ that AutoImpute.py loads via --data
- Imputed output from AutoImpute.py has the same dimensions as the transformed input matrix (cells x 1000 genes)

## Limitations

- Restricting to the top 1000 dispersion genes discards the rest of the transcriptome; imputation covers only this reduced feature set
- The pre-processing pipeline is implemented in R and pinned to the repository's R dependency stack (R.matlab, Matrix, irlba, dynamicTreeCut, etc.); Python dependencies only cover the imputation model itself
- README documents the model for Python 2.7; running the imputation step after transformation may require a legacy environment

## Evidence

- [readme] The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells.: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [readme] pre-process.R invocation: "Run the R-script `pre-process.R` using the following command:
bash
Rscript pre-process.R <input-file-name> <dataset-name>
"
- [intro] AutoImpute model goal: "AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values."
