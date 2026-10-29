---
name: tabnet-based-cell-type-classification
description: Use when you have log1p-normalized cell x gene scRNA-seq data in AnnData
  (.h5ad) format with an annotated label column (e.g. 'class_label') and need accurate
  cell-type classification or label transfer, including on imbalanced datasets or
  small training sets.
when_to_use_negative:
- Labels must be transferred across species — the authors limited validation to samples
  within the same species because neuronal subtypes diverge significantly between
  species
- No pre-existing annotations are available for any training data — SIMS is a supervised
  model relying on labels produced by normalization, clustering, differential expression,
  and expert annotation
- Training speed is the critical constraint — the authors note SIMS 'is not the fastest
  of the methods we benchmarked', with distillation/pruning deferred to future work
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Low-code TabNet-based pipeline for training the cell-type classifier and running
    predict/explain
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying deep-learning architecture optimized for large-scale tabular data
    (handles >45,000 gene features)
- name: PyTorch Lightning
  role: Framework providing the pl.Trainer, EarlyStopping, LearningRateMonitor callbacks
    and WandbLogger used to configure and run training
- name: Scanpy
  role: Preprocessing (filter_cells, filter_genes, normalize_total, log1p, scale)
    and downstream plotting of predictions in the Python ecosystem
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:09a27f972b05d3abf3a9f4531f67ad66106715087e902fde057531baba543763
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tabnet-based-cell-type-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train SIMS, a TabNet-based deep-learning classifier implemented in PyTorch Lightning, to transfer cell-type labels onto single-cell RNA-seq data from a .h5ad AnnData object with pre-annotated labels, achieving high accuracy even with small training sets (<3,500 cells).

## When to use

Use when you have log1p-normalized cell x gene scRNA-seq data in AnnData (.h5ad) format with an annotated label column (e.g. 'class_label') and need accurate cell-type classification or label transfer, including on imbalanced datasets or small training sets.

## When NOT to use

- Labels must be transferred across species — the authors limited validation to samples within the same species because neuronal subtypes diverge significantly between species
- No pre-existing annotations are available for any training data — SIMS is a supervised model relying on labels produced by normalization, clustering, differential expression, and expert annotation
- Training speed is the critical constraint — the authors note SIMS 'is not the fastest of the methods we benchmarked', with distillation/pruning deferred to future work

## Inputs

- AnnData .h5ad (or CSV) expression matrix in cell x gene orientation
- Label file/column with a common 'class_label' column containing cell-type annotations
- Pretrained SIMS checkpoint (.ckpt weights file) for inference-only use
- Optional GSEA JSON gene pathway file (for the downstream app analysis)

## Outputs

- Trained SIMS/TabNet model checkpoints (.ckpt) saved each training epoch
- Per-cell predicted labels and softmax probabilities as a pandas DataFrame (pred_0..pred_n, prob_0..prob_n columns)
- Gene-level explainability matrix from sims.explain()
- Accuracy and Macro F1 metrics per held-out fold when evaluated by cross-validation

## How to apply

Load the AnnData object (cell x gene matrix) and preprocess it with Scanpy: filter cells with min_genes=100 and genes with min_cells=3, then apply sc.pp.normalize_total, sc.pp.log1p, and sc.pp.scale. Initialize SIMS with SIMS(adata=adata, labels_key='class_label') (newer scsims versions use class_label='class_label' with data=adata) and set up the TabNet model with setup_model(n_a=64, n_d=64, weights=sims.weights), where weights inversely weight the loss by label frequency to handle rare cell types. Configure the PyTorch Lightning trainer with EarlyStopping monitoring 'val_loss' (patience=50), a LearningRateMonitor with logging_interval='epoch', and max_epochs=100, then call sims.train(); checkpoints are saved every epoch. For inference on unlabeled data, load the checkpoint via SIMS(weights_path='model.ckpt'), preprocess the new data identically to the training data, and run sims.predict(unlabeled_data, top_k=1) to obtain 'pred_0' labels and 'prob_0' softmax confidences. Optionally run sims.explain() to obtain a gene-level explainability matrix supporting interpretation of the classifier.

## Related tools

- **SIMS** (Low-code TabNet-based pipeline for training the cell-type classifier and running predict/explain) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying deep-learning architecture optimized for large-scale tabular data (handles >45,000 gene features))
- **PyTorch Lightning** (Framework providing the pl.Trainer, EarlyStopping, LearningRateMonitor callbacks and WandbLogger used to configure and run training)
- **Scanpy** (Preprocessing (filter_cells, filter_genes, normalize_total, log1p, scale) and downstream plotting of predictions in the Python ecosystem)

## Examples

```
sims = SIMS(data=adata, class_label='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.setup_trainer(callbacks=[EarlyStopping(monitor='val_loss', patience=50), LearningRateMonitor(logging_interval='epoch')], max_epochs=100); sims.train()
```

## Evaluation signals

- In 5-fold cross-validation across the five benchmark datasets (PBMC68K, Tucker heart, Stewart kidney, Krasnow lung, Han landscape), SIMS accuracy and Macro F1 should be on par with or exceed scANVI, Seurat, scNym, SingleR, and SciBet baselines (Table S1 / Figure 1A-B)
- Training reaches high accuracy even with small training sets (<3,500 cells) in the adult mammalian brain application
- predict() on new data returns a pandas DataFrame with canonical 'pred_0'/'prob_0' columns (not the legacy broken 'first_pred'/'first_prob')
- Trainer callbacks are active: EarlyStopping monitors 'val_loss' with patience=50 and LearningRateMonitor logs per epoch
- Input matrices are verified to be cell x gene (M cells x N genes), log1p-normalized, and the same preprocessing pipeline applied to train and inference data

## Limitations

- Trans-species label transfer is untested; validation was limited to samples within the same species
- As a supervised model, SIMS inherits errors from upstream annotation steps — sequencing technology or incorrect normalization can lead to misnamed clusters
- SIMS is not the fastest benchmarked method; model distillation via student-teacher methods is future work
- Metadata such as sequencing technology and cell line is not yet integrated into the model
- Lack of neuroscience naming conventions makes combining datasets from different labs for training manually intensive; ontology-based harmonization is only a proposed future direction
- scsims requires Python 3.10 or newer (3.11 recommended); v4 changes predict() call sites to pred_0/prob_0 column names

## Evidence

- [readme] The data should be log1p normalized before model training and model inference.: [evidence span withheld — anchor status: external_doc]
