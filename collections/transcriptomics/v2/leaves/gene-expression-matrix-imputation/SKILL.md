---
name: gene-expression-matrix-imputation
description: Use when preparing .csv gene expression matrices (e.g., GSE151189 training
  and GSE59098 validation) with missing values for cross-platform model training in
  Ciclops, where train and test data must share aligned gene distributions.
when_to_use_negative:
- When the training and test matrices share no common genes — Ciclops fails silently,
  writing csv splits with only sample-name and label columns and no gene expression
  columns
- When the user explicitly intends to skip distribution alignment, since the --no_quantile
  flag exists to disable quantile normalization by design
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: Ciclops
  role: Pipeline whose preprocessing performs gene-wise imputation followed by quantile
    normalization; exposes --no_quantile to disable the normalization step
  repo: https://github.com/GuanLab/ciclops.git
- name: Python
  role: Used with pandas to load the .csv training and validation matrices and write
    the imputed, normalized outputs
- name: pip
  role: Installs the ciclops package that contains the preprocessing pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xpro.2022.101583
    title: Protocol for using Ciclops to build models trained on cross-platform transcriptome
      data for clinical outcome prediction
  source_content_hash: sha256:54b648471397897e63f9225c88ae42805d9d5f87aeba6cb9ce7c46beb6894673
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GuanLab/ciclops.git
  tools:
  - Ciclops
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gene-expression-matrix-imputation

## Summary

Preprocessing skill from the Ciclops pipeline that fills missing values in cross-platform transcriptomic matrices by gene-wise imputation (replacing a sample's missing value with that gene's average expression across samples) and then applies quantile normalization to align training and validation distributions before model construction.

## When to use

Use when preparing .csv gene expression matrices (e.g., GSE151189 training and GSE59098 validation) with missing values for cross-platform model training in Ciclops, where train and test data must share aligned gene distributions.

## When NOT to use

- When the training and test matrices share no common genes — Ciclops fails silently, writing csv splits with only sample-name and label columns and no gene expression columns
- When the user explicitly intends to skip distribution alignment, since the --no_quantile flag exists to disable quantile normalization by design

## Inputs

- Training expression matrix in .csv format (e.g., GSE151189), sample names as first column, labels as last column
- Transfer validation expression matrix in .csv format (e.g., GSE59098), sample names as first column, labels as last column

## Outputs

- Preprocessed training CSV files in ./training/ (gene-wise imputed, quantile-normalized, 10-fold cross-validation splits)
- Preprocessed validation CSV files in ./validation/ for transferring test

## How to apply

Load the training matrix (e.g., GSE151189) and validation matrix (e.g., GSE59098) as .csv files with sample names as the first column and labels as the last column, using pandas in Python. Impute missing values gene-wise: for each sample, replace a missing gene value with that gene's average expression level computed across samples. Then apply quantile normalization jointly to the training and validation expression matrices so their empirical distributions are aligned — this is the step disabled by the Ciclops --no_quantile flag. Write the preprocessed matrices to ./training/ and ./validation/ as CSV files, retaining sample names as the first column and labels as the last column, so Ciclops can run model training with 10-fold cross validation and transfer learning.

## Related tools

- **Ciclops** (Pipeline whose preprocessing performs gene-wise imputation followed by quantile normalization; exposes --no_quantile to disable the normalization step) — https://github.com/GuanLab/ciclops.git
- **Python** (Used with pandas to load the .csv training and validation matrices and write the imputed, normalized outputs)
- **pip** (Installs the ciclops package that contains the preprocessing pipeline)

## Examples

```
pip install ciclops; python ciclops --train_path GSE151189.csv --valid_path GSE59098.csv -m lgb
```

## Evaluation signals

- Output CSV files exist in ./training/ and ./validation/ directories
- Preprocessed CSVs contain gene expression columns (guarding against the known silent-failure mode where only sample-name and label columns are written when train/test share no common genes)
- No missing values remain in the imputed matrices after gene-wise imputation
- Empirical distributions of training and validation gene expression are aligned after joint quantile normalization
- First column retains sample names and last column retains labels, matching the .csv schema Ciclops expects

## Limitations

- Ciclops fails silently when training and test sets share no common genes, producing incomplete csv splits with only two columns: sample names and labels
- Results can be unreliable without automatic safeguards, e.g., confidence intervals spanning [0, 1] for AUPRC or NaN evaluation metrics
- No software version or changelog is provided for Ciclops

## Evidence

- [methods] Gene-wise imputation followed by quantile normalization before model training: [evidence span withheld — anchor status: not_found]
- [readme] Ciclops --no_quantile flag: "--no_quantile         If specified, do not use quantile normalization."
- [readme] Expected folder outputs of preprocessing: "`./training/`: preprocessed training datasets for model training and 10-fold cross validation"
- [discussion] Silent-failure mode when no common genes: "the csv files containing the train/test splits in the ./training directory only have two columns: sample names and labels, missing any gene expression columns"
- [readme] Input CSV schema: "Path to your training data, in .csv format; includes sample names as first column and labels as last column"
