---
name: k-fold-cross-validation-evaluation
description: Use when a hierarchical cell-annotation classifier must be trained and
  validated on a labeled scRNA-seq reference (H5 matrix, genes in rows, cells in columns)
  and per-fold accuracy at major and subtype levels is required.
when_to_use_negative:
- Tissue-specific cell types are the target — scWizard may not identify them, e.g.
  alveolar cells in lung cancers can land in the Unknown cluster
- The goal is unbiased generalization on an independent cohort rather than training-set
  reliability — use an external labeled dataset (e.g. the MCC 11,024-cell dataset)
  for that
- The annotation model itself is not being evaluated — e.g. only downstream analyses
  (GSVA, inferCNV, Monocle, CellphoneDB) are needed
tools:
- name: scWizard
  role: Provides the hierarchical deep neural network classifier (ReLU/softmax; 1
    hidden layer for major types, 3 for subtypes) and the training data plumbing that
    is cross-validated
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Performs subclustering of each predicted major cell type prior to subtype-level
    annotation in the two-step scheme
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:fa0d723c5e6f32d7f03f282917d752801f42a0037ba6360998e53a1db1155541
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# k-fold-cross-validation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates scWizard's hierarchical deep neural network cell-annotation model by 10-fold cross-validation on its built-in reference (113,976 cells, 7 major types, 47 subtypes, 199 samples, 13 cancer types), estimating concordance at both major and subtype levels before deployment on new data.

## When to use

Use when a hierarchical cell-annotation classifier must be trained and validated on a labeled scRNA-seq reference (H5 matrix, genes in rows, cells in columns) and per-fold accuracy at major and subtype levels is required.

## When NOT to use

- Tissue-specific cell types are the target — scWizard may not identify them, e.g. alveolar cells in lung cancers can land in the Unknown cluster
- The goal is unbiased generalization on an independent cohort rather than training-set reliability — use an external labeled dataset (e.g. the MCC 11,024-cell dataset) for that
- The annotation model itself is not being evaluated — e.g. only downstream analyses (GSVA, inferCNV, Monocle, CellphoneDB) are needed

## Inputs

- scWizard training reference H5 file (113,976 cells; genes as rows, cells as columns) from https://pan.baidu.com/s/1_J_17OKRLo4BQB33C_Ux3Q
- Cell-type labels for major types and curated stromal subtypes (canonical-marker-based)

## Outputs

- cv_accuracy.csv with per-fold and average major-level and subtype-level accuracy
- Flags for folds deviating strongly from the average accuracy

## How to apply

Download the scWizard training reference (113,976 cells from 10 datasets, 199 samples, 13 cancer types; H5 format with genes as rows and cells as columns) and use its provided labels: major cell types (endothelial, myeloid, T/NK, stromal, etc.) plus manually curated stromal subtypes based on canonical markers. Build the two-step hierarchical classifier — a deep neural network with ReLU and softmax activations, one hidden layer for major cell types and three hidden layers for subtypes, the output layer sized to the number of cell types. Subtype annotation is applied after Seurat subclustering of each predicted major cell type, mirroring scWizard's 'two-step' design. Divide the reference into ten parts, train on nine parts and test on the held-out part in turn, computing per-fold accuracy as the fraction of cells whose predicted label is 'consistent' with the original study's annotation, then average the ten folds. Write per-fold and average major- and subtype-level metrics to cv_accuracy.csv, flagging any fold whose accuracy deviates strongly from the mean.

## Related tools

- **scWizard** (Provides the hierarchical deep neural network classifier (ReLU/softmax; 1 hidden layer for major types, 3 for subtypes) and the training data plumbing that is cross-validated) — https://github.com/Dulab2020/scWizard
- **Seurat** (Performs subclustering of each predicted major cell type prior to subtype-level annotation in the two-step scheme)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Average ten-fold concordance for major cell types should approximate the reported high value (~0.97–0.98)
- Subtype-level accuracy should fall near the reported ranges (e.g. myeloid ~0.85, T/NK ~0.79; per the card, 0.81–0.89 on the training set)
- Per-fold accuracies are recorded individually in cv_accuracy.csv and any fold deviating strongly from the ten-fold average is flagged
- Per-fold accuracy is computed as the fraction of cells whose predicted label matches ('consistent') the original study's annotation
- The H5 training matrix is correctly oriented: rows represent genes and columns represent cells

## Limitations

- Tissue-specific cell types may not be identified and are named in the Unknown cluster (e.g. alveolar cells in lung cancers)
- Identification of tissue-specific subtypes and cellular states is deferred to future updates
- Accuracy is measured against original study annotations, so reference-label quality bounds the measured concordance
- Tissue-specific subtypes and cellular states: scWizard will be updated and improved to identify more cell types including tissue specific subtypes and cellular states in the future

## Evidence
