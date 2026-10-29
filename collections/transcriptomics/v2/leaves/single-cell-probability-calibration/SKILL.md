---
name: single-cell-probability-calibration
description: Use when a trained SIMS model must emit per-cell classification confidences,
  or when novel/unseen cell types in the target AnnData should be flagged as 'unknown'
  instead of receiving a forced label.
when_to_use_negative:
- The target dataset contains only cell types seen during training and no confidence
  gating is needed — direct predict()/label transfer suffices without calibration
- The model has not yet been trained or no validation split exists — temperature scaling
  requires held-out logits to fit the temperature parameter
- Cross-species label transfer is the goal — the paper states SIMS has only been validated
  within the same species, so 'unknown' flagging behavior across species is untested
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Pipeline whose trained checkpoint is calibrated; provides the logits, predict
    interface, and class_label-based labeling
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying model architecture whose raw logits are temperature-scaled before
    softmax
- name: PyTorch Lightning
  role: Training framework with val_loss monitoring/checkpointing that produces the
    checkpoint used for calibration
- name: Scanpy
  role: Preprocessing (filter_cells, filter_genes, normalize_total, log1p, scale)
    of the target AnnData so inference inputs match training distribution
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:b72969b9d0dc686b8870f8e0e4e2642c11d3e50e882775de9f97280c4571743f
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
  - Scanpy
  - TabNet
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-cell-probability-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Post-hoc temperature scaling of a trained SIMS/TabNet single-cell classifier to convert raw logits into calibrated per-class probabilities, with a user-adjustable confidence threshold that flags low-confidence cells as 'unknown'.

## When to use

Use when a trained SIMS model must emit per-cell classification confidences, or when novel/unseen cell types in the target AnnData should be flagged as 'unknown' instead of receiving a forced label.

## When NOT to use

- The target dataset contains only cell types seen during training and no confidence gating is needed — direct predict()/label transfer suffices without calibration
- The model has not yet been trained or no validation split exists — temperature scaling requires held-out logits to fit the temperature parameter
- Cross-species label transfer is the goal — the paper states SIMS has only been validated within the same species, so 'unknown' flagging behavior across species is untested

## Inputs

- Trained SIMS/TabNet model checkpoint (.ckpt, trained via PyTorch Lightning)
- Target AnnData (.h5ad) expression matrix in cell x gene orientation, processed identically to training data (normalize_total, log1p, scale)
- Held-out validation logits from the labeled training set for fitting the temperature parameter
- User-specified confidence threshold value
- Label file with the 'class_label' column used at training time

## Outputs

- Calibrated per-class probability vectors for every cell (post temperature-scaled softmax)
- Thresholded per-cell labels, including 'unknown' assignments for low-confidence cells
- calibrated_predictions.csv containing probability vectors and final labels

## How to apply

Load the trained SIMS/TabNet checkpoint (trained via PyTorch Lightning with val_loss monitoring) together with the target AnnData whose labels come from the 'class_label' column, and compute raw logits for every cell in the log1p-normalized, scaled expression matrix. Fit a single temperature parameter on held-out validation logits by minimizing negative log-likelihood (post-hoc temperature scaling), then divide the logits by this scalar before the softmax to obtain calibrated per-class probabilities. Apply the user-specified confidence threshold to the maximum calibrated probability per cell: cells below the threshold receive the label 'unknown'. To validate, run a leave-one-out experiment withholding one cell type from training and confirm the held-out type's cells fall below the threshold and are flagged 'unknown' (Figure 4G-H) while seen types retain confident, correct labels. Write the calibrated probability vectors and thresholded labels (including 'unknown' assignments) to calibrated_predictions.csv.

## Related tools

- **SIMS** (Pipeline whose trained checkpoint is calibrated; provides the logits, predict interface, and class_label-based labeling) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying model architecture whose raw logits are temperature-scaled before softmax)
- **PyTorch Lightning** (Training framework with val_loss monitoring/checkpointing that produces the checkpoint used for calibration)
- **Scanpy** (Preprocessing (filter_cells, filter_genes, normalize_total, log1p, scale) of the target AnnData so inference inputs match training distribution)

## Examples

```
sims = SIMS(weights_path='myawesomemodel.ckpt'); top1 = sims.predict(unlabeled_data, top_k=1); unlabeled_data.obs['sims_confidence'] = top1['prob_0'].values
```

## Evaluation signals

- In leave-one-out validation, cells of the withheld (unseen) cell type are predominantly labeled 'unknown' at the chosen confidence threshold (Figure 4G-H)
- Cells from cell types seen during training retain confident, correct labels with maximum calibrated probability above the threshold
- Calibrated probabilities sum to 1 per cell after temperature-scaled softmax, and probabilities are better calibrated (lower NLL) than raw softmax outputs
- calibrated_predictions.csv contains one probability vector and one label (including 'unknown') per input cell

## Limitations

- Threshold choice is user-adjustable and validation is qualitative — success is judged by whether held-out types fall below the threshold, so miscalibrated thresholds can over- or under-flag 'unknown' cells
- SIMS is a supervised model relying on pre-existing annotations; errors from technical factors (sequencing technology, incorrect normalization) in the 'class_label' labels propagate into calibration
- Trans-species prediction remains untested, so calibration behavior on divergent subtypes from other species is unknown

## Evidence

- [methods] temperature scaling yields calibrated unknown-cell flagging: [evidence span withheld — anchor status: not_found]
- [results] threshold determines unknown labeling: [evidence span withheld — anchor status: not_found]
- [readme] log1p-normalized inference input requirement: [evidence span withheld — anchor status: external_doc]
- [readme] checkpoint loading and predict for inference on unlabeled data: [evidence span withheld — anchor status: external_doc]
