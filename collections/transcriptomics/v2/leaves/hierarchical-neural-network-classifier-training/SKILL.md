---
name: hierarchical-neural-network-classifier-training
description: Use when building or reproducing automated cancer scRNA-seq cell-type
  annotation with a hierarchical deep-learning classifier, or when benchmarking major-type
  vs subtype annotation accuracy via cross-validation on the scWizard training reference.
when_to_use_negative:
- Tissue-specific cell types (e.g., alveolar cells in lung cancers) are the annotation
  target — the scWizard reference may not identify them and they fall into the Unknown
  cluster
- Input data is not cancer scRNA-seq / tumor microenvironment cells, since the reference
  covers 13 cancer types and 7 TME major types
- A labeled reference and supervised training are not possible and marker-based or
  correlation-based annotation tools are preferred
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scWizard
  role: R package providing the deep neural network (ReLU + softmax) training, the
    two-step hierarchical annotation model, and the tenfold validation test; installed
    via devtools::install_github('Dulab2020/scWizard')
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Used for subclustering of each predicted major cell type before subtype annotation
    in the second step of the hierarchical model
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:e884de2f4c144fcda82792db1b6368fec1b0227ad343fb040d13dba720115fc1
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# hierarchical-neural-network-classifier-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and cross-validate scWizard's two-step hierarchical deep neural network (ReLU + softmax) that classifies tumor microenvironment cells into 7 major types and 47 subtypes using a built-in reference of 113,976 cells across 13 cancer types, and estimate its accuracy via ten-fold validation.

## When to use

Use when building or reproducing automated cancer scRNA-seq cell-type annotation with a hierarchical deep-learning classifier, or when benchmarking major-type vs subtype annotation accuracy via cross-validation on the scWizard training reference.

## When NOT to use

- Tissue-specific cell types (e.g., alveolar cells in lung cancers) are the annotation target — the scWizard reference may not identify them and they fall into the Unknown cluster
- Input data is not cancer scRNA-seq / tumor microenvironment cells, since the reference covers 13 cancer types and 7 TME major types
- A labeled reference and supervised training are not possible and marker-based or correlation-based annotation tools are preferred

## Inputs

- scWizard training reference dataset (113,976 cells, 199 samples, 13 cancer types) in H5 format, genes as rows and cells as columns (https://pan.baidu.com/s/1_J_17OKRLo4BQB33C_Ux3Q, extraction code 0ccq)
- Cell-type labels: 7 major cell types (endothelial, myeloid, T/NK, stromal, etc.) and manually curated 47 stromal/immune subtypes based on canonical markers
- Optional custom training set and label file via scWizard's 'Choose custom trainset' / 'Choose custom label for trainset'

## Outputs

- Per-fold and average ten-fold cross-validation accuracies for major cell type and subtype annotation (e.g., cv_accuracy.csv)
- Trained two-step hierarchical deep neural network (one hidden layer for major types, three for subtypes) usable for cell annotation in scWizard
- Flagged folds whose accuracy deviates strongly from the average

## How to apply

Download the scWizard training reference (113,976 cells, 199 samples, 13 cancer types, H5 format with genes as rows and cells as columns) from the pan.baidu.com link (extraction code 0ccq) and install the R package from GitHub. Build the 'two-step' annotation model: individual cells are classified into major cell types before clustering, and subtype annotation is applied after Seurat subclustering of each predicted major type. The network uses ReLU and softmax activations, one input layer sized to the number of genes after PCA, one hidden layer for major cell type annotation, three hidden layers for subtype annotation, and an output layer sized to the number of cell types; hidden-layer node counts can be tuned per the README criteria (e.g., between input and output size, or 2/3 input + 2/3 output). For the tenfold validation test, divide the reference into ten parts, train on nine parts and test on the remaining part in turn. Per-fold accuracy is the fraction of cells whose predicted label matches the original study's annotation ('consistent'); average the ten folds, expecting ~0.97-0.98 concordance for major types and ~0.85 (myeloid) / ~0.79 (T/NK) for subtypes.

## Related tools

- **scWizard** (R package providing the deep neural network (ReLU + softmax) training, the two-step hierarchical annotation model, and the tenfold validation test; installed via devtools::install_github('Dulab2020/scWizard')) — https://github.com/Dulab2020/scWizard
- **Seurat** (Used for subclustering of each predicted major cell type before subtype annotation in the second step of the hierarchical model)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/scWizard")
```

## Evaluation signals

- Average ten-fold cross-validation concordance for major cell types approximates the reported ~0.97-0.98
- Subtype-level accuracies fall in the reported ranges: ~0.85 for myeloid subtypes and ~0.79 for T/NK subtypes
- Per-fold accuracy is computed as the fraction of cells whose predicted label matches the original study's annotation ('consistent'), and per-fold values are written per level
- Model architecture matches the paper: ReLU and softmax activations, 1 hidden layer for major type annotation, 3 hidden layers for subtype annotation, output layer sized to the number of cell types
- Training data loads as H5 with rows as genes and columns as cells, and the Python environment is installed before running annotation

## Limitations

- Tissue-specific cell types (e.g., alveolar cells in lung cancers) may not be identified and are named in the Unknown cluster
- Identification of tissue-specific subtypes and cellular states is deferred to future updates of scWizard
- The neural network classifier requires a Python environment (install via the 'install Python' button) and the trainset copied to the app/www/python folder before annotation can run

## Evidence
