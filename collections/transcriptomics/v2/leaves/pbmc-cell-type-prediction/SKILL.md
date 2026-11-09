---
name: pbmc-cell-type-prediction
description: Use when a trained Miscell model's latent feature matrix and matching
  cell-type labels are available and you need to quantify how linearly separable the
  cell types are (AUC/accuracy per type).
when_to_use_negative:
- The goal is de novo discovery of cell types or clusters — use Miscell's FIt-SNE
  + DBSCAN or scanpy Leiden clustering workflow instead of a supervised linear classifier
- 'The dataset is small: Miscell demands a large sample size to obtain good performance,
  so a linear classifier on its latent features of a small dataset is unreliable'
- Input is raw scRNA-seq count data — features must first be extracted with the trained
  Miscell encoder from normalized HVG features
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Miscell
  role: Self-supervised deep neural network (densenet encoder, moco-dim 64) whose
    latent features serve as input to the linear cell-type classifier
  repo: https://github.com/lixiangchun/Miscell
- name: scikit-learn
  role: Provides the single-layer linear classifier (logistic regression) trained
    on Miscell latent features to predict the 7 PBMC cell types
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:a7774bd5f21d131f1230f55752de2eed6b38501135b1fda4a5ff9b8ed78767a7
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pbmc-cell-type-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a single-layer linear classifier (logistic regression) on Miscell latent features extracted from the 43,095-cell PBMC scRNA-seq dataset to classify 7 PBMC cell types, evaluating per-type one-vs-rest AUROC and accuracy.

## When to use

Use when a trained Miscell model's latent feature matrix and matching cell-type labels are available and you need to quantify how linearly separable the cell types are (AUC/accuracy per type).

## When NOT to use

- The goal is de novo discovery of cell types or clusters — use Miscell's FIt-SNE + DBSCAN or scanpy Leiden clustering workflow instead of a supervised linear classifier
- The dataset is small: Miscell demands a large sample size to obtain good performance, so a linear classifier on its latent features of a small dataset is unreliable
- Input is raw scRNA-seq count data — features must first be extracted with the trained Miscell encoder from normalized HVG features

## Inputs

- Miscell latent feature matrix for the PBMC dataset (43,095 peripheral blood mononuclear cells)
- 7 PBMC cell-type labels per cell
- Trained Miscell checkpoint file (e.g. checkpoint_0199.pth.tar) if features must be re-extracted
- Normalized HVG expression matrix in .npz format (e.g. data/x_new_scanpy_HVG.npz, in_features=1686)

## Outputs

- Per-cell-type one-vs-rest AUROC values for the 7 PBMC cell types
- Per-cell-type classification accuracy values
- cell_type_classification_metrics.csv with per-cell-type AUROC and accuracy

## How to apply

Extract the latent feature matrix from the Miscell pretrained checkpoint (e.g. densenet11, moco-dim 64) using utils.extract_features on the normalized HVG expression matrix loaded from an .npz file (e.g. data/x_new_scanpy_HVG.npz with in_features=1686). Train a single-layer linear classifier such as scikit-learn logistic regression mapping each cell's 64-dimensional latent vector to one of the 7 PBMC cell-type labels. Compute one-vs-rest AUROC and classification accuracy per cell type. The paper's reported range — AUROC 0.908 (CD8+ T cell) to 0.997 (monocyte) and accuracy 92.6%–99.8% — serves as the reference benchmark to validate the reproduction.

## Related tools

- **Miscell** (Self-supervised deep neural network (densenet encoder, moco-dim 64) whose latent features serve as input to the linear cell-type classifier) — https://github.com/lixiangchun/Miscell
- **scikit-learn** (Provides the single-layer linear classifier (logistic regression) trained on Miscell latent features to predict the 7 PBMC cell types)

## Examples

```
model = utils.load_pretrained_model(arch, in_features, checkpoint, return_feature=True); features = utils.extract_features(model, X); clf = LogisticRegression().fit(features, labels); pred = clf.predict(features)
```

## Evaluation signals

- Per-cell-type AUROC falls within the reported range 0.908 (CD8+ T cell) to 0.997 (monocyte)
- Per-cell-type accuracy lies within the reported range 92.6% to 99.8%
- Extracted feature dimensionality equals the moco-dim latent size (64) and matches the number of cells (43,095)
- cell_type_classification_metrics.csv contains exactly 7 rows, one per PBMC cell type, with AUROC and accuracy columns

## Limitations

- Miscell demands a large sample size to obtain good performance, limiting use on small datasets
- The clinically significant latent features extracted by Miscell lack explainability
- The biological mechanisms underlying the latent features of Miscell trained on these single-cell datasets remain unknown

## Evidence
