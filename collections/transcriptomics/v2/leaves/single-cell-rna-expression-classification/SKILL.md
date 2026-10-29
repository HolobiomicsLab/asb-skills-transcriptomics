---
name: single-cell-rna-expression-classification
description: Use when you have labeled cell-by-gene single-cell RNA-seq matrices (AnnData
  .h5ad or CSV) with a 'class_label' annotation and need automated label transfer
  to unlabeled cells, especially with many cell types or imbalanced labels.
when_to_use_negative:
- Prediction across species is desired — the authors 'limited our work to samples
  within the same species' because neuronal subtypes diverge significantly between
  species
- No pre-existing cell-type annotations are available for training, since SIMS is
  supervised and relies on labels derived from normalization, clustering, and expert
  annotation
- Metadata such as sequencing technology or cell line must be integrated into the
  model — this is a proposed future direction and is not implemented
edam_operation: http://edamontology.org/operation_2991
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Low-code pipeline providing the SIMS class (setup_model, setup_trainer, train,
    predict, explain) wrapping TabNet for single-cell label transfer
  repo: github.com/jlehrer1/sims
- name: TabNet
  role: Underlying deep-learning architecture optimized for large tabular (cell x
    gene) data; configured with n_a=64, n_d=64 and class-frequency loss weights
  repo: github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Training framework used to configure the pl.Trainer with EarlyStopping (val_loss,
    patience=50), LearningRateMonitor, and WandbLogger callbacks
- name: Scanpy
  role: Used to load .h5ad AnnData and perform light filtering and normalization (filter_cells
    min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:8524d1668d9aa87975c1cb268d87add71f41ccab4d3ee7a9cd0a1ecd6e94749e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jlehrer1/sims
  tools:
  - PyTorch Lightning
  - SIMS
  - Scanpy
  - TabNet
  edam_operation: http://edamontology.org/operation_2991
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# single-cell-rna-expression-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains SIMS, a TabNet-based deep-learning classifier implemented in PyTorch Lightning, to transfer cell-type labels from annotated single-cell RNA-seq data (e.g., AnnData .h5ad) to unlabeled cells, with accuracy/F1 evaluation and attention-based explainability.

## When to use

Use when you have labeled cell-by-gene single-cell RNA-seq matrices (AnnData .h5ad or CSV) with a 'class_label' annotation and need automated label transfer to unlabeled cells, especially with many cell types or imbalanced labels.

## When NOT to use

- Prediction across species is desired — the authors 'limited our work to samples within the same species' because neuronal subtypes diverge significantly between species
- No pre-existing cell-type annotations are available for training, since SIMS is supervised and relies on labels derived from normalization, clustering, and expert annotation
- Metadata such as sequencing technology or cell line must be integrated into the model — this is a proposed future direction and is not implemented

## Inputs

- AnnData (.h5ad) or CSV cell x gene expression matrix with log1p normalization
- Label file or .obs column containing 'class_label' cell-type annotations for training data
- Unlabeled .h5ad matrix for inference, preprocessed identically to the training data

## Outputs

- Trained SIMS/TabNet model checkpoint (.ckpt) per epoch via model checkpointing
- pandas DataFrame of top-k predicted cell-type labels and softmax probabilities (pred_0/prob_0, pred_1/prob_1, pred_2/prob_2)
- Gene-level explainability matrix for interpretability of predictions
- Held-out accuracy and Macro F1 metrics (e.g., written to metrics_table.csv)

## How to apply

Load the cell-by-gene AnnData expression matrices and instantiate the classifier with SIMS(adata=adata, labels_key='class_label'). Preprocess with light filtering (min_genes=100 per cell, min_cells=3 per gene) followed by normalize_total, log1p, and scale so data are log1p normalized for both training and inference. Configure the TabNet model with sims.setup_model(n_a=64, n_d=64, weights=sims.weights), weighting the loss inversely by label frequency to handle imbalanced cell-type distributions, then set up a PyTorch Lightning trainer with EarlyStopping(monitor='val_loss', patience=50), LearningRateMonitor(logging_interval='epoch'), a Weights & Biases logger, and max_epochs=100. Call sims.train(), then run sims.predict(unlabeled_data, top_k=3) to obtain pred_0/prob_0 columns and sims.explain() for the gene-level explainability matrix. Judge success against held-out accuracy and Macro F1 comparable to the paper's benchmarks (e.g., 84.2% accuracy / 0.791 Macro F1 over 17 brain cell types on an E18+P1 combined split).

## Related tools

- **SIMS** (Low-code pipeline providing the SIMS class (setup_model, setup_trainer, train, predict, explain) wrapping TabNet for single-cell label transfer) — github.com/jlehrer1/sims
- **TabNet** (Underlying deep-learning architecture optimized for large tabular (cell x gene) data; configured with n_a=64, n_d=64 and class-frequency loss weights) — github.com/braingeneers/SIMS
- **PyTorch Lightning** (Training framework used to configure the pl.Trainer with EarlyStopping (val_loss, patience=50), LearningRateMonitor, and WandbLogger callbacks)
- **Scanpy** (Used to load .h5ad AnnData and perform light filtering and normalization (filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale))

## Examples

```
sims = SIMS(adata=adata, labels_key='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.train()
```

## Evaluation signals

- Held-out test accuracy and Macro F1 match published benchmarks, e.g., ~84.2% accuracy / 0.791 Macro F1 across 17 cell types on the E18+P1 combined-age split
- Cross-sample transfer (e.g., training on two P1 samples, testing on two E18 samples) yields ~73.6% accuracy / 0.674 Macro F1, consistent with Figure 5 and Tables S9-S10
- predict() returns a pandas DataFrame with canonical pred_0/prob_0 column names, one row per input cell, with prob_0 summing to ~1 across top-k softmax probabilities
- Misclassified cells follow biologically sensible developmental trajectories rather than being random errors
- Model checkpoints are written every training epoch and early stopping triggers on val_loss with patience=50

## Limitations

- SIMS is supervised and depends on pre-existing annotations; technical factors such as sequencing technology or incorrect normalization may lead to misnamed clusters that propagate through label transfer
- Trans-species label transfer is untested because neuronal subtypes diverge significantly between species
- Accuracy degrades under cross-age/cross-sample distribution shift (73.6% vs 84.2% for the P1->E18 transfer), though errors still follow developmental trajectories
- SIMS is not the fastest benchmarked classifier; speed optimizations via model distillation are deferred to future work
- Lack of cell-naming conventions across labs makes combining datasets from different sources for training manually intensive

## Evidence

- [readme] Normalization requirement: [evidence span withheld — anchor status: external_doc]
