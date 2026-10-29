---
name: malignant-cell-classification-benchmarking
description: Use when quantifying whether swapping a classifier's default gene signatures/training
  data for a signature-calibrated reference improves malignant-vs-normal cell identification
  accuracy on benchmark scRNA-seq datasets.
when_to_use_negative:
- The goal is malignant-region identification in spatial transcriptomics rather than
  per-cell classification of scRNA-seq data
- No labeled training data are available — retraining requires adata.obs['Raw_annotation']
  labels to be present
- The question requires de novo cancer-cell-state or CNV-inference analysis (use sc_malignant_deg.gmt
  / Malignant_MPs.Gavish_2023.gmt or CopyKAT/inferCNVpy) rather than benchmarking
  supervised classifiers
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Cancer-Finder
  role: Supervised malignant-cell classifier retrained from scratch on the calibrated
    training set with selectable model_method ('LogisticRegression', 'RandomForest',
    'XGBoost')
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: ikarus
  role: Stepwise ML pipeline retrained on the calibrated training set with its built-in
    gene signatures substituted by combined_tumor_up_down_degs.txt features
  repo: https://github.com/BIMSBbioinfo/ikarus
- name: scMalignantFinder
  role: Source of the signature-calibrated training set, DEG feature list, and benchmark
    test datasets on Zenodo record 17888140
  repo: https://github.com/Jonyyqn/scMalignantFinder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:436e35b005ef4ebbf780f87bd501b9f7a83c1ee6fda9eb44b75f9bb2ac680d64
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Jonyyqn/scMalignantFinder
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - ikarus
  - scMalignantFinder
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# malignant-cell-classification-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrains existing supervised malignant-cell classifiers (Cancer-Finder and ikarus) on the scMalignantFinder signature-calibrated training set (combine_training.h5ad from Zenodo 17888140) and benchmarks the retrained models against originally reported metrics across 13 scRNA-seq test datasets.

## When to use

Use when quantifying whether swapping a classifier's default gene signatures/training data for a signature-calibrated reference improves malignant-vs-normal cell identification accuracy on benchmark scRNA-seq datasets.

## When NOT to use

- The goal is malignant-region identification in spatial transcriptomics rather than per-cell classification of scRNA-seq data
- No labeled training data are available — retraining requires adata.obs['Raw_annotation'] labels to be present
- The question requires de novo cancer-cell-state or CNV-inference analysis (use sc_malignant_deg.gmt / Malignant_MPs.Gavish_2023.gmt or CopyKAT/inferCNVpy) rather than benchmarking supervised classifiers

## Inputs

- combine_training.h5ad — signature-calibrated training set with adata.obs['Raw_annotation'] labels ('Normal', 'Malignant', 'Tumor') from Zenodo record 17888140
- combined_tumor_up_down_degs.txt — union-of-DEGs feature list (tumor up/down regulated genes)
- 13 benchmark evaluation scRNA-seq datasets, including test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad

## Outputs

- retrained_classifier_benchmark.csv — per-dataset and aggregate accuracy and balanced accuracy for retrained vs originally reported models
- Per-cell predicted labels ('Normal'/'Malignant') and malignancy probabilities for each of the 13 test datasets
- Retrained Cancer-Finder and ikarus model objects

## How to apply

Download the signature-calibrated training set combine_training.h5ad and the union-of-DEGs feature list combined_tumor_up_down_degs.txt from Zenodo record 17888140, confirming training labels in adata.obs['Raw_annotation'] with values 'Normal', 'Malignant', and 'Tumor'. Retrain Cancer-Finder from scratch on this calibrated set using the DEG feature list (model_method among 'LogisticRegression', 'RandomForest', or 'XGBoost'), applying sc.pp.normalize_total(adata, target_sum=1e4) with no internal log-transformation. Retrain ikarus on the same data, substituting its built-in gene signatures with the DEG features. Run prediction on the same 13 benchmark evaluation scRNA-seq datasets (including test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad), collect per-cell predicted labels ('Normal'/'Malignant') and malignancy probabilities, then compute per-dataset accuracy and balanced accuracy and aggregate. The rationale is that the calibrated training set (malignant cells refined using nine curated cancer gene signatures) and union-of-DEGs features correct the reference-data limitations that compromise existing supervised classifiers.

## Related tools

- **Cancer-Finder** (Supervised malignant-cell classifier retrained from scratch on the calibrated training set with selectable model_method ('LogisticRegression', 'RandomForest', 'XGBoost')) — https://github.com/Patchouli-M/SequencingCancerFinder
- **ikarus** (Stepwise ML pipeline retrained on the calibrated training set with its built-in gene signatures substituted by combined_tumor_up_down_degs.txt features) — https://github.com/BIMSBbioinfo/ikarus
- **scMalignantFinder** (Source of the signature-calibrated training set, DEG feature list, and benchmark test datasets on Zenodo record 17888140) — https://github.com/Jonyyqn/scMalignantFinder

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('combine_training.h5ad'); sc.pp.normalize_total(adata, target_sum=1e4)
```

## Evaluation signals

- Retrained Cancer-Finder reproduces accuracy 0.828 and balanced accuracy 0.669 on the 13 benchmark datasets (vs originally reported 0.654 and 0.531)
- Retrained ikarus reproduces accuracy 0.488 and balanced accuracy 0.612 on the same 13 datasets (vs originally reported 0.446 and 0.533)
- Improvements are driven primarily by enhanced specificity for identifying normal cells, not by loss of malignant-cell sensitivity
- Training labels in adata.obs['Raw_annotation'] contain exactly the values 'Normal', 'Malignant', and 'Tumor'
- retrained_classifier_benchmark.csv contains both retrained and originally reported metrics for every one of the 13 datasets

## Limitations

- scMalignantFinder-derived resources are designed to identify malignant cells from cancerous lineages, relying on the prerequisite identification of basic cell types
- Capabilities are untested for cancers with unknown or controversial origins, such as synovial sarcoma and certain brain tumors
- Relying solely on inferred CNVs has limitations in annotating malignant cells, especially in cancers with minimal genomic structural variation or unknown origins

## Evidence

- [readme] ikarus pipeline description: [evidence span withheld — anchor status: external_doc]
