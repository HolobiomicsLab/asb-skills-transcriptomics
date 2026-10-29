---
name: smote-minority-celltype-oversampling
description: Use when a scRNA-seq reference atlas has imbalanced cell-type abundances
  and rare types (e.g., CD4T, plasma, mast, pDC) are mis-annotated or ignored during
  model training.
when_to_use_negative:
- Reference data cell-type abundances are already balanced, so SMOTE would synthesize
  redundant minority samples without benefit
- Predicting disease-mode query data where SELINA's fine-tuning step is skipped via
  '--disease' and expression profiles deviate substantially from the reference
- The task is batch-effect removal within merged reference data (MADA/LISI evaluation)
  rather than rare-cell-type rescue
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SELINA
  role: Annotation framework whose cell-type balancing step applies SMOTE oversampling
    of rare types before MADA pre-training; invoked via 'selina train' and 'selina
    predict'
  repo: https://github.com/SELINA-team/SELINA.py
- name: Python
  role: Runs the SELINA package and the SMOTE (imbalanced-learn) oversampling during
    training
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:418e97bc27dada0c1330e87a8191181c7a3989f3cf2a6f7aaa484b3506c034dc
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
  - PyTorch
  - Python
  - R
  - SELINA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# smote-minority-celltype-oversampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Oversample rare cell types in single-cell reference data with SMOTE (Synthetic Minority Oversampling Technique) before training SELINA's MADA network, so minority populations are not ignored during modeling and their annotation accuracy is improved.

## When to use

Use when a scRNA-seq reference atlas has imbalanced cell-type abundances and rare types (e.g., CD4T, plasma, mast, pDC) are mis-annotated or ignored during model training.

## When NOT to use

- Reference data cell-type abundances are already balanced, so SMOTE would synthesize redundant minority samples without benefit
- Predicting disease-mode query data where SELINA's fine-tuning step is skipped via '--disease' and expression profiles deviate substantially from the reference
- The task is batch-effect removal within merged reference data (MADA/LISI evaluation) rather than rare-cell-type rescue

## Inputs

- Paired reference expression/annotation files (xx_expr.txt gene-by-cell matrices and xx_meta.txt cell-type labels), e.g. 4 liver reference datasets with 15,859 cells
- Preprocessed query expression matrix (query_expr.txt from 'selina preprocess')

## Outputs

- SMOTE-balanced reference training set feeding 'selina train' and its pre-trained_params.pt / pre-trained_meta.pkl model files
- Per-cell predictions and probabilities from 'selina predict' (prob-cutoff 0.9) for both SMOTE and baseline models
- Comparison table of per-cell-type correct counts and F1 scores (e.g. smote_ablation_f1_table.csv) for rare types (CD4T, plasma, mast, pDC, cholangiocyte, PL, PE)

## How to apply

Preprocess the query data with 'selina preprocess' using the tool defaults (count-cutoff 1000, gene-cutoff 500, cell-cutoff 10, variable-genes 2000, npcs 30, cluster-res 0.6). Balance the reference data by oversampling minority cell types with SMOTE as the first of SELINA's three main steps (cell type balancing, pre-training, fine-tuning), before pre-training the multi-adversarial domain adaptation network on the four liver reference datasets (15,859 cells). Run 'selina train' twice — once with SMOTE balancing and once as a baseline without it — then annotate the query dataset with each model via 'selina predict' using prob-cutoff 0.9. Compare per-class F1 scores and correct-assignment counts for the rare cell types (CD4T, plasma, mast, pDC, cholangiocyte, PL, PE) between the two runs to confirm the SMOTE-trained model recovers more rare cells.

## Related tools

- **SELINA** (Annotation framework whose cell-type balancing step applies SMOTE oversampling of rare types before MADA pre-training; invoked via 'selina train' and 'selina predict') — https://github.com/SELINA-team/SELINA.py
- **Python** (Runs the SELINA package and the SMOTE (imbalanced-learn) oversampling during training)

## Examples

```
selina train --path-in ./reference_data/ --path-out ./res/pre-train/ --outprefix pre-trained
```

## Evaluation signals

- SMOTE-trained model assigns more query cells to each rare cell type than the baseline model (e.g. roughly one-third of CD4T cells and about half of mast cells correctly annotated after SMOTE)
- Per-class F1 scores for rare cell types (CD4T, plasma, mast, pDC, cholangiocyte, PL, PE) increase relative to the no-SMOTE baseline
- Training script completes ('Finish Training') and produces pre-trained_params.pt plus pre-trained_meta.pkl containing the reference cell types and genes
- Comparison table (e.g. smote_ablation_f1_table.csv) contains correct-assignment counts and F1 values for both SMOTE and baseline runs, reproducing the Figure 3A-3B trends

## Limitations

- Cell numbers across different cell types are often imbalanced, so minority cell types are always ignored in the modeling process unless oversampled
- The imbalanced-dataset problem persists at atlas scale: the number of datasets in different tissues and the cell-type abundance in each tissue is still imbalanced despite SELINA's 1.7-million-cell reference
- SMOTE gains are tissue/reference dependent; the reported recovery was demonstrated on liver reference-query data (4 reference datasets, 15,859 cells)

## Evidence

- [results] SMOTE oversampling improved rare cell-type annotation: [evidence span withheld — anchor status: not_found]
