---
name: repeated-explain-value-aggregation
description: Use when you have a trained SIMS/TabNet classifier on single-cell RNA-seq
  data and need to quantify whether its sparse feature masks consistently select the
  same genes for a given cell type across independent explain runs.
when_to_use_negative:
- The model has not been trained yet or training was cut off before convergence —
  explain masks from an underfit model are not comparable across runs
- Input data is raw counts rather than log1p-normalized and scaled; the README requires
  normalize_total, log1p, and scale before model inference
- The goal is trans-species label transfer; the paper limited validation to samples
  within the same species
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: SIMS
  role: Provides the low-code pipeline whose explain() procedure samples the sparse
    TabNet feature-selection masks that are aggregated across repeated runs
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying architecture whose sparse feature masks yield per-gene explain
    values optimized for tabular data with tens of thousands of features
- name: PyTorch Lightning
  role: Runs the training loop with EarlyStopping (val_loss, patience=50) and LearningRateMonitor
    callbacks so a stable checkpoint is available for repeated explain runs
- name: Scanpy
  role: Used for input preprocessing (filter_cells, filter_genes, normalize_total,
    log1p, scale) and for downstream visualization of explainability results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:c702d1a99ad0fd6811a2c7720cdfabfd248887ef54de71f27a978bd341775e58
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# repeated-explain-value-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Repeatedly samples the sparse feature-selection masks produced by a trained SIMS/TabNet model (e.g., 300 explain runs on one cell class) and aggregates them into per-gene mean explain values to assess whether top-ranked gene selection is stable across independent runs.

## When to use

Use when you have a trained SIMS/TabNet classifier on single-cell RNA-seq data and need to quantify whether its sparse feature masks consistently select the same genes for a given cell type across independent explain runs.

## When NOT to use

- The model has not been trained yet or training was cut off before convergence — explain masks from an underfit model are not comparable across runs
- Input data is raw counts rather than log1p-normalized and scaled; the README requires normalize_total, log1p, and scale before model inference
- The goal is trans-species label transfer; the paper limited validation to samples within the same species

## Inputs

- Trained SIMS/TabNet model checkpoint (e.g., .ckpt from PyTorch Lightning trainer)
- Annotated single-cell expression matrix (.h5ad AnnData or .csv, cell x gene, log1p-normalized) with a class_label column
- Target cell-class label (e.g., Pvalb+ interneuron class) for which explain masks are sampled
- Optional: a reference marker-gene set for the target class to validate the recovered gene list

## Outputs

- Per-gene mean explain-value table aggregated over 300 explain runs
- Dispersion index for the top 50 genes (expected ~0.2 for top genes vs ~1e-6 for median genes)
- Ranked top-gene list that can be compared against reported marker sets (e.g., Pvalb+ markers)

## How to apply

Train the SIMS model (TabNet architecture, n_a=64, n_d=64, loss weighted by label frequency via sims.weights) with PyTorch Lightning using EarlyStopping on val_loss (patience=50) and LearningRateMonitor per epoch for up to 100 epochs. After training, run the TabNet explain procedure (sims.explain) 300 times for the target cell class — e.g., Pvalb+ interneurons from the Allen mouse cortex/hippocampus dataset — to sample the sparse feature-selection masks. Aggregate the 300 runs into per-gene mean explain values and compute the dispersion index for the top 50 genes; expect top genes near ~0.2 mean explain value versus ~1e-6 for the median gene. Finally, compare the top-ranked gene set against biologically reported markers for the class (e.g., Satb1, Pvalb, Cck, Gad1, Kcnip, Igf1, Dlx1, Dlx5 for Pvalb+ INs) to confirm the selection is both stable and biologically meaningful.

## Related tools

- **SIMS** (Provides the low-code pipeline whose explain() procedure samples the sparse TabNet feature-selection masks that are aggregated across repeated runs) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying architecture whose sparse feature masks yield per-gene explain values optimized for tabular data with tens of thousands of features)
- **PyTorch Lightning** (Runs the training loop with EarlyStopping (val_loss, patience=50) and LearningRateMonitor callbacks so a stable checkpoint is available for repeated explain runs)
- **Scanpy** (Used for input preprocessing (filter_cells, filter_genes, normalize_total, log1p, scale) and for downstream visualization of explainability results)

## Examples

```
from scsims import SIMS; sims = SIMS(weights_path='model.ckpt'); explainability_matrix, _labels = sims.explain('pvalb_unlabeled.h5ad')
```

## Evaluation signals

- Top-ranked genes reach mean explain values near ~0.2 across the 300 runs, while median genes sit near ~1e-6, demonstrating concentrated selection
- The top-ranked gene set recovers the reported marker set for the target class (e.g., Satb1, Pvalb, Cck, Gad1 for Pvalb+ INs)
- Dispersion of explain values for the top 50 genes is low, indicating consistent selection across independent explain runs
- Explainability matrix returned by sims.explain has one entry per gene in the trained model's feature space and pairs with the label output

## Limitations

- SIMS is a supervised model that relies on pre-existing annotations; errors from technical factors (sequencing technology, incorrect normalization) can propagate into differential-expression-driven cluster names and thus into the explain results
- Feature masks are only interpretable within the trained label space; combining datasets from different labs is manually intensive due to a lack of naming conventions in neuroscience
- SIMS has only been validated for label transfer within the same species; trans-species explainability or prediction remains untested

## Evidence

- [readme] Finally, to look at the explainability of the model, we similarly run: [evidence span withheld — anchor status: external_doc]
- [readme] explainability_matrix, _labels = sims.explain('my/new/unlabeled.h5ad'): [evidence span withheld — anchor status: external_doc]
