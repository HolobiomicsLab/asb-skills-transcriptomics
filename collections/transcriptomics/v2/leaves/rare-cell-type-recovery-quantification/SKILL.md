---
name: rare-cell-type-recovery-quantification
description: Use when testing whether oversampling rare cell types (e.g. CD4T, mast,
  plasma, pDC, cholangiocyte) improves reference-based single-cell annotation accuracy,
  and you need a per-type F1/count comparison table.
when_to_use_negative:
- Cell types in the reference are already balanced, so oversampling cannot change
  minority-class representation
- The query is a disease dataset whose expression profiles deviate strongly from normal
  references, where the SELINA disease mode (no fine-tuning) should be used instead
- A pre-trained SELINA tissue model is already adequate and the goal is annotation
  rather than an SMOTE ablation
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SELINA
  role: Runs preprocess, train (with/without SMOTE cell-type balancing), and predict
    to generate baseline vs SMOTE annotations
  repo: https://github.com/SELINA-team/SELINA.py
- name: Python
  role: Executes the SELINA python package and computes per-class F1 scores and correct-assignment
    counts
  repo: https://github.com/SELINA-team/SELINA.py
- name: R
  role: Supports the SELINA R package for preprocessing (Seurat object) and downstream
    comparison of annotations
  repo: https://github.com/SELINA-team/SELINA.py
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:500286acd73d1e6fa30652527157bd67eb104da40b3355e125b9843e18ada8c4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SELINA-team/SELINA.py
  repo_url: https://github.com/SELINA-team/SELINA.py
  tools:
  - Python
  - R
  - SELINA
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rare-cell-type-recovery-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A SMOTE ablation workflow that quantifies how synthetic minority oversampling in SELINA rescues rare cell-type annotation by comparing baseline vs SMOTE-trained models on liver reference-query data and assembling per-type correct counts and F1 scores.

## When to use

Use when testing whether oversampling rare cell types (e.g. CD4T, mast, plasma, pDC, cholangiocyte) improves reference-based single-cell annotation accuracy, and you need a per-type F1/count comparison table.

## When NOT to use

- Cell types in the reference are already balanced, so oversampling cannot change minority-class representation
- The query is a disease dataset whose expression profiles deviate strongly from normal references, where the SELINA disease mode (no fine-tuning) should be used instead
- A pre-trained SELINA tissue model is already adequate and the goal is annotation rather than an SMOTE ablation

## Inputs

- 4 liver scRNA-seq reference datasets (15,859 cells) with cell-type labels
- 1 liver scRNA-seq query dataset with ground-truth annotations
- Reference expression matrices in SELINA-supported plain, h5, or mtx format

## Outputs

- smote_ablation_f1_table.csv with per-cell-type correct counts and F1 values under baseline and SMOTE conditions
- query_predictions.txt per-cell predicted labels from each model
- query_probability.txt per-cell probabilities for each reference cell type

## How to apply

Preprocess the liver query dataset with 'selina preprocess' using default settings (count-cutoff 1000, gene-cutoff 500, cell-cutoff 10, variable-genes 2000, npcs 30, cluster-res 0.6). Train a baseline SELINA model on the 4 liver reference datasets (15,859 cells) with 'selina train' without cell-type balancing, then train a second model on the same reference data with SMOTE oversampling applied to rare cell types before pre-training. Annotate the query dataset with each model using 'selina predict' with prob-cutoff 0.9 to obtain per-cell labels and probabilities. Compute correct-assignment counts and per-class F1 scores for the rare types (CD4T, plasma, mast, pDC, cholangiocyte, PL, PE) under both conditions and assemble them into a single comparison table (smote_ablation_f1_table.csv) reproducing the Figure 3A-3B trends.

## Related tools

- **SELINA** (Runs preprocess, train (with/without SMOTE cell-type balancing), and predict to generate baseline vs SMOTE annotations) — https://github.com/SELINA-team/SELINA.py
- **Python** (Executes the SELINA python package and computes per-class F1 scores and correct-assignment counts) — https://github.com/SELINA-team/SELINA.py
- **R** (Supports the SELINA R package for preprocessing (Seurat object) and downstream comparison of annotations) — https://github.com/SELINA-team/SELINA.py

## Examples

```
selina train --path-in ./reference_data/ --path-out ./res/pre-train/ --outprefix pre-trained
```

## Evaluation signals

- SMOTE-trained model assigns more query cells to each rare type (CD4T, plasma, mast, pDC, cholangiocyte, PL, PE) than the baseline model
- After SMOTE, roughly one-third of CD4T cells and about half of mast cells are correctly annotated, with large F1 improvements across most rare cell types
- smote_ablation_f1_table.csv contains paired correct counts and F1 values for every rare type under both conditions
- Predictions use prob-cutoff 0.9; each predicted cell type in query_predictions.txt has a corresponding probability in query_probability.txt

## Limitations

- SMOTE oversampling only helps minority cell types; it does not address reference imbalance across tissues, which the authors note is still imbalanced despite the atlas scale
- Disease query data may deviate a lot from normal references; SELINA removed the fine-tuning step for disease prediction, which is not the default setup in this workflow
- Results depend on the liver reference composition (4 datasets, 15,859 cells); recovery fractions may differ for other tissues

## Evidence

- [results] rare cell rescue: [evidence span withheld — anchor status: not_found]
