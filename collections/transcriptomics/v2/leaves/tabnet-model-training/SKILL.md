---
name: tabnet-model-training
description: Use when a labeled single-cell RNA-seq dataset (cell x gene AnnData/.h5ad
  with a 'class_label' column) must be trained into a cell-type classifier for label
  transfer across splits, ages, or cell lines.
when_to_use_negative:
- The data is not single-cell transcriptomics (e.g., bulk RNA-seq, spatial proteomics,
  or an already-aggregated feature table) — SIMS expects cell x gene matrices with
  per-cell labels
- Label transfer across species is the goal — the authors limited validation to samples
  within the same species because neuronal subtypes diverge significantly between
  species
- No pre-existing cell-type annotations are available and none can be produced — SIMS
  is supervised and relies on annotated training data
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Low-code pipeline wrapping TabNet for single-cell classification; instantiated
    with SIMS(adata=adata, labels_key='class_label') and trained via sims.train()
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying deep-learning model architecture optimized for large-scale tabular
    data (handles >45,000 gene features); configured with n_a=64, n_d=64 and class-frequency
    loss weights
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Training framework providing the pl.Trainer with EarlyStopping (monitor='val_loss',
    patience=50), LearningRateMonitor, WandbLogger, and max_epochs=100
- name: Scanpy
  role: 'Preprocessing of AnnData input: filter_cells(min_genes=100), filter_genes(min_cells=3),
    normalize_total, log1p, scale before training/inference'
- name: Gruffi
  role: Used to remove stressed cells from organoid datasets before re-evaluating
    SIMS label transfer
  repo: https://github.com/jn-goe/gruffi
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_002
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:1b94f7d4891e01e284416ab72b26caf6e2f86237898b49e80a645bb162e06b01
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
  - Gruffi
  - PyTorch Lightning
  - SIMS
  - Scanpy
  - TabNet
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# tabnet-model-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a SIMS (scalable, interpretable machine learning for single cell) TabNet-based classifier on labeled single-cell RNA-seq AnnData matrices for cell-type label transfer, using loss weighting by label frequency, early stopping on val_loss (patience=50), and up to 100 epochs via PyTorch Lightning.

## When to use

Use when a labeled single-cell RNA-seq dataset (cell x gene AnnData/.h5ad with a 'class_label' column) must be trained into a cell-type classifier for label transfer across splits, ages, or cell lines.

## When NOT to use

- The data is not single-cell transcriptomics (e.g., bulk RNA-seq, spatial proteomics, or an already-aggregated feature table) — SIMS expects cell x gene matrices with per-cell labels
- Label transfer across species is the goal — the authors limited validation to samples within the same species because neuronal subtypes diverge significantly between species
- No pre-existing cell-type annotations are available and none can be produced — SIMS is supervised and relies on annotated training data

## Inputs

- Cell-by-gene AnnData expression matrix (.h5ad), log1p-normalized and scaled, with a 'class_label' observation column
- Label file/dataset with a common 'class_label' column for supervised training
- Optional large unlabeled corpus (.h5ad) for TabNet self-supervised pretraining (v4)

## Outputs

- Trained SIMS/TabNet model checkpoint (.ckpt) saved every training epoch
- Held-out test metrics (accuracy, Macro F1) per split
- Per-cell predicted labels and softmax probabilities (pandas DataFrame with pred_0/prob_0 columns from sims.predict)
- Explainability matrix of gene attributions (sims.explain)

## How to apply

Instantiate SIMS on a log1p-normalized, scaled, cell-by-gene AnnData object with SIMS(adata=adata, labels_key='class_label'). Configure the underlying TabNet with sims.setup_model(n_a=64, n_d=64, weights=sims.weights), weighting the loss inversely by label frequency so rare cell types in imbalanced datasets (e.g., 17 developing-cortex types, 42 Allen cortex/hippocampus types) are learned. Set up the PyTorch Lightning trainer with EarlyStopping(monitor='val_loss', patience=50), LearningRateMonitor(logging_interval='epoch'), a Weights & Biases logger, and max_epochs=100, then call sims.train(). For small labeled sets, optionally warm-start from TabNet self-supervised pretraining (pretraining_ratio=0.2, then load_pretrainer before train()). Evaluate held-out accuracy and Macro F1 against reported benchmarks (e.g., 97.6% accuracy / 0.983 Macro F1 on the Allen 80/20 split; 84.2% / 0.791 on the E18+P1 cortical development split).

## Related tools

- **SIMS** (Low-code pipeline wrapping TabNet for single-cell classification; instantiated with SIMS(adata=adata, labels_key='class_label') and trained via sims.train()) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying deep-learning model architecture optimized for large-scale tabular data (handles >45,000 gene features); configured with n_a=64, n_d=64 and class-frequency loss weights) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Training framework providing the pl.Trainer with EarlyStopping (monitor='val_loss', patience=50), LearningRateMonitor, WandbLogger, and max_epochs=100)
- **Scanpy** (Preprocessing of AnnData input: filter_cells(min_genes=100), filter_genes(min_cells=3), normalize_total, log1p, scale before training/inference)
- **Gruffi** (Used to remove stressed cells from organoid datasets before re-evaluating SIMS label transfer) — https://github.com/jn-goe/gruffi

## Examples

```
sims = SIMS(adata=adata, labels_key='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.setup_trainer(callbacks=[EarlyStopping(monitor='val_loss', patience=50), LearningRateMonitor(logging_interval='epoch')], max_epochs=100); sims.train()
```

## Evaluation signals

- 80/20-split test accuracy ~97.6% and Macro F1 ~0.983 on the Allen adult mouse cortex/hippocampus dataset (42 cell types)
- 7% training subsample (3,285 cells) still yields >95% accuracy and Macro F1 ~0.90, confirming data efficiency
- E18+P1 combined-age split reaches ~84.2% accuracy / 0.791 Macro F1 across 17 cell types; P1->E18 cross-age transfer ~73.6% / 0.674
- 11A-trained organoid model predicts PGP1 at ~71.3% and GM8330 at ~67.4% accuracy, with radial glia misclassified as immature projection neurons ~82% (PGP1) and ~42% (GM8330)
- Training log shows EarlyStopping on val_loss active and model checkpoints written every epoch

## Limitations

- Validated only within the same species; trans-species prediction is untested because neuronal subtypes diverge significantly between species
- As a supervised model, SIMS depends on pre-existing annotations; technical factors such as sequencing technology or incorrect normalization can propagate misnamed clusters
- Not the fastest benchmarked method; speed optimizations like model distillation via student-teacher methods are deferred to future work
- Lack of neuroscience naming conventions makes combining datasets from different labs for training manually intensive; ontology-based harmonization is a proposed future direction

## Evidence

- [readme] README: loss weighting rationale: [evidence span withheld — anchor status: external_doc]
- [readme] README: preprocessing requirements: [evidence span withheld — anchor status: external_doc]
