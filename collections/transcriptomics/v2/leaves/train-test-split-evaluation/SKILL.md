---
name: train-test-split-evaluation
description: Use when evaluating a supervised cell-type classifier on annotated AnnData
  single-cell/nuclei data where you need quantified accuracy on held-out cells and
  cross-sample generalization bounds.
when_to_use_negative:
- 'Trans-species label transfer: the authors limited validation to samples within
  the same species because neuronal subtypes diverge significantly between species'
- Data lacking pre-existing annotations — SIMS is supervised and requires normalized,
  clustered, differentially-expressed and expert-annotated labels; annotation errors
  propagate to the classifier
- Cases requiring the fastest classifier — the authors note SIMS 'is not the fastest
  of the methods we benchmarked'
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Low-code TabNet-based classification pipeline whose train() / predict() methods
    are used for the 80/20 split and donor-holdout experiments
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying model architecture SIMS is built on; configured with n_a=64, n_d=64
    for tabular single-cell expression classification
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Trainer framework supplying EarlyStopping on val_loss (patience=50), LearningRateMonitor,
    and checkpointing for up to 100 epochs
  repo: https://github.com/braingeneers/SIMS
- name: Scanpy
  role: Preprocessing (filter_cells, filter_genes, normalize_total, log1p, scale)
    of the .h5ad expression matrix before training and inference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:f09fe7256de5e768acdb2b662f8cc9459c5e6f22735622eb3741af23ac23098c
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Train-Test Split Evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark SIMS label-transfer accuracy by holding out a fraction of an annotated single-nuclei dataset (e.g., 20% of the human adult cerebral cortex dataset) and measuring held-out accuracy and Macro F1, plus trans-sample (donor-holdout) generalization.

## When to use

Use when evaluating a supervised cell-type classifier on annotated AnnData single-cell/nuclei data where you need quantified accuracy on held-out cells and cross-sample generalization bounds.

## When NOT to use

- Trans-species label transfer: the authors limited validation to samples within the same species because neuronal subtypes diverge significantly between species
- Data lacking pre-existing annotations — SIMS is supervised and requires normalized, clustered, differentially-expressed and expert-annotated labels; annotation errors propagate to the classifier
- Cases requiring the fastest classifier — the authors note SIMS 'is not the fastest of the methods we benchmarked'

## Inputs

- Annotated AnnData (.h5ad) single-nuclei expression matrix in cell x gene form with a 'class_label' observation column (e.g., Allen human adult cerebral cortex, 49,495 nuclei)
- Donor/sample metadata (e.g., H200.1023, H200.1025, H200.1030) for defining trans-sample splits

## Outputs

- Held-out (20%) accuracy and Macro F1 score (expected ~98.0% accuracy and Macro F1 ~0.974)
- Trans-sample accuracies for the three train-one-donor/test-two-donors combinations (expected 93.1%-95.8%)
- Trained SIMS model checkpoint (saved per epoch by model checkpointing)

## How to apply

Load the annotated dataset (e.g., 49,495 Allen human adult cortex nuclei from donors H200.1023, H200.1025, H200.1030 with 'class_label' annotations) as AnnData, preprocess with Scanpy (filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale), and initialize SIMS with the labels key. Configure the TabNet-based model with n_a=64, n_d=64 and class-frequency loss weighting (weights=sims.weights) to handle imbalanced labels. Train on the 80% split using a PyTorch Lightning trainer with EarlyStopping(monitor='val_loss', patience=50), LearningRateMonitor per epoch, and max_epochs=100, then compute accuracy and Macro F1 on the held-out 20%. For trans-sample generalization, repeat training with one donor held out entirely (train on one donor, test on the other two). The rationale is that within-split accuracy validates the model learns discriminative features, while donor-holdout tests robustness to inter-sample technical variability.

## Related tools

- **SIMS** (Low-code TabNet-based classification pipeline whose train() / predict() methods are used for the 80/20 split and donor-holdout experiments) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying model architecture SIMS is built on; configured with n_a=64, n_d=64 for tabular single-cell expression classification) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Trainer framework supplying EarlyStopping on val_loss (patience=50), LearningRateMonitor, and checkpointing for up to 100 epochs) — https://github.com/braingeneers/SIMS
- **Scanpy** (Preprocessing (filter_cells, filter_genes, normalize_total, log1p, scale) of the .h5ad expression matrix before training and inference)

## Examples

```
from scsims import SIMS; sims = SIMS(adata=adata, labels_key='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.setup_trainer(callbacks=[EarlyStopping(monitor='val_loss', patience=50), LearningRateMonitor(logging_interval='epoch')], max_epochs=100); sims.train()
```

## Evaluation signals

- Within-dataset 80/20 evaluation reaches ~98.0% accuracy with Macro F1 ~0.974 (Figure 4, Tables S2-S8)
- Trans-sample (train one donor, test two) accuracies fall in the 93.1%-95.8% range
- Predictions are returned as a pandas DataFrame with canonical pred_0/prob_0 column names (top_k=1 for the best label)
- Data must be log1p-normalized (and scaled) identically for training and inference; mismatched preprocessing invalidates comparison
- EarlyStopping on val_loss with patience=50 within max_epochs=100 confirms convergence on the training split

## Limitations

- Validation restricted to same-species samples; trans-species prediction remains untested because neuronal subtypes diverge significantly between species
- As a supervised model, accuracy inherits errors from upstream annotations (normalization, clustering, differential expression) and technical factors such as sequencing technology
- Combining datasets from different labs for training is manually intensive due to lack of naming conventions; ontology-based harmonization is only proposed future work

## Evidence

- [results] SIMS trained on 80% of the human adult cerebral cortex single-nuclei dataset and tested on the remaining 20% achieved 98.0% accuracy and a Macro F1 score of 0.974.: [evidence span withheld — anchor status: not_found]
- [readme] README preprocessing requirement: [evidence span withheld — anchor status: external_doc]
