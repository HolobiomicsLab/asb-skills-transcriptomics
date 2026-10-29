---
name: supervised-classifier-retraining-on-scrna-seq
description: Use when evaluating whether poor reference/training labels—rather than
  classifier architecture—limit malignant-cell annotation accuracy, by retraining
  Cancer-Finder or ikarus on a signature-calibrated .h5ad training set and comparing
  to originally reported metrics.
when_to_use_negative:
- The goal is out-of-the-box malignant-cell annotation rather than a controlled retraining
  ablation — use the pretrained scMalignantFinder model.joblib instead of retraining
  comparators
- The input data lack both malignant and normal epithelial cell annotations (adata.obs['Raw_annotation']
  labels), so supervised retraining cannot be performed or validated
- The task involves spatial transcriptomics region identification or cancer cell state
  analysis rather than scRNA-seq cell-level classification benchmarking
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: Cancer-Finder
  role: Supervised classifier retrained from scratch on the signature-calibrated training
    set; accuracy improved 0.654→0.828, balanced accuracy 0.531→0.669
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: ikarus
  role: Stepwise tumor-vs-normal pipeline whose built-in gene signatures are replaced
    with combined_tumor_up_down_degs.txt before retraining; accuracy improved 0.446→0.488,
    balanced accuracy 0.533→0.612
  repo: https://github.com/BIMSBbioinfo/ikarus
- name: scMalignantFinder
  role: Source of the signature-calibrated training set (combine_training.h5ad) and
    DEG feature list used for retraining both classifiers
  repo: https://github.com/Jonyyqn/scMalignantFinder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-025-07942-y
    title: scMalignantFinder distinguishes malignant cells in single-cell and spatial
      transcriptomics by leveraging cancer signatures
  source_content_hash: sha256:608a86425c2b21284a56c2b15fbb1a3b387b407aefe75c750400936bb344917d
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# supervised-classifier-retraining-on-scrna-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrains existing supervised malignant-cell classifiers (Cancer-Finder and ikarus) from scratch on the scMalignantFinder signature-calibrated training set, using its union-of-DEGs feature list, and benchmarks the retrained models across 13 scRNA-seq test datasets to quantify whether better training labels improve malignant-cell identification.

## When to use

Use when evaluating whether poor reference/training labels—rather than classifier architecture—limit malignant-cell annotation accuracy, by retraining Cancer-Finder or ikarus on a signature-calibrated .h5ad training set and comparing to originally reported metrics.

## When NOT to use

- The goal is out-of-the-box malignant-cell annotation rather than a controlled retraining ablation — use the pretrained scMalignantFinder model.joblib instead of retraining comparators
- The input data lack both malignant and normal epithelial cell annotations (adata.obs['Raw_annotation'] labels), so supervised retraining cannot be performed or validated
- The task involves spatial transcriptomics region identification or cancer cell state analysis rather than scRNA-seq cell-level classification benchmarking

## Inputs

- combine_training.h5ad — signature-calibrated training set with labels in adata.obs['Raw_annotation'] ('Normal'/'Malignant'/'Tumor') from Zenodo record 17888140
- combined_tumor_up_down_degs.txt — combined tumor up/down DEG feature list used in place of classifier-built-in signatures
- 13 benchmark evaluation scRNA-seq test datasets, including test_cancerCellLine.h5ad (cancer cell line malignant cells) and test_TabulaSapiens.h5ad (normal epithelial cells)

## Outputs

- Retrained Cancer-Finder and ikarus models fitted on the calibrated training set with union-of-DEGs features
- Per-cell predicted labels ('Normal'/'Malignant') and malignancy probabilities for each of the 13 test datasets
- retrained_classifier_benchmark.csv — per-dataset and aggregated accuracy/balanced-accuracy comparison of retrained vs originally reported models

## How to apply

Download combine_training.h5ad and combined_tumor_up_down_degs.txt from Zenodo record 17888140 and confirm training labels exist in adata.obs['Raw_annotation'] with values 'Normal', 'Malignant', and 'Tumor'. Retrain Cancer-Finder from scratch on the calibrated training set using the union-of-DEGs feature list (model_method selectable among 'LogisticRegression', 'RandomForest', or 'XGBoost'), applying sc.pp.normalize_total(adata, target_sum=1e4) with no internal log-transformation, since the calibrated set was preprocessed that way in the source study. Retrain ikarus on the same calibrated set, substituting its built-in tumor gene signatures with the combined_tumor_up_down_degs.txt DEG features. Predict per-cell labels ('Normal'/'Malignant') and malignancy probabilities on the same 13 benchmark datasets (including the deposited test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad), compute accuracy and balanced accuracy per dataset, aggregate, and write the retrained-vs-original comparison to retrained_classifier_benchmark.csv. The rationale is that supervised malignant-cell classifiers are 'greatly compromised by the lack of optimal reference datasets', so calibrating labels with nine curated pan-cancer gene signatures before retraining should recover most of the lost performance.

## Related tools

- **Cancer-Finder** (Supervised classifier retrained from scratch on the signature-calibrated training set; accuracy improved 0.654→0.828, balanced accuracy 0.531→0.669) — https://github.com/Patchouli-M/SequencingCancerFinder
- **ikarus** (Stepwise tumor-vs-normal pipeline whose built-in gene signatures are replaced with combined_tumor_up_down_degs.txt before retraining; accuracy improved 0.446→0.488, balanced accuracy 0.533→0.612) — https://github.com/BIMSBbioinfo/ikarus
- **scMalignantFinder** (Source of the signature-calibrated training set (combine_training.h5ad) and DEG feature list used for retraining both classifiers) — https://github.com/Jonyyqn/scMalignantFinder

## Examples

```
import scanpy as sc; sc.pp.normalize_total(adata, target_sum=1e4)  # retrain Cancer-Finder/ikarus on combine_training.h5ad with combined_tumor_up_down_degs.txt features
```

## Evaluation signals

- Retrained Cancer-Finder reproduces average accuracy ≈0.828 and balanced accuracy ≈0.669 across the 13 test datasets
- Retrained ikarus reproduces average accuracy ≈0.488 and balanced accuracy ≈0.612 across the same 13 datasets
- Training labels validated as present in adata.obs['Raw_annotation'] with exactly the values 'Normal', 'Malignant', and 'Tumor'
- Per-dataset metrics computed for all 13 benchmark sets (including test_cancerCellLine.h5ad and test_TabulaSapiens.h5ad) and aggregated, not just a subset
- retrained_classifier_benchmark.csv written with retrained-vs-original metric pairs for both classifiers

## Limitations

- ikarus gains only modestly from retraining (accuracy 0.446→0.488), so the calibrated training set does not rescue all supervised architectures equally
- Cancer-Finder's improvement is driven primarily by enhanced specificity for normal cells, not by improved malignant-cell sensitivity
- The approach inherits scMalignantFinder's scope constraints: it targets malignant cells within cancerous lineages and relies on prerequisite basic cell-type identification, and cancers with unknown or controversial origins (e.g., synovial sarcoma, certain brain tumors) remain untested

## Evidence

- [results] Retraining Cancer-Finder on the calibrated training set improved its average accuracy from 0.654 to 0.828: [evidence span withheld — anchor status: not_found]
- [readme] ikarus is a tumor/normal classifier: [evidence span withheld — anchor status: external_doc]
