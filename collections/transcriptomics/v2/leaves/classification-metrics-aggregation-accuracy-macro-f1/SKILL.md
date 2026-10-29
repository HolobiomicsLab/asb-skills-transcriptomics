---
name: classification-metrics-aggregation-accuracy-macro-f1
description: Use when comparing cell-type classification methods on pre-annotated
  single-cell RNA-seq datasets and you need per-fold held-out accuracy and Macro F1
  aggregated into a method-by-dataset benchmark table.
when_to_use_negative:
- You need trans-species label transfer — the article limits its validation to samples
  within the same species
- Your goal is runtime/speed benchmarking rather than classification quality — SIMS
  is explicitly not the fastest of the benchmarked methods
- The input data has no pre-existing cell annotations, since SIMS is a supervised
  classifier relying on pre-existing annotations
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: SIMS
  role: TabNet-based classifier under evaluation; configured with n_a=64, n_d=64 and
    class-frequency loss weighting, trained 100 epochs with EarlyStopping patience=50
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Training framework hosting the EarlyStopping and LearningRateMonitor callbacks
    used in each SIMS cross-validation fold
- name: Scanpy
  role: Reading .h5ad AnnData inputs and light preprocessing (filter_cells min_genes=100,
    filter_genes min_cells=3, normalize_total, log1p, scale) before training and inference
- name: scANVI
  role: Baseline label transfer method trained on the same 5-fold splits with default
    recommended settings
- name: Seurat
  role: Baseline label transfer method trained on the same 5-fold splits with default
    recommended settings
- name: SingleR
  role: Baseline label transfer method trained on the same 5-fold splits with default
    recommended settings
- name: SciBet
  role: Baseline label transfer method trained on the same 5-fold splits with default
    recommended settings
- name: scNym
  role: Baseline label transfer method trained on the same 5-fold splits with default
    recommended settings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:5d5d20cb165f41fac9352783684edb524684d589ab41a8916e7ba502d7ca4337
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
  - SciBet
  - Seurat
  - SingleR
  - scANVI
  - scNym
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# classification-metrics-aggregation-accuracy-macro-f1

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark single-cell label transfer classifiers (SIMS vs scANVI, Seurat, Scnym, SingleR, SciBet) using 5-fold cross-validation with held-out fold accuracy and Macro F1 aggregated into a comparison table, mirroring Figure 1A-B and Table S1 of the SIMS paper.

## When to use

Use when comparing cell-type classification methods on pre-annotated single-cell RNA-seq datasets and you need per-fold held-out accuracy and Macro F1 aggregated into a method-by-dataset benchmark table.

## When NOT to use

- You need trans-species label transfer — the article limits its validation to samples within the same species
- Your goal is runtime/speed benchmarking rather than classification quality — SIMS is explicitly not the fastest of the benchmarked methods
- The input data has no pre-existing cell annotations, since SIMS is a supervised classifier relying on pre-existing annotations

## Inputs

- Five benchmark single-cell RNA-seq datasets as AnnData (.h5ad) objects with pre-annotated cell labels in a 'class_label' column: PBMC68K, Tucker heart, Stewart kidney, Krasnow lung, Han landscape
- Trained predictions from SIMS and five baseline classifiers (scANVI, Seurat, scNym, SingleR, SciBet) on held-out folds
- Fold assignments from 5-fold cross-validation splits

## Outputs

- benchmark_comparison.csv containing accuracy and macro_f1 fields for all 6 methods across the 5 datasets
- Per-fold metric values aggregated into a comparison table mirroring Figure 1A-B and Table S1

## How to apply

Load each benchmark dataset (PBMC68K, Tucker heart, Stewart kidney, Krasnow lung, Han landscape) as AnnData with pre-annotated labels, then run 5-fold cross-validation for every method-dataset pair, holding out one fold per split and predicting labels for the held-out cells. Train SIMS with the TabNet-based setup (setup_model with n_a=64, n_d=64, weights=sims.weights; EarlyStopping on 'val_loss' with patience=50; max_epochs=100) and train each baseline (scANVI, Seurat, scNym, SingleR, SciBet) on the same splits with their default recommended settings. For every held-out fold compute accuracy and Macro F1 — Macro F1 is essential because class-frequency loss weighting in SIMS targets rare cell types and imbalanced label distributions, so per-class performance must be averaged unweighted. Aggregate the per-fold metrics into a comparison table mirroring Figure 1A-B and Table S1, checking that SIMS is on par with or exceeds the baselines.

## Related tools

- **SIMS** (TabNet-based classifier under evaluation; configured with n_a=64, n_d=64 and class-frequency loss weighting, trained 100 epochs with EarlyStopping patience=50) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Training framework hosting the EarlyStopping and LearningRateMonitor callbacks used in each SIMS cross-validation fold)
- **Scanpy** (Reading .h5ad AnnData inputs and light preprocessing (filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale) before training and inference)
- **scANVI** (Baseline label transfer method trained on the same 5-fold splits with default recommended settings)
- **Seurat** (Baseline label transfer method trained on the same 5-fold splits with default recommended settings)
- **SingleR** (Baseline label transfer method trained on the same 5-fold splits with default recommended settings)
- **SciBet** (Baseline label transfer method trained on the same 5-fold splits with default recommended settings)
- **scNym** (Baseline label transfer method trained on the same 5-fold splits with default recommended settings)

## Examples

```
sims = SIMS(data=adata, class_label='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.setup_trainer(callbacks=[EarlyStopping(monitor='val_loss', patience=50), LearningRateMonitor(logging_interval='epoch')], max_epochs=100); sims.train()
```

## Evaluation signals

- benchmark_comparison.csv exists and contains accuracy and macro_f1 fields for all 6 methods across the 5 datasets
- SIMS accuracy values are on par with or exceed the baseline classifiers as reported in Table S1
- Every method-dataset pair has 5 per-fold metric values from the 5-fold cross-validation, aggregated consistently
- Macro F1 is computed with unweighted per-class averaging so rare cell types (handled via sims.weights class-frequency loss weighting) are not masked by majority classes
- Predictions are made only on held-out fold cells never seen in training for that split

## Limitations

- SIMS has only been validated for label transfer within the same species; trans-species prediction remains untested
- SIMS is not the fastest of the benchmarked methods; speed optimizations such as model distillation are deferred to future work
- As a supervised model, SIMS relies on pre-existing annotations that can introduce errors from technical factors such as sequencing technology or incorrect normalization
- Lack of naming conventions across labs makes combining datasets for training manually intensive, complicating multi-dataset benchmarking

## Evidence

- [results] SIMS consistently outperformed the majority of label transfer methods in both accuracy and Macro F1 score: [evidence span withheld — anchor status: not_found]
- [intro] We show that SIMS either outperforms or is on par with state-of-the-art single-cell classifiers.: "We show that SIMS either outperforms or is on par with state-of-the-art single-cell classifiers."
- [methods] weighting loss by label freq: "sims.setup_model(n_a=64, n_d=64, weights=sims.weights)  # weighting loss by label freq"
