---
name: linear-classifier-training-on-embeddings
description: Use when you have a pretrained Miscell latent feature matrix (e.g. 43,095
  PBMC cells) plus ground-truth cell-type labels and want to quantify how well a linear
  probe predicts each of the 7 PBMC cell types.
when_to_use_negative:
- The dataset is small — Miscell demands a large sample size to obtain good performance
  on deep neural network embeddings
- Input is raw gene expression rather than extracted latent features; you must first
  run Miscell training/feature extraction (batch correction is not included in Miscell
  and must be done beforehand)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Miscell
  role: Self-supervised deep neural network encoder that produces the latent features
    the linear classifier is trained on
  repo: https://github.com/lixiangchun/Miscell
- name: scikit-learn
  role: Provides the single-layer linear classifier (logistic regression) fit on latent
    features, and one-vs-rest AUROC/accuracy evaluation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:751737f6528320fbf280c1394c9897b6471e8cf456b11d67ebcb7d1a86a4f1f2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  repo_url: https://github.com/lixiangchun/Miscell
  tools:
  - Miscell
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# linear-classifier-training-on-embeddings

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains a single-layer linear classifier (e.g. scikit-learn logistic regression) on Miscell latent features to predict cell types, and reports one-vs-rest AUROC and accuracy per cell type. Demonstrates that self-supervised deep embeddings are linearly separable enough for cell-type classification.

## When to use

Use when you have a pretrained Miscell latent feature matrix (e.g. 43,095 PBMC cells) plus ground-truth cell-type labels and want to quantify how well a linear probe predicts each of the 7 PBMC cell types.

## When NOT to use

- The dataset is small — Miscell demands a large sample size to obtain good performance on deep neural network embeddings
- Input is raw gene expression rather than extracted latent features; you must first run Miscell training/feature extraction (batch correction is not included in Miscell and must be done beforehand)

## Inputs

- Miscell latent feature matrix extracted from the PBMC scRNA-seq dataset (43,095 peripheral blood mononuclear cells)
- Cell-type labels for the 7 PBMC cell types per cell

## Outputs

- cell_type_classification_metrics.csv with per-cell-type one-vs-rest AUROC and classification accuracy
- Fitted single-layer linear classifier over Miscell latent features

## How to apply

First extract latent features from the input expression matrix (e.g. data/x_new_scanpy_HVG.npz, 1686 HVG input features) using a pretrained Miscell checkpoint via utils.load_pretrained_model(..., return_feature=True) and utils.extract_features. Then fit a single-layer linear classifier — scikit-learn logistic regression — mapping each cell's latent vector to its cell type. Evaluate one-vs-rest AUROC and classification accuracy per cell type; for the 7 PBMC types expect AUROC between 0.908 (CD8+ T cell) and 0.997 (monocyte) and accuracy between 92.6% and 99.8%. The rationale is that Miscell's self-supervised, contrastive-learning encoder produces a latent space where cell-type structure is linearly accessible, avoiding costly deep generative reconstruction. Write the per-cell-type metrics to cell_type_classification_metrics.csv.

## Related tools

- **Miscell** (Self-supervised deep neural network encoder that produces the latent features the linear classifier is trained on) — https://github.com/lixiangchun/Miscell
- **scikit-learn** (Provides the single-layer linear classifier (logistic regression) fit on latent features, and one-vs-rest AUROC/accuracy evaluation)

## Examples

```
model = utils.load_pretrained_model(arch, in_features, 'checkpoint_0199.pth.tar', return_feature=True); features = utils.extract_features(model, X); LogisticRegression().fit(features, labels)
```

## Evaluation signals

- Per-cell-type one-vs-rest AUROC falls within the reported range 0.908 (CD8+ T cell) to 0.997 (monocyte)
- Per-cell-type accuracy falls within the reported 92.6%–99.8% range
- Output file cell_type_classification_metrics.csv exists and contains AUROC and accuracy rows for all 7 PBMC cell types
- Classifier is single-layer and linear (e.g. sklearn LogisticRegression) — no nonlinear layers between latent features and predictions

## Limitations

- Miscell demands a large sample size to obtain good performance; small datasets may yield poor embeddings and hence weaker linear-probe metrics
- Miscell does not internally include batch-correction methods; batch effects must be corrected as a preprocessing step before training and feature extraction
- Feature extraction requires a pretrained checkpoint (e.g. checkpoint_0199.pth.tar) and, for training, at least two GPUs

## Evidence
