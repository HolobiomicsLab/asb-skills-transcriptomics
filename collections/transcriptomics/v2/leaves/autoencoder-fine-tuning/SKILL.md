---
name: autoencoder-fine-tuning
description: Use when annotating normal (non-disease) scRNA-seq query data with a
  pre-trained SELINA model and the query distribution differs from the reference,
  requiring adaptation of pre-trained parameters before label transfer.
when_to_use_negative:
- Query data is a disease dataset (e.g. NSCLC, type 2 diabetes, Alzheimer's) — use
  --disease mode, which removes the fine-tuning step since disease expression profiles
  deviate substantially from normal data
- No pre-trained model exists yet for the target tissue/reference — selina train must
  run first to produce pre-trained_params.pt
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SELINA
  role: Framework implementing the autoencoder fine-tuning (Fine-tuning1/Fine-tuning2)
    invoked via 'selina predict'
  repo: https://github.com/SELINA-team/SELINA.py
- name: PyTorch
  role: Deep-learning backend used to train the autoencoder and serialize model parameters
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:38e8ebd1da178b355aab92e9d9b32f17ce96fed2bb0d008104f6f562b7cf921a
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Autoencoder fine-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fine-tunes SELINA's MADA pre-trained parameters on query data with an autoencoder whose encoder reuses the feature extractor plus cell-type discriminator, so pre-trained reference features adapt to the query data distribution before label transfer.

## When to use

Use when annotating normal (non-disease) scRNA-seq query data with a pre-trained SELINA model and the query distribution differs from the reference, requiring adaptation of pre-trained parameters before label transfer.

## When NOT to use

- Query data is a disease dataset (e.g. NSCLC, type 2 diabetes, Alzheimer's) — use --disease mode, which removes the fine-tuning step since disease expression profiles deviate substantially from normal data
- No pre-trained model exists yet for the target tissue/reference — selina train must run first to produce pre-trained_params.pt

## Inputs

- pre-trained_params.pt (MADA pre-trained model parameters from selina train)
- pre-trained_meta.pkl (reference cell types and genes)
- query_expr.txt (normalized, hg38-symbol gene-by-cell expression matrix from selina preprocess)
- query_res.rds (Seurat object with dimension reduction and clustering)

## Outputs

- query_predictions.txt (predicted cell type per query cell)
- query_probability.txt (per-cell probability over reference cell types)
- query_pred.png (UMAP with cell-type labels)
- query_DiffGenes.tsv (differentially expressed genes per cell type)
- query_cellsources.txt (predicted cell source per cell, disease mode only)

## How to apply

After 'selina train' produces pre-trained_params.pt from SMOTE-balanced, MADA-pre-trained reference data, run fine-tuning on the preprocessed query expression matrix (query_expr.txt, gene-by-cell, hg38 symbols). The autoencoder's encoder reuses the pre-trained feature extractor plus cell-type discriminator; training first fits the decoder to the query distribution (Fine-tuning1, 50 epochs) and then trains the encoder (Fine-tuning2, 10 epochs). 'selina predict' executes both fine-tuning stages automatically, after which the decoder is discarded and the labels from reference data are transferred to query cells using the fully-trained encoder. In disease mode, fine-tuning is skipped entirely by adding --disease, because disease expression profiles may deviate too much from normal references for meaningful adaptation.

## Related tools

- **SELINA** (Framework implementing the autoencoder fine-tuning (Fine-tuning1/Fine-tuning2) invoked via 'selina predict') — https://github.com/SELINA-team/SELINA.py
- **PyTorch** (Deep-learning backend used to train the autoencoder and serialize model parameters)

## Examples

```
selina predict --query-expr ./res/query_expr.txt --model ./res/pre-trained_params.pt --seurat ./res/query_res.rds --path-out ./res/predict/
```

## Evaluation signals

- Fine-tuning1 progress bar completes 50/50 epochs and Fine-tuning2 completes 10/10 epochs during 'selina predict'
- Reference-query LISI increases for cell types common to reference and query and decreases for query-specific cell types, indicating successful batch adaptation
- All four prediction outputs (query_predictions.txt, query_probability.txt, query_pred.png, query_DiffGenes.tsv) are generated in --path-out
- Predicted labels use only cell types present in pre-trained_meta.pkl reference cell-type keys
- In disease mode, verify fine-tuning stages are absent and query_cellsources.txt is additionally produced

## Limitations

- Fine-tuning is removed for disease-mode prediction because diverse abnormal-cell expression profiles deviate too much from normal references for the autoencoder adaptation to be reliable
- Pre-trained normal models must match the query tissue; query genes must be converted to hg38 symbol names or the encoder cannot map features correctly
- Disease pre-trained models cover only NSCLC, type 2 diabetes and Alzheimer's; users are recommended to train on their own disease data

## Evidence

- [readme] Since the expression profiles of disease data may deviate a lot from normal data, we removed the fine-tuning step when predicting for the disease data: [evidence span withheld — anchor status: external_doc]
- [results] fine-tuning analogously increased LISI for reference-query common cell types and decreased it for source-specific types: [evidence span withheld — anchor status: not_found]
