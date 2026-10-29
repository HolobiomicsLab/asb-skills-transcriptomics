---
name: hierarchical-cluster-then-annotate-cell-classification
description: Use when annotating tumor-microenvironment cell subtypes in cancer scRNA-seq
  data and direct per-cell subtype prediction is inaccurate; apply after major-type
  classification to boost subtype accuracy on T/NK or myeloid cells.
when_to_use_negative:
- When the goal is annotating tissue-specific cell types (e.g., alveolar cells in
  lung cancers), which scWizard may not identify and may label as the Unknown cluster
- When data are not cancer/TME scRNA-seq — the pretrained model covers 7 major types
  and 47 subtypes within the tumor microenvironment only
- When a simple direct per-cell subtype prediction is sufficient and per-dataset subclustering
  overhead is not justified
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scWizard
  role: Deep neural network model (ReLU + softmax) for major-type prediction and maximum-likelihood
    subtype annotation of each Seurat subcluster in the two-step mode
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Standardized scRNA-seq pipeline (dimension reduction, cell clustering, differential
    expression) used to subcluster each predicted major cell type before subtype annotation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:74d14554edf860036e164aa9594fa5a104a5060c0a0ce1bdf61e8d4148fbd921
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/scWizard
  repo_url: https://github.com/Dulab2020/scWizard
  tools:
  - Seurat
  - scWizard
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hierarchical-cluster-then-annotate-cell-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A two-step scWizard annotation mode in which major cell types are predicted per cell with a deep neural network, each major type is then subclustered with Seurat, and cell subtypes are assigned per subcluster via maximum-likelihood scoring. This improves T/NK and myeloid subtype annotation accuracy versus direct per-cell annotation.

## When to use

Use when annotating tumor-microenvironment cell subtypes in cancer scRNA-seq data and direct per-cell subtype prediction is inaccurate; apply after major-type classification to boost subtype accuracy on T/NK or myeloid cells.

## When NOT to use

- When the goal is annotating tissue-specific cell types (e.g., alveolar cells in lung cancers), which scWizard may not identify and may label as the Unknown cluster
- When data are not cancer/TME scRNA-seq — the pretrained model covers 7 major types and 47 subtypes within the tumor microenvironment only
- When a simple direct per-cell subtype prediction is sufficient and per-dataset subclustering overhead is not justified

## Inputs

- Labeled scRNA-seq benchmark datasets (e.g., LYM, Mye_PAAD, T/NK_BRCA, T/NK_KIRC) split into training and testing sets per the comparison protocol
- scWizard built-in reference/training set (113,976 cells, 7 major cell types, 47 subtypes, H5 format, genes as rows, cells as columns)
- Seurat object or counts matrix (csv/rds/h5) or 10X folder as test data

## Outputs

- Major cell type prediction per cell
- Seurat subclusters for each major cell type
- Per-subcluster subtype annotations via maximum-likelihood scoring
- Per-dataset annotation accuracy and F1 (e.g., scwizard_cluster_mode_accuracy.csv with ~0.89 T/NK_BRCA accuracy, ~0.740 T/NK_BRCA F1)

## How to apply

Train the scWizard deep neural network annotation model (ReLU + softmax activation, one hidden layer whose node count follows the README's sizing criteria, or default parameters) on the shared training set. In cluster mode, first predict the major cell type for every individual cell before clustering, then subcluster each major type using Seurat at a chosen resolution, and finally annotate each subcluster by maximum-likelihood scoring of subtypes. Run direct per-cell annotation on the same test set as the baseline and compare per-cluster accuracy. The article reports scWizard_cluster improved accuracy from 0.64/0.71/0.66 to 0.69/0.89/0.76 on Mye_PAAD, T/NK_BRCA and T/NK_KIRC, respectively, so cluster-mode accuracy should exceed direct-mode on each dataset.

## Related tools

- **scWizard** (Deep neural network model (ReLU + softmax) for major-type prediction and maximum-likelihood subtype annotation of each Seurat subcluster in the two-step mode) — https://github.com/Dulab2020/scWizard
- **Seurat** (Standardized scRNA-seq pipeline (dimension reduction, cell clustering, differential expression) used to subcluster each predicted major cell type before subtype annotation)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Cluster-mode accuracy exceeds direct annotation baseline on each test dataset (e.g., T/NK_BRCA 0.71 → 0.89, T/NK_KIRC 0.66 → 0.76, Mye_PAAD 0.64 → 0.69)
- T/NK_BRCA accuracy falls within tolerance of the reported 0.89 and F1 within tolerance of ~0.740
- Per-cluster classification is consistent with original study labels ('consistent', 'inconsistent', or 'unknown') across all four datasets
- Hidden layer node count satisfies the README criteria: between input and output layer sizes, ~2/3(input)+2/3(output), and less than 2× input size

## Limitations

- Tissue-specific cell types such as alveolar cells in lung cancers may not be identified and may be named in the Unknown cluster
- Identification of tissue-specific subtypes and cellular states is deferred to future updates
- Before annotation, a Python environment must be installed via the app and the trainset (from the Baidu link) copied to the 'app/www/python' folder of the scWizard installation directory
- Results depend on the Seurat subclustering resolution choice, which is a tunable parameter

## Evidence

- [results] direct vs cluster accuracy: [evidence span withheld — anchor status: not_found]
