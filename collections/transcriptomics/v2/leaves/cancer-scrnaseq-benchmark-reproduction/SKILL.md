---
name: cancer-scrnaseq-benchmark-reproduction
description: Use when a labeled cancer scRNA-seq test set (e.g. T/NK_BRCA, T/NK_KIRC,
  Mye_PAAD, LYM) must be annotated with a pretrained hierarchical DNN and you need
  to reproduce or compare cluster-after-annotate vs direct annotation accuracy.
when_to_use_negative:
- Input is raw, unlabeled scRNA-seq with no training/test split or reference labels
  — this skill is a benchmark reproduction, not a discovery annotation run
- Tissue-specific cell types such as alveolar cells in lung cancers, which scWizard
  cannot identify and reports as Unknown
- Major cell type annotation only (accuracy 0.98 benchmark) rather than T/NK or myeloid
  subtype annotation, where the two-step mode's advantage was demonstrated
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scWizard
  role: Provides the hierarchical deep neural network annotation model (ReLU + softmax)
    run in direct and two-step scWizard_cluster modes for benchmark comparison
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Subclusters each predicted major cell type at a specific resolution inside
    the scWizard_cluster two-step annotation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:94293ebafce18ef8f31322852a6c6e3b1a7696fa8903aae2f8204e4f99d8a19a
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
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cancer-scrnaseq-benchmark-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the scWizard 'two-step' annotation benchmark: per-cell major-type prediction followed by Seurat subclustering and maximum-likelihood subtype annotation (scWizard_cluster), compared against direct per-cell annotation on T/NK and myeloid scRNA-seq test sets.

## When to use

Use when a labeled cancer scRNA-seq test set (e.g. T/NK_BRCA, T/NK_KIRC, Mye_PAAD, LYM) must be annotated with a pretrained hierarchical DNN and you need to reproduce or compare cluster-after-annotate vs direct annotation accuracy.

## When NOT to use

- Input is raw, unlabeled scRNA-seq with no training/test split or reference labels — this skill is a benchmark reproduction, not a discovery annotation run
- Tissue-specific cell types such as alveolar cells in lung cancers, which scWizard cannot identify and reports as Unknown
- Major cell type annotation only (accuracy 0.98 benchmark) rather than T/NK or myeloid subtype annotation, where the two-step mode's advantage was demonstrated

## Inputs

- Labeled scRNA-seq benchmark datasets: LYM, Mye_PAAD, T/NK_BRCA, T/NK_KIRC (Table S1), split into training/testing sets
- scWizard pretrained reference trainset (H5 format, genes as rows, cells as columns; 113,976 cells, 7 major types, 47 subtypes)
- Counts matrix or Seurat object input (csv, rds, h5, or 10X folder)

## Outputs

- scwizard_cluster_mode_accuracy.csv with per-dataset accuracies (~0.89 T/NK_BRCA, ~0.76 T/NK_KIRC, ~0.69 Mye_PAAD, >0.66 LYM)
- Per-cluster 'consistent'/'inconsistent'/'unknown' label assignments per cell
- T/NK_BRCA F1 score (~0.740) and direct-annotation baseline accuracies (0.64, 0.71, 0.66)

## How to apply

Split each labeled benchmark dataset (LYM, Mye_PAAD, T/NK_BRCA, T/NK_KIRC from Table S1) into training and testing sets per the comparison protocol, then train the scWizard deep neural network (ReLU + softmax activation, one adjustable hidden layer, default parameters) on the shared training data for T/NK and myeloid subtypes. Run the trained model in two modes on each test set: direct per-cell subtype prediction (baseline) and the 'two-step' scWizard_cluster mode — major cell type prediction per cell, Seurat subclustering of each major type at a chosen resolution, then maximum-likelihood subtype scoring of each subcluster. Classify each predicted cell as 'consistent', 'inconsistent', or 'unknown' against original study labels and compute per-cluster accuracy for all four datasets plus F1 on T/NK_BRCA. The rationale is that clustering aggregates signal across cells before subtype scoring, recovering the hierarchical relationships between major types and subtypes the article argues direct classifiers overlook.

## Related tools

- **scWizard** (Provides the hierarchical deep neural network annotation model (ReLU + softmax) run in direct and two-step scWizard_cluster modes for benchmark comparison) — https://github.com/Dulab2020/scWizard
- **Seurat** (Subclusters each predicted major cell type at a specific resolution inside the scWizard_cluster two-step annotation)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- scWizard_cluster accuracy exceeds direct annotation baseline on every dataset (0.69 vs 0.64 Mye_PAAD, 0.89 vs 0.71 T/NK_BRCA, 0.76 vs 0.66 T/NK_KIRC)
- T/NK_BRCA accuracy ~0.89 and F1 ~0.740 reproduced within tolerance
- Per-cluster accuracies on all four datasets (including LYM >0.66) present in scwizard_cluster_mode_accuracy.csv
- Every predicted cell maps to one of 'consistent', 'inconsistent', or 'unknown' against original study labels
- Trained model uses ReLU + softmax with hidden-layer size within the documented criteria (between input and output layer sizes)

## Limitations

- Tissue-specific cell types (e.g. alveolar cells in lung cancers) may not be identified and are named in an Unknown cluster
- Identification of tissue-specific subtypes and cellular states is deferred to future updates
- Requires Python environment installation and the trainset copied to the app/www/python folder before the annotation function can run

## Evidence
