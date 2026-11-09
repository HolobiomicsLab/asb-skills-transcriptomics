---
name: single-cell-rna-label-transfer
description: Use when you need to transfer cell-type annotations from a labeled single-cell
  RNA-seq reference (e.g., .h5ad AnnData with a 'class_label' column) to unlabeled
  query cells, even across datasets with different gene panels or cell lines.
when_to_use_negative:
- Query cells come from a different species than the training reference — the authors
  limited validation to samples within the same species because neuronal subtypes
  diverge significantly between species
- No pre-existing annotations are available for any reference dataset — SIMS is supervised
  and relies on pre-existing annotations (clustering, differential expression, expert
  annotation)
- The input is not a cell x gene expression matrix (e.g., gene x cell) — SIMS requires
  cells as rows since training samples are individual-cell transcriptomes
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Low-code pipeline providing the SIMS class, TabNet-backed training, predict(),
    explain(), and automated gene-intersection for cross-dataset label transfer
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying architecture optimized for large-scale tabular data; sparse feature
    masks enable per-class gene importance (explain values ~0.2 for top genes vs ~1e-6
    median)
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Training framework supplying EarlyStopping (val_loss, patience=50), LearningRateMonitor,
    checkpointing, and the pl.Trainer configured via sims.setup_trainer()
  repo: https://github.com/braingeneers/SIMS
- name: Scanpy
  role: Preprocessing of AnnData (filter_cells min_genes=100, filter_genes min_cells=3,
    normalize_total, log1p, scale) and downstream plotting of predictions/explainability
- name: Gruffi
  role: Optional removal of Gruffi-annotated stressed cells from organoid datasets
    before label transfer evaluation
  repo: https://github.com/jn-goe/gruffi
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_007
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:472ae485d54853a7738c21eca567e89c2b7bc3439d11608466aaa9a202a84544
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# single-cell-rna-label-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and apply SIMS, a TabNet-based deep-learning pipeline implemented in PyTorch Lightning, to transfer cell-type labels across single-cell RNA-seq datasets, including benchmarks against scANVI, Seurat, scNym, SingleR, and SciBet and cross-cell-line/organoid prediction.

## When to use

Use when you need to transfer cell-type annotations from a labeled single-cell RNA-seq reference (e.g., .h5ad AnnData with a 'class_label' column) to unlabeled query cells, even across datasets with different gene panels or cell lines.

## When NOT to use

- Query cells come from a different species than the training reference — the authors limited validation to samples within the same species because neuronal subtypes diverge significantly between species
- No pre-existing annotations are available for any reference dataset — SIMS is supervised and relies on pre-existing annotations (clustering, differential expression, expert annotation)
- The input is not a cell x gene expression matrix (e.g., gene x cell) — SIMS requires cells as rows since training samples are individual-cell transcriptomes

## Inputs

- AnnData (.h5ad) or CSV expression matrix in cell x gene orientation, log1p-normalized
- Label file or obs column named 'class_label' containing cell-type annotations for the training reference
- Unlabeled query AnnData/CSV for prediction (preprocessed identically to training data)
- Optional trained SIMS model checkpoint (.ckpt) via weights_path for inference only

## Outputs

- pandas DataFrame of predictions with columns pred_0..pred_k and prob_0..prob_k (softmax probabilities), or a single sims_label/sims_confidence pair with top_k=1
- Explainability matrix of per-gene importance values from sims.explain()
- Benchmark metrics (accuracy, Macro F1) per method-dataset pair under 5-fold cross-validation
- gene_intersection.csv listing genes shared between query panel and model training features

## How to apply

Load the expression matrix as cell-by-gene AnnData (h5ad or csv), lightly filter (min_genes=100, min_cells=3), then normalize_total, log1p, and scale before ingestion. Initialize SIMS with the data and labels_key='class_label', set up the TabNet model with n_a=64, n_d=64, and weights=sims.weights (inverse label-frequency loss weighting to learn rare cell types), and configure the trainer with EarlyStopping on 'val_loss' (patience=50), a LearningRateMonitor (logging_interval='epoch'), and max_epochs=100. Call sims.train(), then run sims.predict() on the preprocessed query data to get top-k predicted labels with softmax probabilities, or sims.explain() to obtain the sparse feature-selection (explainability) matrix. For query datasets with a different gene panel, intersect the query's gene names with the model's training features and reorder the query matrix to the model's feature ordering before prediction. Apply the identical preprocessing to query data as used during training; validate by checking accuracy and Macro F1 on held-out folds against known-good ranges (e.g., ~97.6% accuracy / 0.983 Macro F1 on the Allen mouse cortex/hippocampus 80/20 split).

## Related tools

- **SIMS** (Low-code pipeline providing the SIMS class, TabNet-backed training, predict(), explain(), and automated gene-intersection for cross-dataset label transfer) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying architecture optimized for large-scale tabular data; sparse feature masks enable per-class gene importance (explain values ~0.2 for top genes vs ~1e-6 median)) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Training framework supplying EarlyStopping (val_loss, patience=50), LearningRateMonitor, checkpointing, and the pl.Trainer configured via sims.setup_trainer()) — https://github.com/braingeneers/SIMS
- **Scanpy** (Preprocessing of AnnData (filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale) and downstream plotting of predictions/explainability)
- **Gruffi** (Optional removal of Gruffi-annotated stressed cells from organoid datasets before label transfer evaluation) — https://github.com/jn-goe/gruffi

## Examples

```
from scsims import SIMS; sims = SIMS(adata=adata, labels_key='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.setup_trainer(max_epochs=100); sims.train()
```

## Evaluation signals

- On the Allen mouse cortex/hippocampus dataset (42 cell types, 80/20 split), test accuracy should reach ~97.6% with Macro F1 ~0.983
- 7% training subsample (3,285 cells) should still exceed 95% accuracy with Macro F1 ~0.90
- Predictions DataFrame must contain pred_0/prob_0 (and higher k) columns for every input cell; no feature-mismatch errors after gene intersection
- 5-fold cross-validation across PBMC68K, Tucker heart, Stewart kidney, Krasnow lung, and Han landscape should show SIMS accuracy on par with or exceeding scANVI, Seurat, scNym, SingleR, and SciBet
- Cross-cell-line organoid transfer (11A → PGP1/GM8330) should yield ~71.3% / ~67.4% accuracy (Macro F1 ~0.564 / ~0.570)

## Limitations

- Validated only for label transfer within the same species; trans-species prediction remains untested
- As a supervised model, SIMS depends on pre-existing annotations; technical factors such as sequencing technology or incorrect normalization can propagate misnamed clusters
- Lack of naming conventions in neuroscience makes combining datasets from different labs for training manually intensive; ontology-based harmonization is only proposed as future work
- SIMS is not the fastest of the benchmarked methods; speed optimizations (distillation, pruning) are deferred to future work

## Evidence

- [readme] README prediction output: [evidence span withheld — anchor status: external_doc]
- [results] Benchmark finding: [evidence span withheld — anchor status: not_found]
