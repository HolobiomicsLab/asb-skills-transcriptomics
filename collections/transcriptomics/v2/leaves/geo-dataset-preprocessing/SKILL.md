---
name: geo-dataset-preprocessing
description: Use when raw GEO expression matrices (RNA-seq or microarray .csv with
  sample names as first column and labels as last column) contain missing gene values
  or platform distribution differences that must be resolved before cross-platform
  training with Ciclops.
when_to_use_negative:
- The input matrices have already been imputed and quantile-normalized, or you intend
  to run Ciclops with the --no_quantile flag
- Training and validation sets share no common genes — the pipeline will fail silently
  and write CSVs containing only sample names and labels
- The downstream task is not cross-platform clinical outcome prediction (e.g., single-platform
  analysis where distribution alignment is unnecessary)
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: Ciclops
  role: Pipeline whose expected preprocessing (gene-wise imputation and quantile normalization,
    disabled by --no_quantile) this skill reproduces or verifies
  repo: https://github.com/GuanLab/ciclops.git
- name: Python
  role: Used with pandas to load the GSE151189 and GSE59098 CSVs, perform gene-wise
    imputation, apply quantile normalization, and write ./training and ./validation
    CSVs
- name: pip
  role: Installs the ciclops package whose CLI paths and quantile flag define the
    preprocessing contract
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xpro.2022.101583
    title: Protocol for using Ciclops to build models trained on cross-platform transcriptome
      data for clinical outcome prediction
  source_content_hash: sha256:07d055f2a6ba838922a21e32faee4b0f11be12ab8e1962667d1acdfed48335ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GuanLab/ciclops.git
  tools:
  - Ciclops
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# geo-dataset-preprocessing

## Summary

Preprocesses GEO transcriptomic training and validation matrices (e.g., GSE151189 RNA-seq and GSE59098 microarray CSVs) for the Ciclops pipeline by gene-wise imputation of missing expression values followed by quantile normalization, so cross-platform distributions are aligned before model training.

## When to use

Use when raw GEO expression matrices (RNA-seq or microarray .csv with sample names as first column and labels as last column) contain missing gene values or platform distribution differences that must be resolved before cross-platform training with Ciclops.

## When NOT to use

- The input matrices have already been imputed and quantile-normalized, or you intend to run Ciclops with the --no_quantile flag
- Training and validation sets share no common genes — the pipeline will fail silently and write CSVs containing only sample names and labels
- The downstream task is not cross-platform clinical outcome prediction (e.g., single-platform analysis where distribution alignment is unnecessary)

## Inputs

- GEO training expression matrix in .csv format (e.g., GSE151189), sample names as first column, labels as last column
- GEO transfer validation expression matrix in .csv format (e.g., GSE59098), sample names as first column, labels as last column

## Outputs

- Preprocessed training CSV written to ./training/ with gene expression columns and no missing values
- Preprocessed validation CSV written to ./validation/ with gene expression columns aligned to the training feature space

## How to apply

Load the training matrix (e.g., GSE151189) and validation matrix (e.g., GSE59098) as CSV with pandas, keeping sample names in the first column and labels in the last column. Impute missing values gene-wise: replace each sample's missing value for a gene with that gene's average expression across samples. Apply quantile normalization jointly to the training and validation expression matrices so their empirical distributions are aligned — this is the step the Ciclops --no_quantile flag would otherwise disable. Write the preprocessed matrices to ./training/ and ./validation/ as CSV files retaining sample names and labels, then confirm gene expression columns survived (the pipeline fails silently when train/test share no common genes) and that no NaNs remain.

## Related tools

- **Ciclops** (Pipeline whose expected preprocessing (gene-wise imputation and quantile normalization, disabled by --no_quantile) this skill reproduces or verifies) — https://github.com/GuanLab/ciclops.git
- **Python** (Used with pandas to load the GSE151189 and GSE59098 CSVs, perform gene-wise imputation, apply quantile normalization, and write ./training and ./validation CSVs)
- **pip** (Installs the ciclops package whose CLI paths and quantile flag define the preprocessing contract)

## Examples

```
import pandas as pd; df = pd.read_csv('GSE151189.csv', index_col=0); df = df.apply(lambda c: c.fillna(c.mean())); df.to_csv('./training/train_preprocessed.csv')
```

## Evaluation signals

- Output CSVs exist in ./training/ and ./validation/ and retain sample names as the first column and labels as the last column
- Preprocessed matrices contain gene expression columns — guarding against the silent-failure mode where train/test share no common genes and only sample-name and label columns are written
- No missing values (NaNs) remain in either imputed matrix
- Training and validation empirical distributions are aligned after joint quantile normalization; identical gene sets/order across the two files

## Limitations

- The pipeline fails silently when training and test sets share no common genes, producing train/test split CSVs with only sample names and labels and no gene expression columns
- Quantile normalization is on by default and can only be disabled via the --no_quantile flag; it is not adaptive to whether it is appropriate for the data
- No changelog or software version is provided for Ciclops, so preprocessing behavior cannot be pinned to a release

## Evidence

- [readme] README usage: csv input format: "in .csv format; includes sample names as first column and labels as last column"
- [readme] README: quantile flag: "--no_quantile         If specified, do not use quantile normalization."
- [methods] Article methods: preprocessing output folders: "preprocessed training datasets for model training and 10-fold cross validation"
- [discussion] Article discussion: silent failure mode: "the csv files containing the train/test splits in the ./training directory only have two columns: sample names and labels, missing any gene expression columns"
