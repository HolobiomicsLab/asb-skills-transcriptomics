---
name: quantile-normalization-cross-platform
description: Use when preparing .csv training and validation expression matrices with
  unequal platform distributions and missing values before running Ciclops cross-platform
  clinical outcome prediction, or when reproducing the Ciclops preprocessing stage.
when_to_use_negative:
- When the user explicitly passes --no_quantile to Ciclops, indicating quantile normalization
  should be skipped
- When inputs are not expression matrices (e.g., already a feature/importance table
  or SHAP output); this skill operates on raw .csv expression matrices only
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Ciclops
  role: Pipeline whose preprocessing stage performs gene-wise imputation and quantile
    normalization; --no_quantile flag disables quantile normalization
  repo: https://github.com/GuanLab/ciclops.git
- name: Python
  role: Used with pandas to load the GSE151189 and GSE59098 .csv matrices, impute
    gene-wise, and write preprocessed CSVs
- name: pip
  role: Installs the ciclops package via 'pip install ciclops'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xpro.2022.101583
    title: Protocol for using Ciclops to build models trained on cross-platform transcriptome
      data for clinical outcome prediction
  source_content_hash: sha256:674edace12aa9157e5611a29f5d7cad37af65a580af69cb3b6c4a8471b0024da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GuanLab/ciclops.git
  tools:
  - Ciclops
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# quantile-normalization-cross-platform

## Summary

Preprocessing step of the Ciclops pipeline that gene-wise imputes missing gene expression values (replacing each sample's missing value with that gene's average expression across samples) and applies quantile normalization to align training (e.g., GSE151189) and validation (e.g., GSE59098) transcriptomic distributions before cross-platform model training.

## When to use

Use when preparing .csv training and validation expression matrices with unequal platform distributions and missing values before running Ciclops cross-platform clinical outcome prediction, or when reproducing the Ciclops preprocessing stage.

## When NOT to use

- When the user explicitly passes --no_quantile to Ciclops, indicating quantile normalization should be skipped
- When inputs are not expression matrices (e.g., already a feature/importance table or SHAP output); this skill operates on raw .csv expression matrices only

## Inputs

- Training expression matrix in .csv format (e.g., GSE151189), sample names as first column, labels as last column
- Transfer validation expression matrix in .csv format (e.g., GSE59098), sample names as first column, labels as last column

## Outputs

- Preprocessed training CSV written to ./training/ with gene expression columns present and no missing values
- Preprocessed validation CSV written to ./validation/ with gene expression columns present and no missing values
- Distribution-aligned training and validation matrices ready for Ciclops model training and 10-fold cross validation

## How to apply

Load the training matrix (e.g., GSE151189) and validation matrix (e.g., GSE59098) as .csv files with sample names as the first column and labels as the last column using pandas. Impute missing values gene-wise: for each sample, replace a missing gene value with that gene's average expression level across samples. Then apply quantile normalization jointly to the training and validation expression matrices so their empirical distributions are aligned — this is the step disabled by the Ciclops --no_quantile flag. Write the preprocessed matrices to the ./training and ./validation directories as CSV files, retaining sample names as the first column and labels as the last column, guarding against the known silent-failure mode where train/test share no common genes and only sample-name and label columns are written.

## Related tools

- **Ciclops** (Pipeline whose preprocessing stage performs gene-wise imputation and quantile normalization; --no_quantile flag disables quantile normalization) — https://github.com/GuanLab/ciclops.git
- **Python** (Used with pandas to load the GSE151189 and GSE59098 .csv matrices, impute gene-wise, and write preprocessed CSVs)
- **pip** (Installs the ciclops package via 'pip install ciclops')

## Examples

```
pip install ciclops; python ciclops --train_path GSE151189.csv --valid_path GSE59098.csv -m lgb
```

## Evaluation signals

- Output CSV files exist in ./training/ and ./validation/ directories
- Gene expression columns are present in output CSVs (guarding against the silent-failure mode where only sample-name and label columns are written when train/test share no common genes)
- No missing values remain in the imputed matrices (gene-wise imputation replaced each missing value with the gene's average expression)
- Empirical distributions of training and validation expression matrices are aligned after joint quantile normalization

## Limitations

- Ciclops fails silently when training and test sets share no common genes, producing csv splits with only two columns (sample names and labels) and no gene expression columns
- Quantile normalization is skipped entirely if the --no_quantile flag is specified, changing the preprocessing behavior
- No software version or changelog is provided for Ciclops, complicating reproducibility of the preprocessing behavior

## Evidence

- [methods] Before model training, Ciclops preprocesses both training and testing datasets by performing gene-wise imputation, replacing a sample's missing gene value with that gene's average expression level, followed by quantile normalization.: [evidence span withheld — anchor status: not_found]
- [readme] --no_quantile         If specified, do not use quantile normalization.: "--no_quantile         If specified, do not use quantile normalization."
- [readme] ./training/: preprocessed training datasets for model training and 10-fold cross validation: "./training/: preprocessed training datasets for model training and 10-fold cross validation"
- [discussion] the csv files containing the train/test splits in the ./training directory only have two columns: sample names and labels, missing any gene expression columns: "the csv files containing the train/test splits in the ./training directory only have two columns: sample names and labels"
- [readme] --train_path TRAIN_PATH
                       Path to your training data, in .csv format; includes sample names as first column and labels as last column: "Path to your training data, in .csv format; includes sample names as first column and labels as last column"
