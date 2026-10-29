---
name: leave-one-out-cell-type-generalization-evaluation
description: Use when evaluating whether a trained single-cell classifier (SIMS/TabNet
  checkpoint) can distinguish seen from unseen cell types, or when calibrating per-cell
  probabilities and flagging low-confidence cells as 'unknown' before label transfer.
when_to_use_negative:
- The target cells come from a different species than the training data — SIMS has
  only been validated for label transfer within the same species
- The expression matrix is not in cell-x-gene form or has not been processed the same
  way as the training data (filter_cells min_genes=100, filter_genes min_cells=3,
  normalize_total, log1p, scale)
- The goal is unsupervised discovery of cell types with no pre-existing annotations
  — SIMS is a supervised model that relies on prior expert annotations
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SIMS
  role: Pipeline providing the trained TabNet classifier, checkpoint loading, and
    predict() that returns top-k labels and softmax probabilities per cell
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying model architecture whose logits are temperature-scaled to produce
    calibrated per-cell class probabilities
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Training framework whose val_loss-monitored checkpoints and validation split
    supply the held-out logits used for temperature fitting
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:d70fd2c420bd9cf855768ddf02fdb1b70ec658f9f367bfd7b341c11233f1f25b
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
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# leave-one-out-cell-type-generalization-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A calibration-based generalization test for the SIMS/TabNet label-transfer classifier: withhold one cell type from training, apply post-hoc temperature scaling to emit calibrated per-cell probabilities, and verify that held-out cells fall below a user-set confidence threshold and are labeled 'unknown' rather than misassigned.

## When to use

Use when evaluating whether a trained single-cell classifier (SIMS/TabNet checkpoint) can distinguish seen from unseen cell types, or when calibrating per-cell probabilities and flagging low-confidence cells as 'unknown' before label transfer.

## When NOT to use

- The target cells come from a different species than the training data — SIMS has only been validated for label transfer within the same species
- The expression matrix is not in cell-x-gene form or has not been processed the same way as the training data (filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale)
- The goal is unsupervised discovery of cell types with no pre-existing annotations — SIMS is a supervised model that relies on prior expert annotations

## Inputs

- Trained SIMS/TabNet model checkpoint (.ckpt, e.g. trained via PyTorch Lightning)
- AnnData object (.h5ad) with cell-x-gene log1p-normalized expression matrix
- Label file with 'class_label' column used at training time
- User-specified confidence threshold for 'unknown' assignment

## Outputs

- Calibrated per-cell per-class probability vectors (e.g. calibrated_predictions.csv)
- Thresholded per-cell labels including 'unknown' assignments
- Leave-one-out qualitative assessment: held-out type labeled 'unknown' vs. seen types confidently labeled

## How to apply

Load the trained SIMS/TabNet checkpoint (trained via PyTorch Lightning with EarlyStopping monitoring 'val_loss', patience=50) together with the target AnnData whose labels come from the 'class_label' column. Compute raw logits for every cell in the cell-x-gene expression matrix (log1p-normalized input), then fit a single temperature parameter on held-out validation logits by minimizing negative log-likelihood and divide logits by this scalar before the softmax to obtain calibrated per-class probabilities. Apply the user-specified confidence threshold to the maximum calibrated probability per cell, assigning cells below the threshold the label 'unknown' and writing calibrated probability vectors plus thresholded labels (e.g. calibrated_predictions.csv). In the leave-one-out setting, withhold one cell type from training, run calibration on the remaining types, and confirm that held-out type cells fall below the threshold and are flagged 'unknown' (as evaluated in Figure 4G-H), while seen cell types retain confident, correct labels.

## Related tools

- **SIMS** (Pipeline providing the trained TabNet classifier, checkpoint loading, and predict() that returns top-k labels and softmax probabilities per cell) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying model architecture whose logits are temperature-scaled to produce calibrated per-cell class probabilities) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Training framework whose val_loss-monitored checkpoints and validation split supply the held-out logits used for temperature fitting)

## Examples

```
from scsims import SIMS; sims = SIMS(weights_path='model.ckpt'); top1 = sims.predict(unlabeled_data, top_k=1); unlabeled_data.obs['sims_label'] = top1['pred_0'].values
```

## Evaluation signals

- In the leave-one-out run, cells of the withheld cell type are predominantly labeled 'unknown' at the chosen confidence threshold
- Cells of seen (in-training) cell types retain confident, correct predicted labels after thresholding
- Maximum calibrated probabilities for unseen-type cells fall below the user threshold while seen-type cells exceed it
- The probability DataFrame follows the expected schema (pred_0/prob_0 columns, one row per input cell)
- The fitted temperature is a single scalar that reduces negative log-likelihood on held-out validation logits

## Limitations

- Trans-species label transfer remains untested; neuronal subtypes diverge significantly between species
- As a supervised model, SIMS inherits errors from upstream annotations (normalization, clustering, differential expression), which can propagate into calibration and thresholding
- Success is judged qualitatively; there is no numeric metric specified for the leave-one-out 'unknown' flagging beyond visual predominance of 'unknown' labels

## Evidence

- [methods] temperature scaling fit on held-out validation logits: [evidence span withheld — anchor status: not_found]
- [other] user-adjustable unknown threshold: [evidence span withheld — anchor status: not_found]
- [readme] predict output schema: [evidence span withheld — anchor status: external_doc]
- [readme] input normalization requirement: [evidence span withheld — anchor status: external_doc]
