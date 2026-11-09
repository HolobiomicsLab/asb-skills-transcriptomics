---
name: tabnet-deep-learning-training
description: Use when an annotated single-cell RNA-seq dataset (AnnData .h5ad, cell
  x gene, log1p-normalized) needs automated cell-type label transfer via a low-code
  deep-learning classifier such as SIMS.
when_to_use_negative:
- Label transfer across species — the authors limited validation to samples within
  the same species because neuronal subtypes diverge significantly between species
- The dataset lacks existing annotations — SIMS is supervised and relies on prior
  normalization, clustering, differential expression, and expert annotation
- Speed-critical high-throughput pipelines — SIMS has a consistent runtime but is
  not the fastest benchmarked method (distillation/pruning deferred to future work)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Low-code TabNet-based pipeline for training and running the single-cell label-transfer
    classifier
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying attention-based deep-learning architecture optimized for large-scale
    tabular data (>45,000 gene features)
- name: PyTorch Lightning
  role: Framework used to configure the trainer, EarlyStopping, LearningRateMonitor,
    and logging
- name: Scanpy
  role: Preprocessing of AnnData (filter_cells, filter_genes, normalize_total, log1p,
    scale) prior to training/inference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:840422e3bd0f6a4a73d48d29a1ce69bdd9c6bd6bdb6c51df2e70bf8062d32ecf
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# tabnet-deep-learning-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a TabNet-based SIMS classifier on single-cell RNA-seq expression matrices (AnnData .h5ad, cell x gene) to transfer cell-type labels, with loss weighted inversely by label frequency, EarlyStopping on val_loss (patience=50), and learning-rate monitoring over up to 100 epochs.

## When to use

Use when an annotated single-cell RNA-seq dataset (AnnData .h5ad, cell x gene, log1p-normalized) needs automated cell-type label transfer via a low-code deep-learning classifier such as SIMS.

## When NOT to use

- Label transfer across species — the authors limited validation to samples within the same species because neuronal subtypes diverge significantly between species
- The dataset lacks existing annotations — SIMS is supervised and relies on prior normalization, clustering, differential expression, and expert annotation
- Speed-critical high-throughput pipelines — SIMS has a consistent runtime but is not the fastest benchmarked method (distillation/pruning deferred to future work)

## Inputs

- Annotated AnnData .h5ad expression matrix (cell x gene) with a 'class_label' metadata column
- Pretrained SIMS model checkpoint (.ckpt) for inference or fine-tuning
- Optional unlabeled AnnData matrix for prediction after label transfer

## Outputs

- Trained SIMS/TabNet model checkpoint
- Per-cell predicted labels with softmax probabilities (predict DataFrame columns pred_0/prob_0, ...)
- Accuracy and Macro F1 metrics on held-out splits (e.g. 98.0% accuracy, 0.974 Macro F1 on the cortex 80/20 split)
- Explainability matrix from sims.explain()

## How to apply

Load the annotated expression matrix as AnnData (e.g. the Allen human adult cerebral cortex single-nuclei dataset, 49,495 nuclei, 'class_label' column), lightly filter (min_genes=100, min_cells=3), then normalize_total, log1p, and scale before training. Initialize SIMS(data=adata, class_label='class_label'), call setup_model(n_a=64, n_d=64, weights=sims.weights) so the loss is weighted inversely by label frequency to handle class imbalance, and configure the PyTorch Lightning trainer with EarlyStopping(monitor='val_loss', patience=50), LearningRateMonitor(logging_interval='epoch'), and max_epochs=100 before sims.train(). Evaluate on a held-out 20% split with accuracy and Macro F1 — the cortex benchmark reaches ~98.0% accuracy and Macro F1 ~0.974 — and for donor generalization run trans-sample train/test splits (accuracy expected in the 93.1%–95.8% range). For inference on unlabeled data, load the saved checkpoint with SIMS(weights_path=...) and run predict(unlabeled_data, top_k=1) after applying identical preprocessing.

## Related tools

- **SIMS** (Low-code TabNet-based pipeline for training and running the single-cell label-transfer classifier) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying attention-based deep-learning architecture optimized for large-scale tabular data (>45,000 gene features))
- **PyTorch Lightning** (Framework used to configure the trainer, EarlyStopping, LearningRateMonitor, and logging)
- **Scanpy** (Preprocessing of AnnData (filter_cells, filter_genes, normalize_total, log1p, scale) prior to training/inference)

## Examples

```
sims = SIMS(data=adata, class_label='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.setup_trainer(callbacks=[EarlyStopping(monitor='val_loss', patience=50), LearningRateMonitor(logging_interval='epoch')], max_epochs=100); sims.train()
```

## Evaluation signals

- 80/20 within-dataset split on the human adult cortex single-nuclei data reaches ~98.0% accuracy and Macro F1 ~0.974
- Trans-sample (train one donor, test other two) accuracies fall in the 93.1%–95.8% range reported in Figure 4 and Tables S2-S8
- Predict DataFrame schema contains pred_0/prob_0 (and higher-k) columns with one row per input cell
- Input matrix verified as cell x gene and log1p-normalized (same preprocessing as training data) before predict()
- EarlyStopping callback active on val_loss with patience=50; checkpoints saved each epoch

## Limitations

- Trans-species label transfer is untested; validation limited to same-species samples
- Supervised model depends on quality of pre-existing annotations; sequencing technology or incorrect normalization can produce misnamed clusters
- Requires arbitrarily many cells per label and struggles conceptually when accuracy degrades as cell-type count grows (motivation for frequency-weighted loss)
- Data must be log1p-normalized and oriented cell x gene; inference data must be preprocessed identically to training data
- Metadata (sequencing technology, cell line) not yet integrated into the model; ontology-based naming harmonization only proposed for future work

## Evidence

- [readme] README predict output schema: [evidence span withheld — anchor status: external_doc]
