---
name: consistent-inconsistent-cell-labeling
description: Use when benchmarking a cell-type annotation method against original
  study labels on labeled scRNA-seq test sets, and you need per-cluster accuracy plus
  a consistent/inconsistent/unknown triage of each prediction.
when_to_use_negative:
- Input data lacks original study cell labels, making the consistent/inconsistent/unknown
  comparison against a gold standard impossible
- The goal is annotating tissue-specific cell types (e.g., alveolar cells in lung
  cancers), which scWizard may not identify and may assign to the Unknown cluster
- You need a fully novel subtype discovery rather than evaluating agreement with existing
  labels — this skill only measures concordance with predefined labels
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scWizard
  role: Deep neural network annotation model (ReLU + softmax) run in direct per-cell
    mode and in two-step scWizard_cluster mode to produce the predictions being scored
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Subclustering of each predicted major cell type at a specific resolution inside
    the two-step scWizard_cluster workflow before maximum-likelihood subtype scoring
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:177ff96ccf4e951e295057f7802c59c5f3a0f9f14d0e462e02afda35ef1cec2c
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# consistent-inconsistent-cell-labeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmark evaluation scheme from the scWizard paper that classifies each predicted cell annotation as 'consistent', 'inconsistent', or 'unknown' against original study labels, then computes per-cluster accuracy (and F1) to quantify whether the two-step scWizard_cluster mode improves subtype annotation over direct per-cell annotation.

## When to use

Use when benchmarking a cell-type annotation method against original study labels on labeled scRNA-seq test sets, and you need per-cluster accuracy plus a consistent/inconsistent/unknown triage of each prediction.

## When NOT to use

- Input data lacks original study cell labels, making the consistent/inconsistent/unknown comparison against a gold standard impossible
- The goal is annotating tissue-specific cell types (e.g., alveolar cells in lung cancers), which scWizard may not identify and may assign to the Unknown cluster
- You need a fully novel subtype discovery rather than evaluating agreement with existing labels — this skill only measures concordance with predefined labels

## Inputs

- Labeled scRNA-seq benchmark datasets with original study cell labels (LYM, Mye_PAAD, T/NK_BRCA, T/NK_KIRC; metadata in Table S1)
- Trained scWizard deep neural network annotation model (trainset in H5 format, rows = genes, columns = cells)
- Seurat-object or counts-matrix test-set expression data (csv/rds/h5 or 10X folder)

## Outputs

- Per-cell prediction classes: 'consistent', 'inconsistent', or 'unknown' versus original study labels
- Per-dataset (per-cluster) accuracy values for direct vs. two-step scWizard_cluster annotation (target ~0.89 T/NK_BRCA, ~0.76 T/NK_KIRC, ~0.69 Mye_PAAD, >0.66 LYM)
- F1 score on T/NK_BRCA (~0.740)
- scwizard_cluster_mode_accuracy.csv summary table

## How to apply

Split each labeled benchmark dataset (e.g., LYM, Mye_PAAD, T/NK_BRCA, T/NK_KIRC from Table S1) into training and testing sets per the comparison protocol and train the scWizard deep neural network model (ReLU + softmax activation, adjustable single hidden layer, default parameters otherwise). Run the trained model in two modes: direct per-cell subtype prediction (baseline) and the 'two-step' scWizard_cluster mode, where major cell types are predicted per cell, each major type is subclustered with Seurat at a chosen resolution, and each subcluster receives a subtype label by maximum-likelihood scoring. For every predicted cell, compare the predicted label to the original study label and assign 'consistent' (match), 'inconsistent' (mismatch), or 'unknown' (prediction falls outside the labeled categories). Aggregate these into per-cluster (per-dataset) accuracy — target values ~0.89 for T/NK_BRCA, ~0.76 for T/NK_KIRC, ~0.69 for Mye_PAAD, >0.66 for LYM — plus F1 (~0.740 on T/NK_BRCA), and write results to scwizard_cluster_mode_accuracy.csv. The skill is applied correctly when the two-step mode's accuracy exceeds the direct-annotation baseline on every dataset.

## Related tools

- **scWizard** (Deep neural network annotation model (ReLU + softmax) run in direct per-cell mode and in two-step scWizard_cluster mode to produce the predictions being scored) — https://github.com/Dulab2020/scWizard
- **Seurat** (Subclustering of each predicted major cell type at a specific resolution inside the two-step scWizard_cluster workflow before maximum-likelihood subtype scoring)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Two-step scWizard_cluster accuracy exceeds the direct per-cell annotation baseline on each of the four datasets (e.g., 0.69 vs 0.64 on Mye_PAAD, 0.89 vs 0.71 on T/NK_BRCA, 0.76 vs 0.66 on T/NK_KIRC)
- T/NK_BRCA accuracy and F1 fall within tolerance of the reported 0.89 and 0.740
- Every predicted cell receives exactly one of the labels 'consistent', 'inconsistent', or 'unknown' with no missing classifications
- Output CSV (scwizard_cluster_mode_accuracy.csv) contains per-dataset accuracies for all four datasets plus the T/NK_BRCA F1
- Tenfold cross-validation on training data reproduces the model's reported accuracy ranges (0.98 major types, 0.85 myeloid subtypes, 0.79 T/NK subtypes)

## Limitations

- Tissue-specific cell types may not be identified and are named in Unknown cluster, inflating the 'unknown' fraction of predictions
- Accuracy depends on subclustering resolution and hidden-layer node configuration, which are user-adjustable and can shift results
- Performance is specific to TME cell types covered by the 113,976-cell reference (7 major types, 47 subtypes across 13 cancer types); out-of-scope subtypes cannot be scored as consistent

## Evidence
