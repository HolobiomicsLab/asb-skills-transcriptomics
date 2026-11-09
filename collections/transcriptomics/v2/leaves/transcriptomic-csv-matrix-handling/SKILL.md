---
name: transcriptomic-csv-matrix-handling
description: Use when loading raw RNA-seq/microarray expression .csv files (sample
  names in first column, labels in last column) that contain missing gene values and
  must be imputed and quantile-normalized before cross-platform training in Ciclops.
when_to_use_negative:
- The --no_quantile flag is specified, meaning quantile normalization should be skipped
- Input already contains complete, normalized expression values shared between train
  and test sets — re-imputing or re-normalizing could distort the data
- Training and validation sets share no common genes — the silent-failure mode makes
  downstream preprocessing meaningless
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: Python
  role: Used with pandas to load .csv matrices, perform gene-wise imputation, and
    write preprocessed CSVs
- name: Ciclops
  role: Consumes the preprocessed training and validation CSVs; its --no_quantile
    flag toggles the quantile normalization step this skill implements
  repo: https://github.com/GuanLab/ciclops.git
- name: pip
  role: Installs the ciclops package used downstream of preprocessing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xpro.2022.101583
    title: Protocol for using Ciclops to build models trained on cross-platform transcriptome
      data for clinical outcome prediction
  source_content_hash: sha256:d8f0a8f561ddfd2619bc9e944e10c4ceef4654e0ed0e3b02b817b2b68eebe561
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GuanLab/ciclops.git
  tools:
  - Ciclops
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# transcriptomic-csv-matrix-handling

## Summary

Preprocess cross-platform transcriptomic .csv matrices (e.g., GSE151189 training and GSE59098 validation) for Ciclops by performing gene-wise imputation of missing values followed by joint quantile normalization, so training and validation empirical distributions are aligned before model construction.

## When to use

Use when loading raw RNA-seq/microarray expression .csv files (sample names in first column, labels in last column) that contain missing gene values and must be imputed and quantile-normalized before cross-platform training in Ciclops.

## When NOT to use

- The --no_quantile flag is specified, meaning quantile normalization should be skipped
- Input already contains complete, normalized expression values shared between train and test sets — re-imputing or re-normalizing could distort the data
- Training and validation sets share no common genes — the silent-failure mode makes downstream preprocessing meaningless

## Inputs

- Training expression matrix in .csv format (e.g., GSE151189), sample names as first column, labels as last column
- Validation/transfer expression matrix in .csv format (e.g., GSE59098), sample names as first column, labels as last column

## Outputs

- Preprocessed training CSV files in ./training/ with gene expression columns, sample names first, labels last
- Preprocessed validation CSV files in ./validation/ with gene expression columns, sample names first, labels last
- Imputed, quantile-normalized matrices with no remaining missing values

## How to apply

Load the training matrix (e.g., GSE151189) and validation matrix (e.g., GSE59098) as .csv with sample names as the first column and labels as the last column using pandas in Python. Impute missing values gene-wise, replacing each sample's missing value for a gene with that gene's average expression across samples. Apply quantile normalization to the training and validation expression matrices jointly so their empirical distributions are aligned — this is the step the Ciclops --no_quantile flag would otherwise disable. Write the preprocessed matrices to ./training/ and ./validation/ as CSV files, retaining sample names as the first column and labels as the last column.

## Related tools

- **Python** (Used with pandas to load .csv matrices, perform gene-wise imputation, and write preprocessed CSVs)
- **Ciclops** (Consumes the preprocessed training and validation CSVs; its --no_quantile flag toggles the quantile normalization step this skill implements) — https://github.com/GuanLab/ciclops.git
- **pip** (Installs the ciclops package used downstream of preprocessing)

## Examples

```
import pandas as pd; train = pd.read_csv('GSE151189.csv', index_col=0); train = train.fillna(train.mean()); valid = pd.read_csv('GSE59098.csv', index_col=0); valid = valid.fillna(valid.mean()); train.to_csv('./training/train_preprocessed.csv'); valid.to_csv('./validation/valid_preprocessed.csv')
```

## Evaluation signals

- Output CSV files exist in ./training/ and ./validation/
- Gene expression columns are present in the output CSVs — guarding against the known silent-failure mode where only sample-name and label columns are written
- No missing values remain in the imputed matrices
- Sample names remain the first column and labels remain the last column in the written CSVs
- Training and validation empirical distributions are aligned after joint quantile normalization

## Limitations

- Program fails silently when training and test sets share no common genes, producing incomplete csv splits with only sample-name and label columns
- Results can be unreliable (confidence intervals spanning [0, 1] for AUPRC, or NaN evaluation metrics) without automatic safeguards
- No fixed hyperparameters are guaranteed; users must tune hyperparameters themselves for a better fit

## Evidence

- [methods] Gene-wise imputation and quantile normalization: [evidence span withheld — anchor status: not_found]
- [readme] --no_quantile flag: "--no_quantile         If specified, do not use quantile normalization."
- [readme] CSV format convention: "Path to your training data, in .csv format; includes sample names as first column and labels as last column"
- [discussion] Silent failure mode: "the csv files containing the train/test splits in the ./training directory only have two columns: sample names and labels, missing any gene expression columns"
