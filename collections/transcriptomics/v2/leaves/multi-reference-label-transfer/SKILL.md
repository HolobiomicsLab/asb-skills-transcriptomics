---
name: multi-reference-label-transfer
description: Use when a gene-by-cell scRNA-seq query matrix needs cell-type labels
  transferred from multiple heterogeneous reference datasets, especially with imbalanced/rare
  cell types or batch effects between reference and query.
when_to_use_negative:
- The query is non-human scRNA-seq — SELINA's reference atlas covers 230 distinct
  human cell types, with transfer to other species only anticipated, not demonstrated
- Only a single small marker list is available with no reference expression data —
  marker-based tools like Garnett or SCINA fit that scenario instead
- The query is a disease dataset run without the --disease flag, where fine-tuning
  against normal references would be inappropriate
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SELINA
  role: Core framework executing preprocess, train (SMOTE + MADA pre-training), and
    predict (autoencoder fine-tuning and label transfer) steps
  repo: https://github.com/SELINA-team/SELINA.py
- name: presto
  role: Fast differential expression used to generate cluster/cell-type marker gene
    tables during preprocessing output
  repo: https://github.com/immunogenomics/presto
- name: harmony
  role: Batch-effect integration referenced in the article's tool ecosystem for harmonizing
    reference/query data
  repo: https://github.com/immunogenomics/harmony
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:2ad3fd71b5bad94d312c09add91d3730811c8de4bd7638510bec50af7941306a
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
  - CHETAH
  - Garnett
  - Python
  - R
  - SCINA
  - SELINA
  - SingleCellNet
  - SingleR
  - harmony
  - presto
  - scPred
  - scmap
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# multi-reference-label-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate human scRNA-seq query cells using SELINA, a deep-learning framework that jointly leverages multiple pre-curated reference datasets (1.7 million cells, 230 human cell types) via SMOTE cell-type balancing, multi-adversarial domain adaptation (MADA) pre-training, and autoencoder fine-tuning before label transfer.

## When to use

Use when a gene-by-cell scRNA-seq query matrix needs cell-type labels transferred from multiple heterogeneous reference datasets, especially with imbalanced/rare cell types or batch effects between reference and query.

## When NOT to use

- The query is non-human scRNA-seq — SELINA's reference atlas covers 230 distinct human cell types, with transfer to other species only anticipated, not demonstrated
- Only a single small marker list is available with no reference expression data — marker-based tools like Garnett or SCINA fit that scenario instead
- The query is a disease dataset run without the --disease flag, where fine-tuning against normal references would be inappropriate

## Inputs

- Query gene-by-cell scRNA-seq expression matrix in plain, h5, or mtx format
- Reference dataset folder (one or more normalized expression matrices with cell-type labels) or a pre-trained model from SELINA-team/SELINA-reference
- Optional prob-cutoff threshold (default 0.9) and QC cutoffs for preprocessing

## Outputs

- pre-trained_params.pt (model parameters) and pre-trained_meta.pkl (reference cell types/genes)
- query_predictions.txt (per-cell predicted cell type)
- query_probability.txt (probability per reference cell type per cell)
- query_res.rds Seurat object and query_cluster.png UMAP from preprocessing
- query_cellsources.txt when run in disease mode

## How to apply

First run `selina preprocess` on the query to normalize the expression profile, convert genes to hg38 symbol names, and perform PCA (30 PCs), UMAP, and Louvain clustering (resolution 0.6) with default QC cutoffs (count-cutoff 1000, gene-cutoff 500, cell-cutoff 10, 2000 variable genes). Then run `selina train` on the reference folder to produce pre-trained_params.pt and pre-trained_meta.pkl; training internally oversamples rare cell types with SMOTE before MADA-based supervised pre-training. Finally run `selina predict` with --prob-cutoff 0.9 against either your own trained model or the pre-trained models from SELINA-team/SELINA-reference; the autoencoder fine-tunes the pre-trained parameters to fit the query distribution before transferring reference labels. For disease queries (e.g., NSCLC, type 2 diabetes, Alzheimer's), add the --disease flag, which skips fine-tuning and additionally predicts cell sources.

## Related tools

- **SELINA** (Core framework executing preprocess, train (SMOTE + MADA pre-training), and predict (autoencoder fine-tuning and label transfer) steps) — https://github.com/SELINA-team/SELINA.py
- **presto** (Fast differential expression used to generate cluster/cell-type marker gene tables during preprocessing output) — https://github.com/immunogenomics/presto
- **harmony** (Batch-effect integration referenced in the article's tool ecosystem for harmonizing reference/query data) — https://github.com/immunogenomics/harmony

## Examples

```
selina predict --query-expr ./res/query_expr.txt --model ./res/pre-trained_params.pt --seurat ./res/query_res.rds --path-out ./res/predict/disease --disease
```

## Evaluation signals

- Prediction outputs the four expected files (query_predictions.txt, query_probability.txt, query_pred.png, query_DiffGenes.tsv) and probabilities exceed the 0.9 cutoff for accepted labels
- On the Allen Brain Atlas external benchmark, SELINA achieves average accuracy ~93.32% and MacroF1 ~0.9336 across the 19 cell types with >1,000 cells, ranking second of 9 tools
- With SMOTE enabled, rare cell types are rescued (about one-third of CD4 T cells and half of mast cells correctly annotated) versus the non-SMOTE baseline model
- Training terminates with expected Fine-tuning1 (50/50) and Fine-tuning2 (10/10) progress iterations and emits pre-trained_params.pt plus pre-trained_meta.pkl
- Preprocessing generates query_cluster.png showing UMAP clusters consistent with resolution-0.6 Louvain clustering and differentially expressed genes per cluster

## Limitations

- The reference still suffers from imbalanced tissue coverage and cell-type abundance despite its 1.7-million-cell scale
- Disease pre-trained models cover only non-small-cell lung carcinoma, type 2 diabetes, and Alzheimer's; for other diseases users should train on their own data
- Graph-based structural/explainability methods and transfer to other species are future plans, not implemented

## Evidence
