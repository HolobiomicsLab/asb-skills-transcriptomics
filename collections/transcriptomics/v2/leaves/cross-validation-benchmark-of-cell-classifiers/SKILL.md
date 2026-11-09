---
name: cross-validation-benchmark-of-cell-classifiers
description: Use when benchmarking automated single-cell cell-type classifiers (e.g.,
  SIMS vs scANVI, Seurat, Scnym, SingleR, SciBet) on pre-annotated datasets to quantify
  accuracy and Macro F1 under cross-validation.
when_to_use_negative:
- Data are unlabeled with no trusted class_label annotations, since SIMS is a supervised
  method requiring pre-existing expert annotations
- Goal is trans-species label transfer — the article limits evaluation to samples
  within the same species
- The input needs the fastest possible classifier — SIMS 'is not the fastest of the
  methods we benchmarked'
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SIMS
  role: TabNet-based label transfer classifier being benchmarked; trained with setup_model(n_a=64,
    n_d=64) and loss weighting by label frequency
  repo: github.com/jlehrer1/sims
- name: PyTorch Lightning
  role: Training framework used to configure the trainer with EarlyStopping(monitor='val_loss',
    patience=50) and LearningRateMonitor
- name: TabNet
  role: Underlying model architecture SIMS is built on, optimized for large-scale
    tabular (cell x gene) data
- name: Scanpy
  role: Used to read .h5ad data and apply light filtering (filter_cells min_genes=100,
    filter_genes min_cells=3), normalize_total, log1p, and scale before training/inference
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:f81bb3e9da77b4c7171da23c6b730a954ca164e647f7150741846317acb22585
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-validation-benchmark-of-cell-classifiers

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A 5-fold cross-validation benchmarking protocol comparing the SIMS TabNet-based single-cell label transfer tool against scANVI, Seurat, Scnym, SingleR, and SciBet on accuracy and Macro F1 across five diverse scRNA-seq datasets.

## When to use

Use when benchmarking automated single-cell cell-type classifiers (e.g., SIMS vs scANVI, Seurat, Scnym, SingleR, SciBet) on pre-annotated datasets to quantify accuracy and Macro F1 under cross-validation.

## When NOT to use

- Data are unlabeled with no trusted class_label annotations, since SIMS is a supervised method requiring pre-existing expert annotations
- Goal is trans-species label transfer — the article limits evaluation to samples within the same species
- The input needs the fastest possible classifier — SIMS 'is not the fastest of the methods we benchmarked'

## Inputs

- Five benchmark scRNA-seq datasets (PBMC68K, Tucker heart, Stewart kidney, Krasnow lung, Han landscape) as .h5ad AnnData objects in cell x gene orientation
- Cell-type annotation column (class_label) in adata.obs for each dataset
- Trained SIMS model checkpoints (e.g., .ckpt weights_path) and equivalent trained models for scANVI, Seurat, scNym, SingleR, SciBet

## Outputs

- benchmark_comparison.csv with per-fold accuracy and macro_f1 for all 6 methods across the 5 datasets
- Aggregated comparison table mirroring Figure 1A-B and Table S1

## How to apply

Load each benchmark dataset (PBMC68K, Tucker heart, Stewart kidney, Krasnow lung, Han landscape) as an AnnData (.h5ad) object with cell-by-gene matrices and pre-annotated labels in a 'class_label' column. For SIMS, initialize with SIMS(adata=adata, labels_key='class_label'), call setup_model(n_a=64, n_d=64, weights=sims.weights) to weight the loss inversely by label frequency (helps rare cell types), and configure the PyTorch Lightning trainer with EarlyStopping(monitor='val_loss', patience=50), LearningRateMonitor(logging_interval='epoch'), and max_epochs=100 before sims.train(). Train each baseline (scANVI, Seurat, scNym, SingleR, SciBet) on identical splits with their default recommended settings. Run 5-fold cross-validation for every method-dataset pair, holding out one fold and predicting labels for held-out cells. Compute accuracy and Macro F1 per held-out fold and aggregate into a comparison table mirroring Figure 1A-B and Table S1.

## Related tools

- **SIMS** (TabNet-based label transfer classifier being benchmarked; trained with setup_model(n_a=64, n_d=64) and loss weighting by label frequency) — github.com/jlehrer1/sims
- **PyTorch Lightning** (Training framework used to configure the trainer with EarlyStopping(monitor='val_loss', patience=50) and LearningRateMonitor)
- **TabNet** (Underlying model architecture SIMS is built on, optimized for large-scale tabular (cell x gene) data)
- **Scanpy** (Used to read .h5ad data and apply light filtering (filter_cells min_genes=100, filter_genes min_cells=3), normalize_total, log1p, and scale before training/inference)

## Examples

```
sims = SIMS(adata=adata, labels_key='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.setup_trainer(callbacks=[EarlyStopping(monitor='val_loss', patience=50)], max_epochs=100); sims.train()
```

## Evaluation signals

- benchmark_comparison.csv exists and contains accuracy and macro_f1 fields for all 6 methods across the 5 datasets
- SIMS accuracy values are on par with or exceed the baseline classifiers as reported in Table S1
- Per-fold metrics are aggregated across all 5 folds for every method-dataset pair (30 method-dataset combinations)
- Held-out fold predictions use the same splits for SIMS and all five baselines

## Limitations

- SIMS has only been validated for label transfer within the same species; trans-species prediction remains untested
- As a supervised model, SIMS relies on pre-existing annotations (normalization, clustering, differential expression, expert annotation), which can propagate errors from technical factors
- SIMS is not the fastest of the benchmarked methods; speed optimizations such as model distillation are deferred to future work
- Lack of naming conventions in neuroscience makes combining datasets from different labs for training manually intensive

## Evidence

- [readme] readme: [evidence span withheld — anchor status: external_doc]
