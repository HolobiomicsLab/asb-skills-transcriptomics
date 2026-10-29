---
name: gene-expression-csv-formatting
description: Use when preparing cross-platform transcriptome data (e.g., GEO series
  such as GSE151189 or GSE59098) for ingestion by Ciclops, whose --train_path/--valid_path
  inputs must be .csv files with sample names as the first column and labels as the
  last column.
when_to_use_negative:
- The dataset is already a preprocessed feature table with the required Ciclops column
  layout (first column sample names, last column labels) — reformatting risks corrupting
  columns.
- The downstream analysis does not use Ciclops or a tool requiring this specific sample-first/label-last
  .csv convention.
- Training and validation datasets share no common genes — the formatting step alone
  will not fix this; Ciclops will fail silently, producing csv splits with only sample
  names and labels and no gene expression columns.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Ciclops
  role: Consumes the formatted .csv files via --train_path and --valid_path; its input
    contract (first column sample names, last column labels) defines the required
    format.
  repo: https://github.com/GuanLab/ciclops.git
- name: Python
  role: Used to run the ciclops command on the formatted csv inputs and to download/preprocess
    GEO datasets into the required layout.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xpro.2022.101583
    title: Protocol for using Ciclops to build models trained on cross-platform transcriptome
      data for clinical outcome prediction
  source_content_hash: sha256:19ce1e9866efeb5538179806a25bdd10de2c51325662a1114837d9aacaa54799
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GuanLab/ciclops.git
  tools:
  - Ciclops
  - LightGBM
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gene-expression-csv-formatting

## Summary

Formats GEO transcriptomic datasets (e.g., GSE151189 and GSE59098) into the .csv layout Ciclops requires: sample names in the first column, gene expression values in between, and class labels in the last column. Correct formatting is a prerequisite for the subsequent Ciclops training and transfer-learning pipeline.

## When to use

Use when preparing cross-platform transcriptome data (e.g., GEO series such as GSE151189 or GSE59098) for ingestion by Ciclops, whose --train_path/--valid_path inputs must be .csv files with sample names as the first column and labels as the last column.

## When NOT to use

- The dataset is already a preprocessed feature table with the required Ciclops column layout (first column sample names, last column labels) — reformatting risks corrupting columns.
- The downstream analysis does not use Ciclops or a tool requiring this specific sample-first/label-last .csv convention.
- Training and validation datasets share no common genes — the formatting step alone will not fix this; Ciclops will fail silently, producing csv splits with only sample names and labels and no gene expression columns.

## Inputs

- Transcriptomic dataset downloaded from GEO (e.g., GSE151189, GSE59098)
- Gene expression matrix with per-sample rows, gene columns, and outcome labels

## Outputs

- Formatted .csv file per dataset: sample names as first column, labels as last column, suitable for --train_path/--valid_path in Ciclops

## How to apply

Download the target GEO transcriptome datasets (the protocol uses GSE151189 and GSE59098) and export each as a .csv file. Ensure the file layout matches Ciclops' documented requirement: sample names occupy the first column, gene expression measurements fill the middle columns, and class labels occupy the last column. Save one .csv per dataset so each can be passed independently as --train_path or --valid_path to the pipeline. Before running, sanity-check that the training and validation files share common gene columns, because Ciclops fails silently when train and test sets share no common genes, producing split csvs with only sample-name and label columns.

## Related tools

- **Ciclops** (Consumes the formatted .csv files via --train_path and --valid_path; its input contract (first column sample names, last column labels) defines the required format.) — https://github.com/GuanLab/ciclops.git
- **Python** (Used to run the ciclops command on the formatted csv inputs and to download/preprocess GEO datasets into the required layout.)

## Examples

```
python ciclops --train_path GSE151189.csv --valid_path GSE59098.csv
```

## Evaluation signals

- The .csv opens with the expected row count (one row per sample plus header) and the first column contains sample names, not gene identifiers.
- The last column contains the class labels for the clinical outcome, with no label values missing or shifted into gene columns.
- Both the training and validation csvs contain the same set of gene expression columns (common genes), avoiding Ciclops' silent no-common-genes failure.
- Running 'python ciclops --train_path <train.csv> --valid_path <valid.csv>' completes and the generated ./training/ and ./validation/ folders contain preprocessed splits with gene expression columns present (not just sample names and labels).

## Limitations

- Ciclops fails silently when the training and test sets share no common genes: the csv files in ./training/ contain only two columns (sample names and labels), missing all gene expression columns — verify common genes before attributing downstream failure to formatting.
- Even correctly formatted inputs can yield unreliable results (NaN evaluation metrics or confidence intervals spanning [0, 1] for AUPRC) without automatic safeguards, so formatting must be validated against downstream run outputs.
- Model performance disparities may stem from data collection methods or environmental factors drastically impacting gene expression, independent of formatting quality.

## Evidence

- [readme] README usage contract: "--train_path TRAIN_PATH
                       Path to your training data, in .csv format; includes sample names as first column and labels as last column"
- [readme] valid_path contract: "--valid_path VALID_PATH
                       Path to your transfer validation data, in .csv format; includes sample names as first column and labels as last column"
- [discussion] silent no-common-genes failure: "the csv files containing the train/test splits in the ./training directory only have two columns: sample names and labels, missing any gene expression columns"
- [discussion] unreliable output signal: "confidence intervals spanning [0, 1] for AUPRC, or NaN values for any of the evaluation metrics"
