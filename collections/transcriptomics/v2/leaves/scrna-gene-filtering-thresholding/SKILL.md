---
name: scrna-gene-filtering-thresholding
description: Use when a raw single-cell RNA-seq count matrix (.csv, cells as rows,
  genes as columns) must be converted into the pre-processed 1000-gene matrix format
  consumed by AutoImpute.py before imputation.
when_to_use_negative:
- The matrix has already been pre-processed and is in <dataset_name>.{mat, csv} format
  — feed it directly to AutoImpute.py instead of re-running pre-processing
- The downstream tool is not AutoImpute and requires different gene-selection or normalization
  conventions (e.g. different gene counts or normalization schemes)
- The data are not single-cell gene expression counts (e.g. bulk RNA-seq or proteomics),
  for which the >=3-reads-in-3-cells filter is not calibrated
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutoImpute
  role: Downstream model whose expected input format (<dataset_name>.{mat, csv}) and
    pre-processing pipeline (pre-process.R) this skill implements
  repo: github.com/divyanshu-talwar/AutoImpute
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:bcd8e09b853de18d08b1b82e89e9b09febe63d920ed52bc1dac8fc9b98880d67
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
yes_rate: 0.875
n_claims: 8
---

# scrna-gene-filtering-thresholding

## Summary

Preprocesses raw scRNA-seq count matrices for the AutoImpute autoencoder by gene filtering, median library-size normalization, dispersion-based gene selection (top 1000 genes), and log2 transformation, producing the <dataset_name>.{mat, csv} input the model expects.

## When to use

Use when a raw single-cell RNA-seq count matrix (.csv, cells as rows, genes as columns) must be converted into the pre-processed 1000-gene matrix format consumed by AutoImpute.py before imputation.

## When NOT to use

- The matrix has already been pre-processed and is in <dataset_name>.{mat, csv} format — feed it directly to AutoImpute.py instead of re-running pre-processing
- The downstream tool is not AutoImpute and requires different gene-selection or normalization conventions (e.g. different gene counts or normalization schemes)
- The data are not single-cell gene expression counts (e.g. bulk RNA-seq or proteomics), for which the >=3-reads-in-3-cells filter is not calibrated

## Inputs

- Raw scRNA-seq count matrix in .csv format (cells as rows, genes as columns), placed in AutoImpute\ Model/data/raw/

## Outputs

- Pre-processed expression matrix in <dataset_name>.{mat, csv} format with exactly 1000 gene columns (highest-dispersion genes), log2-transformed

## How to apply

Load the raw count matrix from data/raw as a .csv file, then run the R-script pre-process.R which implements the pipeline: retain only genes detected with >=3 reads in at least three cells, discard un-expressed genes, apply median normalization by dividing each read count by its cell total and multiplying by the median of total read counts across cells, select the top 1000 genes ranked by dispersion (coefficient of variance), and log2 transform after adding a pseudo-count of 1. Save the result in the <dataset_name>.{mat, csv} format that AutoImpute.py reads via --data. The rationale is to remove sparse/uninformative genes and stabilize variance so the overcomplete autoencoder learns the inherent distribution of the input scRNA-seq data.

## Related tools

- **AutoImpute** (Downstream model whose expected input format (<dataset_name>.{mat, csv}) and pre-processing pipeline (pre-process.R) this skill implements) — github.com/divyanshu-talwar/AutoImpute

## Examples

```
Rscript pre-process.R Blakeley_raw_data.csv blakeley
```

## Evaluation signals

- The processed matrix contains exactly 1000 gene columns, corresponding to the top-dispersion genes
- Output files are saved in the pre-processed <dataset_name>.{mat, csv} format in AutoImpute\ Model/data/ that AutoImpute.py accepts via --data
- Genes retained pass the filter of >=3 reads detected in at least three cells; un-expressed genes are discarded
- Expression values are log2-transformed after adding 1 as a pseudo-count (no zero or negative values from the transform)
- Each cell's total counts are normalized relative to the median of total read counts across cells (median normalization applied)

## Limitations

- Fixing selection to the top 1000 genes by dispersion discards lower-dispersion genes, which may lose biologically relevant signal outside that set
- Python 2.7 and R dependency stacks (numpy, scikit-learn, tensorflow; R.matlab, Matrix, irlba, etc.) are required, which may pose environment-compatibility issues
- The >=3-reads-in-3-cells filter is calibrated for sparse single-cell data with dropouts; applying it to denser or non-count data is unsupported by the source
- No discussion section content was available in the source document, so caveats from the authors' discussion are not captured here

## Evidence

- [other] The raw gene expression matrix is preprocessed by filtering for bad genes, normalizing by library size, pruning by gene-selection, and log transforming before being fed into the AutoImpute model.: [evidence span withheld — anchor status: not_found]
- [readme] README: input format and pipeline image: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells. It's output is an imputed count matrix with the same dimension."
- [readme] README: pre-process.R invocation: "Rscript pre-process.R <input-file-name> <dataset-name>"
- [intro] Article: AutoImpute learns inherent distribution: "AutoImpute learns the inherent distribution of the input scRNA-seq data and imputes the missing values accordingly with minimal modification to the biologically silent gene expression values."
