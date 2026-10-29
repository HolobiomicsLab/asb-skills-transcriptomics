---
name: cross-sample-label-transfer
description: Use when you have an annotated single-cell/nuclei RNA-seq dataset (e.g.,
  .h5ad with 'class_label' in AnnData.obs) and need to predict cell-type labels on
  cells from a different donor or sample using SIMS label transfer.
when_to_use_negative:
- Cells come from a different species than the training data — the authors explicitly
  limited validation to samples within the same species because 'neuronal subtypes
  diverge significantly between species'
- The training data lacks expert annotations ('class_label') — SIMS is fully supervised
  and cannot bootstrap labels from scratch
- Input matrices are gene x cell rather than cell x gene — SIMS assumes transcriptomes
  of individual cells as training samples
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Low-code supervised label-transfer pipeline (scsims) that trains a TabNet-based
    classifier on annotated cells and predicts labels on held-out samples
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying deep-learning architecture optimized for large tabular data (>45,000
    gene features) used by SIMS
- name: PyTorch Lightning
  role: Implementation framework providing the Trainer with EarlyStopping (patience=50
    on val_loss) and LearningRateMonitor callbacks
- name: Scanpy
  role: Preprocessing of the .h5ad input (filter_cells, filter_genes, normalize_total,
    log1p, scale) before model ingestion
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:743fee74c0ba8e46da7d3a5cf58384abedf59f3e5031843b95df701b572ab6d0
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-sample-label-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a SIMS (TabNet-based) single-cell RNA-seq classifier on an annotated donor/sample and transfer cell-type labels to held-out samples, validated on the Allen human adult cerebral cortex single-nuclei dataset (49,495 nuclei from donors H200.1023, H200.1025, H200.1030) where trans-sample accuracy falls in the 93.1%–95.8% range.

## When to use

Use when you have an annotated single-cell/nuclei RNA-seq dataset (e.g., .h5ad with 'class_label' in AnnData.obs) and need to predict cell-type labels on cells from a different donor or sample using SIMS label transfer.

## When NOT to use

- Cells come from a different species than the training data — the authors explicitly limited validation to samples within the same species because 'neuronal subtypes diverge significantly between species'
- The training data lacks expert annotations ('class_label') — SIMS is fully supervised and cannot bootstrap labels from scratch
- Input matrices are gene x cell rather than cell x gene — SIMS assumes transcriptomes of individual cells as training samples

## Inputs

- Annotated single-cell/single-nuclei RNA-seq expression matrix in .h5ad (AnnData) or .csv format, cells as rows x genes as columns
- Label file / obs column ('class_label') with expert-annotated cell-type labels for the training sample(s)
- Unlabeled or held-out expression matrix from a different donor/sample, preprocessed identically (log1p-normalized, scaled)

## Outputs

- Per-cell predicted labels and softmax probabilities (pandas DataFrame with pred_0/prob_0 columns, or top-k via top_k parameter)
- Trained SIMS model checkpoint (.ckpt) transferable to new samples
- Trans-sample accuracy and Macro F1 metrics for each train-donor/test-donor combination

## How to apply

Load each donor's annotated expression matrix (cell x gene) as an AnnData object and preprocess identically to training data: filter_cells(min_genes=100), filter_genes(min_cells=3), normalize_total, log1p, and scale. Initialize SIMS(adata=adata, labels_key='class_label'), configure the TabNet model with setup_model(n_a=64, n_d=64, weights=sims.weights) to weight the loss inversely by label frequency for imbalanced cell types, and set up a PyTorch Lightning trainer with EarlyStopping(monitor='val_loss', patience=50) and LearningRateMonitor(logging_interval='epoch') for up to 100 epochs. For trans-sample evaluation, train on one donor and test on the other two (three combinations for three donors), recording accuracy and Macro F1 on the held-out sample(s). Predictions on unlabeled samples use sims.predict(data, top_k=1) returning pred_0/prob_0 columns that can be written back to adata.obs. Expect ~98.0% accuracy (Macro F1 ~0.974) within-dataset and 93.1%–95.8% trans-sample accuracy per Figure 4 and Tables S2–S8; large deviations signal preprocessing mismatch or annotation errors in the training labels.

## Related tools

- **SIMS** (Low-code supervised label-transfer pipeline (scsims) that trains a TabNet-based classifier on annotated cells and predicts labels on held-out samples) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying deep-learning architecture optimized for large tabular data (>45,000 gene features) used by SIMS)
- **PyTorch Lightning** (Implementation framework providing the Trainer with EarlyStopping (patience=50 on val_loss) and LearningRateMonitor callbacks)
- **Scanpy** (Preprocessing of the .h5ad input (filter_cells, filter_genes, normalize_total, log1p, scale) before model ingestion)

## Examples

```
from scsims import SIMS; sims = SIMS(adata=adata, labels_key='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.setup_trainer(callbacks=[EarlyStopping(monitor='val_loss', patience=50)], max_epochs=100); sims.train(); preds = sims.predict(test_adata, top_k=1)
```

## Evaluation signals

- Within-dataset 80/20 split should reach ~98.0% accuracy with Macro F1 ~0.974 on the human adult cortex dataset
- Trans-sample (train on one donor, test on the other two) accuracies should fall in the 93.1%–95.8% range (Figure 4, Tables S2–S8)
- predict() output must be a DataFrame with pred_0/prob_0 columns (v4 canonical names, not the broken first_pred/first_prob)
- Held-out data must undergo identical preprocessing (min_genes=100, min_cells=3, normalize_total, log1p, scale) or accuracy will degrade
- Model checkpoints are saved every training epoch and EarlyStopping halts after 50 epochs without val_loss improvement

## Limitations

- Only validated for label transfer within the same species; trans-species prediction is untested
- As a supervised method, SIMS inherits errors from upstream normalization, clustering, and expert annotation; technical factors like sequencing technology can cause misnamed clusters
- Lack of naming conventions in neuroscience makes combining datasets from different labs for training manually intensive; ontology-based harmonization is only a proposed future direction
- Metadata (sequencing technology, cell line) is not yet integrated into the model
- SIMS is not the fastest benchmarked method; distillation/pruning optimizations are deferred to future work

## Evidence

- [supplementary] Trans-sample accuracy range from task card: [evidence span withheld — anchor status: not_found]
- [readme] Preprocessing requirement for inference: [evidence span withheld — anchor status: external_doc]
