---
name: tabnet-feature-mask-interpretability
description: 'Use when a trained SIMS/TabNet single-cell classifier must be interpreted:
  you need the sparse gene-selection masks for a specific label and evidence that
  the selected gene set is stable across independent explain runs.'
when_to_use_negative:
- 'Trans-species label transfer: SIMS masks have only been validated within the same
  species since neuronal subtypes diverge significantly between species'
- You only need cell-type predictions, not interpretability — use sims.predict instead
  of running repeated explain procedures
- The input is unlabeled and no trained supervised model/annotations exist — SIMS
  relies on pre-existing expert annotations which can carry errors from normalization
  or differential expression
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Pipeline providing the explain() procedure that surfaces TabNet sparse feature-selection
    masks for a target cell class
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying architecture whose attention-based sparse feature masks are the
    interpretability signal being sampled and aggregated
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Trainer framework used to train the SIMS model with EarlyStopping (val_loss,
    patience=50) and LearningRateMonitor callbacks before explain runs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:d2b4ce4d70dcaa6de30174595f464f86f158a8af812e5aad134213101bde66d6
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
  - TabNet
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tabnet-feature-mask-interpretability

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify stability of SIMS/TabNet sparse feature-selection masks by repeatedly running the explain procedure for a target cell class (e.g., Pvalb+ interneurons) and aggregating per-gene explain values, revealing which genes the classifier consistently relies on.

## When to use

Use when a trained SIMS/TabNet single-cell classifier must be interpreted: you need the sparse gene-selection masks for a specific label and evidence that the selected gene set is stable across independent explain runs.

## When NOT to use

- Trans-species label transfer: SIMS masks have only been validated within the same species since neuronal subtypes diverge significantly between species
- You only need cell-type predictions, not interpretability — use sims.predict instead of running repeated explain procedures
- The input is unlabeled and no trained supervised model/annotations exist — SIMS relies on pre-existing expert annotations which can carry errors from normalization or differential expression

## Inputs

- Trained SIMS/TabNet model checkpoint (.ckpt)
- AnnData (.h5ad) cell-by-gene expression matrix, log1p-normalized (normalize_total, log1p, scale), with a class_label column (e.g., Allen mouse cortex/hippocampus atlas)
- Target cell-class label for which masks are sampled (e.g., Pvalb+ interneurons)

## Outputs

- Per-gene mean explain-value table aggregated across runs (e.g., 300 runs per class)
- Dispersion index for top-ranked genes (e.g., top 50 genes, ~0.2 mean explain vs ~1e-6 median)
- Ranked selected-gene set for the target class, comparable to published marker lists (Figure 3E-F)

## How to apply

Train a SIMS model on a log1p-normalized cell-by-gene AnnData matrix (labels_key='class_label'), configuring TabNet with n_a=64 and n_d=64 and weighting the loss inversely by label frequency via sims.weights, then train for up to 100 epochs with EarlyStopping on val_loss (patience=50) and LearningRateMonitor per epoch. After training, run the TabNet explain procedure (sims.explain) many times (e.g., 300 runs) for the target class to sample the sparse feature-selection masks. Aggregate per-gene explain values across runs and compute dispersion (e.g., for the top 50 genes); in the SIMS Allen mouse cortex/hippocampus experiment, top Pvalb+ genes (Satb1, Pvalb, Cck, Gad1) reached mean explain values around 0.2 versus roughly 10^-6 for the median gene. Finally, compare the recovered top-ranked gene set against reported markers (Kcnip, Igf1, Rpp25, Dlx1, Dlx5, Gad1, Cck, Satb1, Pvalb, Lypd6, Dlx6os-1, Bmp3) as in Figure 3E-F to validate biological consistency.

## Related tools

- **SIMS** (Pipeline providing the explain() procedure that surfaces TabNet sparse feature-selection masks for a target cell class) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying architecture whose attention-based sparse feature masks are the interpretability signal being sampled and aggregated) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Trainer framework used to train the SIMS model with EarlyStopping (val_loss, patience=50) and LearningRateMonitor callbacks before explain runs)

## Examples

```
from scsims import SIMS; sims = SIMS(weights_path='model.ckpt'); explainability_matrix, _labels = sims.explain(adata)
```

## Evaluation signals

- Top-ranked genes reach mean explain values near ~0.2 across runs, versus ~1e-6 for the median gene (strong concentration of mask weight)
- The top-ranked gene set across independent runs is consistent (low dispersion for top 50 genes), reproducing the 300-run Allen mouse Pvalb+ experiment
- Recovered gene set recovers reported Pvalb+ markers: Kcnip, Igf1, Rpp25, Dlx1, Dlx5, Gad1, Cck, Satb1, Pvalb, Lypd6, Dlx6os-1, Bmp3 (Figure 3E-F)
- Explainability matrix has one entry per gene in the input matrix and is returned together with the corresponding labels from sims.explain

## Limitations

- Interpretability depends on supervised annotations: incorrect normalization or differential expression can misname clusters and propagate errors into the selected genes
- Consistency was demonstrated within a single species; cross-species mask interpretation is untested because neuronal subtypes diverge between species
- SIMS is not the fastest benchmarked method, and speed optimizations (distillation, pruning) are deferred to future work; repeat explain runs (300 per class) add compute cost

## Evidence

- [readme] Explain API returning masks and labels: [evidence span withheld — anchor status: external_doc]
- [readme] Data preprocessing expectation for model ingestion: [evidence span withheld — anchor status: external_doc]
