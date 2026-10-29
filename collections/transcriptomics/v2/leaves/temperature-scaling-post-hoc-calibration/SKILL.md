---
name: temperature-scaling-post-hoc-calibration
description: Use when a trained SIMS (TabNet, PyTorch Lightning) classifier must emit
  trustworthy per-cell probabilities, and low-confidence cells — e.g., cell types
  withheld from training — need to be labeled 'unknown' via a user-adjustable threshold.
when_to_use_negative:
- The input labels come from a different species than the training data — SIMS calibration
  has only been validated for label transfer within the same species
- The target expression matrix has not undergone the same preprocessing (normalize_total,
  log1p, scale) as the training data — calibration probabilities would be invalid
- The model checkpoint was not trained with the same class_label vocabulary as the
  target dataset, since the calibrated probabilities are over the original trained
  classes only
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: 'Pipeline whose trained checkpoint is post-processed: logits are temperature-scaled,
    softmax probabilities thresholded, and ''unknown'' labels assigned via sims.predict'
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying model architecture producing the raw logits that are calibrated
    by temperature scaling
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Training framework whose checkpoint (val_loss-monitored) and validation logits
    supply the held-out data used to fit the temperature parameter
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:0b0cc0fb4577af31934149f426afec4fc94ec04ef350a7827fef72b81c05ed08
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# temperature-scaling-post-hoc-calibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies post-hoc temperature scaling to a trained SIMS/TabNet checkpoint so that raw logits become calibrated per-cell class probabilities, which are then thresholded to flag low-confidence cells as 'unknown' (e.g., unseen cell types in leave-one-out validation).

## When to use

Use when a trained SIMS (TabNet, PyTorch Lightning) classifier must emit trustworthy per-cell probabilities, and low-confidence cells — e.g., cell types withheld from training — need to be labeled 'unknown' via a user-adjustable threshold.

## When NOT to use

- The input labels come from a different species than the training data — SIMS calibration has only been validated for label transfer within the same species
- The target expression matrix has not undergone the same preprocessing (normalize_total, log1p, scale) as the training data — calibration probabilities would be invalid
- The model checkpoint was not trained with the same class_label vocabulary as the target dataset, since the calibrated probabilities are over the original trained classes only

## Inputs

- Trained SIMS/TabNet model checkpoint (.ckpt, e.g. 'myawesomemodel.ckpt') trained with PyTorch Lightning
- Target AnnData (.h5ad) with cell-by-gene log1p-normalized expression matrix
- Held-out validation logits from the training run (for fitting the temperature parameter)
- User-specified confidence threshold value

## Outputs

- Calibrated per-cell probability vectors over cell-type classes
- Thresholded cell-type labels with low-confidence cells assigned 'unknown'
- calibrated_predictions.csv containing probability vectors and thresholded labels
- Per-cell prediction DataFrame columns pred_0/pred_1/... and prob_0/prob_1/... (from sims.predict)

## How to apply

Load the trained SIMS/TabNet checkpoint (trained via PyTorch Lightning with EarlyStopping monitoring 'val_loss' and LearningRateMonitor) together with the target AnnData whose labels come from the 'class_label' column, after applying the same preprocessing used at training time (filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale). Compute raw logits for every cell in the cell-by-gene expression matrix, then fit a single scalar temperature on held-out validation logits by minimizing negative log-likelihood; divide the logits by this temperature before the softmax to obtain calibrated per-class probabilities. Apply the user-specified confidence threshold to the maximum calibrated probability per cell, assigning cells below the threshold the label 'unknown'. In the leave-one-out setting, withhold one cell type from training, calibrate on the remaining types, and confirm the held-out type's cells fall below threshold and are flagged 'unknown' (as evaluated in Figure 4G-H). Write calibrated probability vectors and thresholded labels, including 'unknown' assignments, to calibrated_predictions.csv.

## Related tools

- **SIMS** (Pipeline whose trained checkpoint is post-processed: logits are temperature-scaled, softmax probabilities thresholded, and 'unknown' labels assigned via sims.predict) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying model architecture producing the raw logits that are calibrated by temperature scaling) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Training framework whose checkpoint (val_loss-monitored) and validation logits supply the held-out data used to fit the temperature parameter)

## Examples

```
sims = SIMS(weights_path='myawesomemodel.ckpt'); top1 = sims.predict(unlabeled_data, top_k=1); unlabeled_data.obs['sims_confidence'] = top1['prob_0'].values
```

## Evaluation signals

- In leave-one-out validation (Figure 4G-H), cells from the withheld cell type fall below the confidence threshold and are predominantly labeled 'unknown'
- Cells from cell types seen during training retain confident, correct labels at the chosen threshold
- Calibrated probability vectors sum to 1 per cell across the trained class vocabulary after temperature-scaled softmax
- calibrated_predictions.csv contains one probability vector and one thresholded label (including 'unknown') per input cell
- Fitted temperature minimizes negative log-likelihood on held-out validation logits

## Limitations

- The confidence threshold is user-adjustable, so 'unknown' flagging behavior depends on the chosen cutoff rather than a fixed criterion
- SIMS is a supervised model that relies on pre-existing annotations; technical factors such as sequencing technology or incorrect normalization can lead to misnamed clusters and biased calibration
- SIMS has only been validated for label transfer within the same species; trans-species probability calibration remains untested

## Evidence

- [other] Fit a single temperature parameter on held-out validation logits by minimizing negative log-likelihood (post-hoc temperature scaling): [evidence span withheld — anchor status: not_found]
- [readme] README: checkpoint loading and prediction: [evidence span withheld — anchor status: external_doc]
- [readme] README: probability outputs: [evidence span withheld — anchor status: external_doc]
- [intro] Article intro: training framework: [evidence span withheld — anchor status: not_found]
