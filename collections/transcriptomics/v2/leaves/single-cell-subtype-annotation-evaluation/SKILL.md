---
name: single-cell-subtype-annotation-evaluation
description: Use when benchmarking deep-learning cell annotation on tumor microenvironment
  scRNA-seq data, specifically to test whether major-type prediction followed by Seurat
  subclustering improves T/NK or myeloid subtype accuracy over direct per-cell annotation.
when_to_use_negative:
- Input data is unlabeled or lacks original study cell labels — the 'consistent/inconsistent/unknown'
  accuracy evaluation against ground truth cannot be computed
- The goal is annotating tissue-specific cell types (e.g., alveolar cells in lung
  cancers), which scWizard may not identify and will assign to the Unknown cluster
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scWizard
  role: Deep neural network (ReLU + softmax) hierarchical model for major cell type
    and T/NK / myeloid subtype annotation; run in both two-step cluster mode and direct
    per-cell mode
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Subclustering of each predicted major cell type before maximum-likelihood
    subtype assignment in the two-step mode
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:b6a264ad4de5464dabc46ffec624447cb8a2755eeeabf7a3079cd7531272ab20
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-subtype-annotation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces scWizard's two-step (annotate-after-clustering) versus direct per-cell subtype annotation benchmark on T/NK and myeloid scRNA-seq test sets, quantifying accuracy gains from Seurat subclustering before subtype assignment.

## When to use

Use when benchmarking deep-learning cell annotation on tumor microenvironment scRNA-seq data, specifically to test whether major-type prediction followed by Seurat subclustering improves T/NK or myeloid subtype accuracy over direct per-cell annotation.

## When NOT to use

- Input data is unlabeled or lacks original study cell labels — the 'consistent/inconsistent/unknown' accuracy evaluation against ground truth cannot be computed
- The goal is annotating tissue-specific cell types (e.g., alveolar cells in lung cancers), which scWizard may not identify and will assign to the Unknown cluster

## Inputs

- Labeled scRNA-seq benchmark datasets (LYM, Mye_PAAD, T/NK_BRCA, T/NK_KIRC; Table S1) split into training/testing sets
- scWizard training data (H5 format, rows = genes, columns = cells) plus matching label file, e.g. from the Baidu Netdisk trainset link
- Seurat object or counts matrix (csv/rds/h5) or 10X folder for the test set

## Outputs

- Per-dataset two-step vs direct annotation accuracies (expected: T/NK_BRCA ~0.89 vs 0.71, T/NK_KIRC ~0.76 vs 0.66, Mye_PAAD ~0.69 vs 0.64)
- T/NK_BRCA F1 score (~0.740)
- Per-cell classification as 'consistent', 'inconsistent', or 'unknown' vs original labels
- scwizard_cluster_mode_accuracy.csv compiling per-dataset accuracies

## How to apply

Load the four labeled benchmark datasets (LYM, Mye_PAAD, T/NK_BRCA, T/NK_KIRC from Table S1) and split each into training and testing sets per the comparison protocol. Train the scWizard deep neural network annotation model (ReLU + softmax activation, one hidden layer whose node count is adjustable, default parameters) on the shared training sets for T/NK and myeloid subtypes; scWizard's hierarchical reference covers 7 major cell types and 47 subtypes from 113,976 cells across 13 cancer types. Run the 'two-step' mode: predict the major cell type per cell, subcluster each major type with Seurat at a chosen resolution, then assign subtypes per subcluster by maximum-likelihood scoring; run the same trained model in direct per-cell annotation mode as the baseline. Classify predictions as 'consistent', 'inconsistent', or 'unknown' against original study labels and compute per-cluster accuracy plus F1 (on T/NK_BRCA), then compile results into scwizard_cluster_mode_accuracy.csv.

## Related tools

- **scWizard** (Deep neural network (ReLU + softmax) hierarchical model for major cell type and T/NK / myeloid subtype annotation; run in both two-step cluster mode and direct per-cell mode) — https://github.com/Dulab2020/scWizard
- **Seurat** (Subclustering of each predicted major cell type before maximum-likelihood subtype assignment in the two-step mode)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Two-step scWizard_cluster accuracy exceeds the direct annotation baseline on every dataset (e.g., T/NK_BRCA 0.89 > 0.71, T/NK_KIRC 0.76 > 0.66, Mye_PAAD 0.69 > 0.64)
- T/NK_BRCA accuracy falls within tolerance of the reported 0.89 and F1 of ~0.740
- LYM dataset accuracy exceeds the ~0.66 reported threshold
- Every predicted cell maps to one of 'consistent', 'inconsistent', or 'unknown' relative to original study labels; output CSV schema matches scwizard_cluster_mode_accuracy.csv
- Direct-annotation baseline accuracies reproduce the reported values (0.64 Mye_PAAD, 0.71 T/NK_BRCA, 0.66 T/NK_KIRC)

## Limitations

- Tissue-specific cell types (e.g., alveolar cells in lung cancers) may not be identified and are named in the Unknown cluster; identification of tissue-specific subtypes and cellular states is deferred to future updates
- The model requires downloading the pre-trained trainset (H5) into the app/www/python folder and installing the Python environment before annotation; users need jags installed (R >= 4.0.2)
- Hidden layer node count must follow the stated criteria (between input and output layer sizes, ~2/3 input + 2/3 output, or < 2x input) or defaults, otherwise model configuration may be suboptimal

## Evidence
