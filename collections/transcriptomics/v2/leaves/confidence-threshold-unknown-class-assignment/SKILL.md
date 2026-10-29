---
name: confidence-threshold-unknown-class-assignment
description: Use when a trained SIMS/TabNet checkpoint's raw logits must be turned
  into calibrated per-cell classification probabilities and cells below a confidence
  cutoff should be flagged as 'unknown' instead of receiving a cell-type label.
when_to_use_negative:
- When the goal is trans-species label transfer — SIMS calibration has only been validated
  for samples within the same species
- When no pre-existing annotations are available for training, since temperature scaling
  calibrates a supervised model that relies on expert-annotated clusters
- When raw logit ranks (not calibrated confidences) suffice, e.g. quick top-1 label
  transfer with no downstream confidence gating
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Pipeline whose trained TabNet-based model is post-processed with temperature
    scaling and thresholded to assign 'unknown' labels; predict() returns pred_0/prob_0
    columns
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying model architecture producing the raw logits that are temperature-scaled
    before softmax
- name: PyTorch Lightning
  role: Training framework that saves the checkpoints (with val_loss monitoring) consumed
    during calibration and inference
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:6e825c58476b22f662461d50410228914605df11a316dd63b1ddc78f862faa4f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jlehrer1/sims
  repo_url: https://github.com/braingeneers/SIMS
  tools:
  - PyTorch Lightning
  - SIMS
  - TabNet
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# confidence-threshold-unknown-class-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Post-process a trained SIMS/TabNet single-cell classifier with post-hoc temperature scaling to emit calibrated per-cell cluster probabilities, then apply a user-adjustable confidence threshold so that low-confidence cells are labeled 'unknown' rather than forced into a predicted cell type.

## When to use

Use when a trained SIMS/TabNet checkpoint's raw logits must be turned into calibrated per-cell classification probabilities and cells below a confidence cutoff should be flagged as 'unknown' instead of receiving a cell-type label.

## When NOT to use

- When the goal is trans-species label transfer — SIMS calibration has only been validated for samples within the same species
- When no pre-existing annotations are available for training, since temperature scaling calibrates a supervised model that relies on expert-annotated clusters
- When raw logit ranks (not calibrated confidences) suffice, e.g. quick top-1 label transfer with no downstream confidence gating

## Inputs

- Trained SIMS/TabNet model checkpoint (.ckpt) saved by PyTorch Lightning
- Target AnnData (.h5ad) expression matrix in cell x gene form, log1p-normalized
- Label file/AnnData column 'class_label' used at training time
- User-specified confidence threshold

## Outputs

- Calibrated per-class probability vectors per cell (softmax over temperature-scaled logits)
- Thresholded cell-type labels including 'unknown' assignments (e.g. calibrated_predictions.csv)
- sims_label / sims_confidence columns added to adata.obs via predict(top_k=1)

## How to apply

Load the trained SIMS/TabNet checkpoint (trained via PyTorch Lightning with val_loss monitoring and EarlyStopping patience=50) together with the target AnnData whose labels come from the 'class_label' column, ensuring the new data are preprocessed identically to training (filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale). Compute raw logits for every cell in the cell x gene expression matrix, fit a single scalar temperature on held-out validation logits by minimizing negative log-likelihood, and divide the logits by this temperature before the softmax to obtain calibrated per-class probabilities. Apply the user-specified confidence threshold to the maximum calibrated probability per cell, assigning cells below the threshold the label 'unknown' rather than pred_0. In the leave-one-out validation setting (Figure 4G-H), withhold one cell type from training and verify that the held-out type's cells fall below the threshold and are flagged 'unknown' while seen types retain confident, correct labels; write calibrated probabilities and thresholded labels to calibrated_predictions.csv.

## Related tools

- **SIMS** (Pipeline whose trained TabNet-based model is post-processed with temperature scaling and thresholded to assign 'unknown' labels; predict() returns pred_0/prob_0 columns) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying model architecture producing the raw logits that are temperature-scaled before softmax)
- **PyTorch Lightning** (Training framework that saves the checkpoints (with val_loss monitoring) consumed during calibration and inference)

## Examples

```
top1 = sims.predict(unlabeled_data, top_k=1); unlabeled_data.obs["sims_confidence"] = top1["prob_0"].values; unlabeled_data.obs["sims_label"] = top1["pred_0"].where(top1["prob_0"] >= threshold, "unknown")
```

## Evaluation signals

- In leave-one-out validation (Figure 4G-H), cells of the withheld cell type predominantly fall below the confidence threshold and are labeled 'unknown'
- Cells from training-seen cell types retain confident, correct labels at the chosen threshold
- Calibrated probabilities are proper softmax outputs (each cell's per-class probabilities sum to 1 after temperature scaling)
- The fitted temperature minimizes negative log-likelihood on held-out validation logits rather than train logits
- predict(top_k=1) output DataFrame exposes prob_0 as the maximum calibrated probability used for thresholding

## Limitations

- Calibration is only validated for label transfer within the same species; trans-species prediction of 'unknown' behavior is untested
- As a supervised post-processing, it inherits errors from upstream annotations — incorrect normalization or technical factors can misname clusters and corrupt the confidence signal
- A single scalar temperature is fitted globally, which may not calibrate all cell types equally on imbalanced label distributions

## Evidence

- [other] temperature scaling produces calibrated unknown-label probabilities: [evidence span withheld — anchor status: not_found]
- [other] confidence threshold assigns unknown: [evidence span withheld — anchor status: not_found]
- [readme] README documents softmax probability outputs from predict(): [evidence span withheld — anchor status: external_doc]
- [readme] README shows confidence column written from predict output: [evidence span withheld — anchor status: external_doc]
