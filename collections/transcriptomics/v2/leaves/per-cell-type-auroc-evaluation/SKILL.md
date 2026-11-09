---
name: per-cell-type-auroc-evaluation
description: Use when you have a pretrained Miscell latent feature matrix (e.g. from
  PBMC scRNA-seq) with known cell-type labels and need to quantify how well those
  features separate 7 cell types via a linear probe.
when_to_use_negative:
- 'The dataset is small: Miscell demands a large sample size to obtain good performance,
  so the linear probe on latent features would not be reliable on small datasets'
- 'The data contain uncorrected batch effects: Miscell does not internally include
  any batch-correction methods, so latent features (and hence classifier metrics)
  would be confounded'
- Input is raw gene expression rather than Miscell latent features; this skill evaluates
  the learned representation, so features must first be extracted with the pretrained
  model
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Miscell
  role: Self-supervised deep neural network whose latent feature encoder produces
    the features probed by the linear classifier
  repo: github.com/lixiangchun/Miscell
- name: scikit-learn
  role: Provides the single-layer linear classifier (logistic regression) and metrics
    used to compute one-vs-rest AUROC and accuracy per cell type
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:25f75bef01fe9b9b19a1bde3239fc30490b890018b2fae4be37e7ee837ac9a15
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  tools:
  - Miscell
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# per-cell-type-auroc-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates the cell-type information content of Miscell latent features by training a single-layer linear classifier (scikit-learn logistic regression) on the latent representation of 43,095 PBMCs and computing one-vs-rest AUROC and accuracy for each of the 7 PBMC cell types.

## When to use

Use when you have a pretrained Miscell latent feature matrix (e.g. from PBMC scRNA-seq) with known cell-type labels and need to quantify how well those features separate 7 cell types via a linear probe.

## When NOT to use

- The dataset is small: Miscell demands a large sample size to obtain good performance, so the linear probe on latent features would not be reliable on small datasets
- The data contain uncorrected batch effects: Miscell does not internally include any batch-correction methods, so latent features (and hence classifier metrics) would be confounded
- Input is raw gene expression rather than Miscell latent features; this skill evaluates the learned representation, so features must first be extracted with the pretrained model

## Inputs

- Miscell latent feature matrix for PBMC dataset (43,095 cells), extracted with a pretrained checkpoint (e.g. checkpoint_0199.pth.tar)
- Cell-type label vector assigning each of the 43,095 PBMCs to one of 7 cell types
- Normalized HVG input matrix (e.g. data/x_new_scanpy_HVG.npz with 1,686 input features) used by the pretrained model

## Outputs

- Per-cell-type one-vs-rest AUROC values for the 7 PBMC cell types
- Per-cell-type classification accuracy values
- cell_type_classification_metrics.csv containing the per-cell-type AUROC and accuracy

## How to apply

First extract the latent feature matrix by loading the trained Miscell checkpoint (e.g. checkpoint_0199.pth.tar) with utils.load_pretrained_model(arch, in_features, checkpoint, return_feature=True) and running utils.extract_features on the HVG-normalized input (e.g. data/x_new_scanpy_HVG.npz). Train a single-layer linear classifier (scikit-learn LogisticRegression) on the latent features to map each of the 43,095 peripheral blood mononuclear cells to one of 7 cell types. Compute one-vs-rest AUROC and classification accuracy per cell type. Judge correctness by checking that per-cell-type AUROC falls in the reported 0.908 (CD8+ T cell) to 0.997 (monocyte) range and per-cell-type accuracy lies within 92.6%-99.8%. Write the per-cell-type AUROC and accuracy values to cell_type_classification_metrics.csv.

## Related tools

- **Miscell** (Self-supervised deep neural network whose latent feature encoder produces the features probed by the linear classifier) — github.com/lixiangchun/Miscell
- **scikit-learn** (Provides the single-layer linear classifier (logistic regression) and metrics used to compute one-vs-rest AUROC and accuracy per cell type)

## Examples

```
from sklearn.linear_model import LogisticRegression; clf = LogisticRegression().fit(features, labels)
```

## Evaluation signals

- Per-cell-type one-vs-rest AUROC falls within the reported range 0.908 (CD8+ T cell, lowest) to 0.997 (monocyte, highest)
- Per-cell-type accuracy lies within the reported 92.6%-99.8% range for the 7 PBMC cell types
- All 7 PBMC cell types receive both an AUROC and an accuracy value (schema check on cell_type_classification_metrics.csv)
- The classifier is a single-layer linear model (logistic regression), not a nonlinear model, so it probes linear separability of the latent space
- The feature matrix dimensionality matches the checkpoint's latent feature output for the 43,095 cells

## Limitations

- Miscell demands a large sample size to obtain good performance, so the AUROC/accuracy probe is not meaningful on small datasets
- Miscell does not internally include any batch-correction methods; users must correct for batch effects before running Miscell or metrics may be biased
- The clinically significant latent features extracted by Miscell lack explainability, so high AUROC does not explain which features drive classification

## Evidence

- [results] Expected per-cell-type AUROC and accuracy ranges: [evidence span withheld — anchor status: not_found]
