---
name: multi-dataset-scrna-seq-reference-merging
description: Use when multiple scRNA-seq reference datasets from different sequencing
  platforms must be merged and batch-corrected (e.g. five liver datasets with *_expr.txt
  and *_meta.txt files), or when reference-query batch effects must be quantified
  before label transfer.
when_to_use_negative:
- Input is a single reference dataset with no platform or batch structure to remove
  — SMOTE/MADA pre-training adds no value
- Disease query data where fine-tuning should be skipped, since SELINA removes the
  fine-tuning step for disease mode (--disease flag) because disease expression profiles
  deviate greatly from normal data
- The task is cell-type annotation only with a pre-built reference atlas, not batch-effect
  evaluation of a merged reference
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SELINA
  role: 'Pre-trains the MADA network on the merged reference (with SMOTE oversampling
    of rare cell types) and fine-tunes with an autoencoder (Fine-tuning1: 50 epochs,
    Fine-tuning2: 10 epochs) to remove batch effects'
  repo: https://github.com/SELINA-team/SELINA.py
- name: Harmony
  role: Benchmark integration method run on the same merged liver datasets to compute
    matching LISI scores for comparison (Figures 3D-3F)
  repo: https://github.com/immunogenomics/harmony
- name: Python
  role: Used to merge datasets, run SELINA, and compute LISI scores and UMAP plots
- name: R
  role: Used with the R package stack (Seurat, data.table, optparse) bundled in the
    SELINA conda environment and for Harmony comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:587b8e52cdaddc8b59d2b514d35b7dc03ce75f5aa351fb440c36a8b2ee90c52e
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
  - Harmony
  - Python
  - R
  - SELINA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# multi-dataset-scrna-seq-reference-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Merges multiple single-cell RNA-seq reference datasets (gene-by-cell expression matrices with cell-type and platform metadata) and evaluates whether SELINA's MADA pre-training and autoencoder fine-tuning remove batch effects across sequencing platforms and between reference and query data, benchmarked against Harmony using LISI scores.

## When to use

Use when multiple scRNA-seq reference datasets from different sequencing platforms must be merged and batch-corrected (e.g. five liver datasets with *_expr.txt and *_meta.txt files), or when reference-query batch effects must be quantified before label transfer.

## When NOT to use

- Input is a single reference dataset with no platform or batch structure to remove — SMOTE/MADA pre-training adds no value
- Disease query data where fine-tuning should be skipped, since SELINA removes the fine-tuning step for disease mode (--disease flag) because disease expression profiles deviate greatly from normal data
- The task is cell-type annotation only with a pre-built reference atlas, not batch-effect evaluation of a merged reference

## Inputs

- Paired gene-by-cell expression matrices and metadata files per dataset (e.g. *_expr.txt and *_meta.txt with cell type and sequencing platform), one set per merged liver reference dataset
- Query scRNA-seq expression matrix for reference-query mixing evaluation

## Outputs

- lisi_scores.csv with platform-LISI and reference-query-LISI columns for both SELINA and Harmony
- Embedding UMAP plots written to PNG files
- Pre-trained SELINA model parameters (pre-trained_params.pt) and metadata (pre-trained_meta.pkl)

## How to apply

Merge the paired *_expr.txt and *_meta.txt files (gene-by-cell expression, cell type, sequencing platform) from the individual datasets into a single combined reference. Train SELINA's multi-adversarial domain adaptation network with 'selina train --path-in ./reference_data/ --path-out ./res/pre-train/', which first oversamples rare cell types with SMOTE to prevent minority cell types from being ignored. Then fine-tune the pre-trained parameters with the autoencoder via 'selina predict', which runs Fine-tuning1 for 50 epochs and Fine-tuning2 for 10 epochs to adapt the model to the query data distribution. Extract the integrated embeddings and compute LISI scores separately for platform-common, platform-specific, QR-common, and QR-specific cell groups; as a benchmark, run the same merged data through Harmony (immunogenomics/harmony) and compute matching LISI scores. Correct integration is judged by increased LISI for common cells and decreased LISI for specific cells relative to pre-integration, reproducing the Figure 3C/3G pattern.

## Related tools

- **SELINA** (Pre-trains the MADA network on the merged reference (with SMOTE oversampling of rare cell types) and fine-tunes with an autoencoder (Fine-tuning1: 50 epochs, Fine-tuning2: 10 epochs) to remove batch effects) — https://github.com/SELINA-team/SELINA.py
- **Harmony** (Benchmark integration method run on the same merged liver datasets to compute matching LISI scores for comparison (Figures 3D-3F)) — https://github.com/immunogenomics/harmony
- **Python** (Used to merge datasets, run SELINA, and compute LISI scores and UMAP plots)
- **R** (Used with the R package stack (Seurat, data.table, optparse) bundled in the SELINA conda environment and for Harmony comparison)

## Examples

```
selina train --path-in ./reference_data/ --path-out ./res/pre-train/ --outprefix pre-trained
```

## Evaluation signals

- lisi_scores.csv exists and contains platform-LISI and reference-query-LISI columns for both SELINA and Harmony
- SELINA shows increased LISI scores for platform-common cells and platform-common/QR-common cell types after integration
- SELINA shows decreased LISI scores for platform-specific and QR-specific cells after integration
- The pre-trained parameter file pre-trained_params.pt and metadata pre-trained_meta.pkl are produced by 'selina train' before fine-tuning
- Directional LISI changes for SELINA can be compared against Harmony's LISI scores on the same merged data (Figures 3D-3F pattern)

## Limitations

- Both the feature extractor and cell-type classifier transformations are evaluated on merged liver reference data only; behavior on other tissues is not demonstrated by this card
- Disease datasets deviate greatly from normal data, so SELINA removes the fine-tuning step in disease mode — autoencoder fine-tuning-based batch removal should not be assumed for disease queries
- The comparison benchmark is limited to Harmony; other integration methods are not covered

## Evidence

- [other] Merge the five liver reference datasets (paired *_expr.txt and *_meta.txt files giving gene-by-cell expression, cell type, and sequencing platform) into a single combined reference.: [evidence span withheld — anchor status: not_found]
