---
name: cross-age-label-transfer-evaluation
description: Use when evaluating supervised cell-type label transfer across developmental
  ages or samples in scRNA-seq, e.g. testing whether a classifier trained on one age
  generalizes to another.
when_to_use_negative:
- When the query tissue is from a different species than the training annotations
  — the authors limited SIMS to within-species transfer because neuronal subtypes
  diverge significantly between species
- When no pre-existing expert annotations are available for training, since SIMS is
  fully supervised and relies on prior normalization, clustering, and differential-expression-derived
  labels
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Low-code supervised classifier pipeline (scsims package) used to instantiate,
    train, and evaluate label transfer on the developing cortex data
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying model architecture (n_a=64, n_d=64) optimized for large tabular
    cell x gene matrices
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Trainer framework providing EarlyStopping, LearningRateMonitor callbacks and
    WandbLogger integration
  repo: https://github.com/braingeneers/SIMS
- name: Scanpy
  role: 'Preprocessing of AnnData: filter_cells(min_genes=100), filter_genes(min_cells=3),
    normalize_total, log1p, scale'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:8fe37a69ca605858dfa7f8c370ad20b37117a36d9124c3b3384eaf54a56b7a18
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
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# cross-age-label-transfer-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark SIMS (TabNet-based single-cell classifier) label transfer on the Di Bella developing mouse cortex dataset, evaluating an E18+P1 combined-age split (84.2% accuracy / 0.791 Macro F1 over 17 cell types) versus a cross-age P1-to-E18 transfer (73.6% accuracy / 0.674 Macro F1).

## When to use

Use when evaluating supervised cell-type label transfer across developmental ages or samples in scRNA-seq, e.g. testing whether a classifier trained on one age generalizes to another.

## When NOT to use

- When the query tissue is from a different species than the training annotations — the authors limited SIMS to within-species transfer because neuronal subtypes diverge significantly between species
- When no pre-existing expert annotations are available for training, since SIMS is fully supervised and relies on prior normalization, clustering, and differential-expression-derived labels

## Inputs

- Di Bella mouse cortical development AnnData (.h5ad) expression matrices with a class_label column over 17 cell types
- E18+P1 combined-age train/test split definition
- P1 (two samples) training subset and E18 (two samples) held-out test subset

## Outputs

- Held-out accuracy and Macro F1 for the E18+P1 combined split (~84.2% / 0.791)
- Cross-age accuracy and Macro F1 for P1->E18 transfer (~73.6% / 0.674)
- metrics_table.csv containing both splits' metrics
- Trained SIMS/TabNet model checkpoints

## How to apply

Load the Di Bella mouse cortical development AnnData expression matrices (cell x gene, log1p-normalized) and instantiate SIMS with the class_label key. Configure the TabNet model via setup_model(n_a=64, n_d=64, weights=sims.weights), weighting the loss inversely proportional to label frequency to handle the imbalanced 17-cell-type distribution. Set up the PyTorch Lightning trainer with EarlyStopping(monitor='val_loss', patience=50), LearningRateMonitor(logging_interval='epoch'), a Weights and Biases logger, and max_epochs=100. Train on the E18+P1 combined-age split and record held-out accuracy and Macro F1 (target ~84.2% / 0.791), then retrain on the two P1 samples and evaluate on the two held-out E18 samples (~73.6% / 0.674). Inspect misclassifications: errors follow developmental trajectories rather than being random, which is the qualitative signature of cross-age transfer. Write both splits' metrics to metrics_table.csv and compare against Figure 5 and Tables S9-S10.

## Related tools

- **SIMS** (Low-code supervised classifier pipeline (scsims package) used to instantiate, train, and evaluate label transfer on the developing cortex data) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying model architecture (n_a=64, n_d=64) optimized for large tabular cell x gene matrices) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Trainer framework providing EarlyStopping, LearningRateMonitor callbacks and WandbLogger integration) — https://github.com/braingeneers/SIMS
- **Scanpy** (Preprocessing of AnnData: filter_cells(min_genes=100), filter_genes(min_cells=3), normalize_total, log1p, scale)

## Examples

```
from scsims import SIMS; sims = SIMS(data=adata, class_label='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.train()
```

## Evaluation signals

- E18+P1 combined-age held-out accuracy approximates 84.2% with Macro F1 ~0.791 across 17 cell types
- P1->E18 cross-age accuracy approximates 73.6% with Macro F1 ~0.674
- Confusion analysis shows misclassifications follow developmental trajectories rather than random label swaps
- Training used loss weights inversely proportional to label frequency (sims.weights) and val_loss-based early stopping with patience 50
- Metrics reproduce the values reported in Figure 5 and Tables S9-S10 when written to metrics_table.csv

## Limitations

- Cross-age transfer degrades accuracy (~10.6 points) relative to the combined-age split, so age mismatch is a real failure mode
- SIMS has only been validated within the same species; trans-species prediction remains untested
- As a supervised model, SIMS depends on pre-existing annotations, and incorrect normalization or sequencing-technology differences can propagate errors into labels

## Evidence

- [methods] Combined-split performance target: [evidence span withheld — anchor status: not_found]
- [methods] Cross-age transfer performance: [evidence span withheld — anchor status: not_found]
- [readme] Model setup with frequency weighting: [evidence span withheld — anchor status: external_doc]
- [readme] Data normalization requirement: [evidence span withheld — anchor status: external_doc]
