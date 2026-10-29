---
name: cell-type-classification-evaluation
description: Use when a labeled single-cell RNA-seq AnnData (cell x gene, 'class_label'
  column) needs automated cell-type label transfer with measurable accuracy/F1, including
  imbalanced, many-class (e.g., 42-type mouse brain) or small-training-set settings.
when_to_use_negative:
- 'Trans-species label transfer: the authors limited work to samples within the same
  species because neuronal subtypes diverge significantly between species'
- 'Completely unlabeled data with no expert annotations: SIMS is supervised and relies
  on pre-existing normalization, clustering, differential expression, and expert annotation'
- Input matrices in gene x cell orientation, or raw unnormalized counts not log1p-transformed
  as required by the pipeline
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Low-code label-transfer pipeline being trained and evaluated; wraps the TabNet
    classifier with dataloaders, checkpointing, weights-based loss weighting, and
    predict/explain methods
  repo: github.com/jlehrer1/sims
- name: TabNet
  role: Underlying deep-learning architecture (n_a=64, n_d=64) optimized for tabular
    data, able to handle 45,000+ gene features
  repo: github.com/jlehrer1/sims
- name: PyTorch Lightning
  role: 'Trainer orchestration: EarlyStopping on val_loss (patience=50), LearningRateMonitor
    per epoch, max_epochs=100'
- name: Scanpy
  role: Preprocessing of .h5ad inputs (filter_cells min_genes=100, filter_genes min_cells=3,
    normalize_total, log1p, scale) before model ingestion
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:83dc2727464885191af4eb06587266bcd9813a1968a3cc87879c5f43865d09a9
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cell-type-classification-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train and quantitatively evaluate the SIMS (TabNet-based) single-cell RNA-seq label-transfer classifier on annotated .h5ad datasets, reporting held-out accuracy and Macro F1 under 80/20 splits, trans-sample splits, and training-set ablations (e.g., 7% of cells / 3,285 cells).

## When to use

Use when a labeled single-cell RNA-seq AnnData (cell x gene, 'class_label' column) needs automated cell-type label transfer with measurable accuracy/F1, including imbalanced, many-class (e.g., 42-type mouse brain) or small-training-set settings.

## When NOT to use

- Trans-species label transfer: the authors limited work to samples within the same species because neuronal subtypes diverge significantly between species
- Completely unlabeled data with no expert annotations: SIMS is supervised and relies on pre-existing normalization, clustering, differential expression, and expert annotation
- Input matrices in gene x cell orientation, or raw unnormalized counts not log1p-transformed as required by the pipeline

## Inputs

- Annotated AnnData (.h5ad) expression matrix in cell x gene orientation with a 'class_label' observation column
- Preprocessing applied: sc.pp.filter_cells(min_genes=100), sc.pp.filter_genes(min_cells=3), sc.pp.normalize_total, sc.pp.log1p, sc.pp.scale
- Train-test split specification (80/20, per-donor trans-sample, or a cell-count subsample such as 3,285 cells)

## Outputs

- Held-out label-transfer accuracy on the test split (e.g., ~97.6% mouse, ~98.0% human)
- Macro F1 score (e.g., ~0.983 mouse, ~0.974 human; ~0.90 in the 7% ablation)
- Per-run metric record: accuracy, Macro F1, and training subset size
- Trained SIMS/TabNet model checkpoint (.ckpt) reusable for predict() and explain()

## How to apply

Load the annotated expression matrix (cell x gene, log1p-normalized via normalize_total/log1p/scale after filtering min_genes=100 and min_cells=3) as AnnData, then initialize SIMS with labels_key='class_label'. Set up the TabNet model with n_a=64 and n_d=64 and pass weights=sims.weights so the loss is weighted inversely by label frequency to handle rare cell types and class imbalance. Configure the PyTorch Lightning trainer with EarlyStopping monitoring val_loss (patience=50), a LearningRateMonitor logging per epoch, and max_epochs=100, then call sims.train() on an 80/20 train-test split and compute accuracy and Macro F1 on the held-out 20%. For data-efficiency ablations, repeat training on a small subsample (e.g., 7% of cells, 3,285 cells) under identical settings; for cross-sample generalization, train on one donor and test on the other two (trans-sample combinations). Compare resulting metrics against paper-reported references: 97.6% accuracy / 0.983 Macro F1 (Allen mouse cortex/hippocampus, 42 cell types), >95% accuracy / 0.90 Macro F1 for the 7% ablation, and 98.0% accuracy / 0.974 Macro F1 (human adult cortex single-nuclei, 49,495 nuclei).

## Related tools

- **SIMS** (Low-code label-transfer pipeline being trained and evaluated; wraps the TabNet classifier with dataloaders, checkpointing, weights-based loss weighting, and predict/explain methods) — github.com/jlehrer1/sims
- **TabNet** (Underlying deep-learning architecture (n_a=64, n_d=64) optimized for tabular data, able to handle 45,000+ gene features) — github.com/jlehrer1/sims
- **PyTorch Lightning** (Trainer orchestration: EarlyStopping on val_loss (patience=50), LearningRateMonitor per epoch, max_epochs=100)
- **Scanpy** (Preprocessing of .h5ad inputs (filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale) before model ingestion)

## Examples

```
from scsims import SIMS; sims = SIMS(adata=adata, labels_key='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.setup_trainer(max_epochs=100); sims.train()
```

## Evaluation signals

- Allen mouse cortex/hippocampus (42 cell types, 80/20 split): test accuracy ~97.6% with Macro F1 ~0.983
- 7%-cell ablation (3,285 training cells): accuracy >95% and Macro F1 ~0.90 (Median F1 >0.95)
- Human adult cortex single-nuclei (80/20): accuracy ~98.0% with Macro F1 ~0.974
- Human trans-sample (train on one donor, test on other two): accuracies in the 93.1%-95.8% range (Figure 4, Tables S2-S8)
- Input schema check: matrix is cell x gene and log1p-normalized; predict() returns DataFrame columns pred_0/prob_0 (top_k=1)

## Limitations

- Only validated for label transfer within the same species; trans-species prediction untested because neuronal subtypes diverge significantly between species
- As a supervised model, SIMS inherits errors from pre-existing annotations; incorrect normalization or sequencing-technology effects can lead to misnamed clusters
- SIMS is not the fastest benchmarked method; distillation/pruning speedups deferred to future work; metadata (sequencing technology, cell line) not yet integrated; lack of naming conventions across labs makes combining training datasets manually intensive

## Evidence

- [methods] weighting loss by label frequency: "sims.setup_model(n_a=64, n_d=64, weights=sims.weights)  # weighting loss by label freq"
- [methods] early stopping and LR monitoring callbacks: "callbacks=[EarlyStopping(monitor="val_loss", patience=50,), LearningRateMonitor(logging_interval="epoch"),]"
- [readme] SIMS initialization on AnnData with class labels: "sims = SIMS(adata=adata, labels_key='class_label')
sims.train()"
- [readme] required preprocessing before model ingestion: "sc.pp.filter_cells(adata, min_genes=100)
sc.pp.filter_genes(adata, min_cells=3)"
- [abstract] data-efficiency claim supporting the ablation: "achieving high accuracy even with small training sets (<3,500 cells) and across different samples"
- [discussion] TabNet handles large feature spaces: "SIMS, being based on TabNet, and therefore optimized for tabular data,30 can work well with over 45,000 features (Figure 2)."
- [discussion] trans-species limitation: "we have limited our work to samples within the same species. This is because neuronal subtypes diverge significantly between species46"
