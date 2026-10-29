---
name: class-imbalance-loss-weighting
description: Use when training a SIMS classifier on annotated scRNA-seq data (h5ad
  AnnData with a 'class_label' column) whose cell-type distribution is imbalanced
  and rare populations must still be learned.
when_to_use_negative:
- The dataset is perfectly balanced across cell types and loss weighting would add
  unnecessary variance to gradient updates
- You are running SIMS's unsupervised pretraining stage, which never reads cell-type
  labels and therefore cannot apply class-frequency weights
- Labels come from a different species or lack a shared naming convention with the
  training set, so weighted loss cannot compensate for ontological mismatch
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Provides sims.weights (inverse label-frequency factors) consumed by setup_model
    to weight the loss; scsims Python package
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying model architecture whose training loss is weighted by class frequency
    (n_a=64, n_d=64)
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Training framework in which the weighted loss is optimized with EarlyStopping
    on val_loss (patience=50) and up to 100 epochs
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_004
  - task_005
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:8d06b138ae6fef31d533d5b7972012fb53526fa7e6bd0b6643257ffcbc1551db
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
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# class-imbalance-loss-weighting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Weight the SIMS/TabNet training loss inversely proportional to cell-type label frequency so rare cell types are not outvoted by abundant ones during supervised label transfer on imbalanced single-cell RNA-seq datasets.

## When to use

Use when training a SIMS classifier on annotated scRNA-seq data (h5ad AnnData with a 'class_label' column) whose cell-type distribution is imbalanced and rare populations must still be learned.

## When NOT to use

- The dataset is perfectly balanced across cell types and loss weighting would add unnecessary variance to gradient updates
- You are running SIMS's unsupervised pretraining stage, which never reads cell-type labels and therefore cannot apply class-frequency weights
- Labels come from a different species or lack a shared naming convention with the training set, so weighted loss cannot compensate for ontological mismatch

## Inputs

- AnnData (.h5ad) cell-by-gene log1p-normalized expression matrix with a class_label column of cell-type annotations
- Per-class label frequency counts exposed as sims.weights by the SIMS package

## Outputs

- Trained SIMS/TabNet classifier checkpoint whose loss was frequency-weighted
- Held-out accuracy and Macro F1 metrics reflecting improved minority-class performance

## How to apply

After instantiating SIMS on an AnnData object with labels_key='class_label', call sims.setup_model(n_a=64, n_d=64, weights=sims.weights); the weights attribute holds per-class inverse-frequency factors that scale the loss contribution of each training cell. This is the article's recommended configuration and is applied identically across all its experiments (5-dataset benchmark, Allen mouse cortex/hippocampus 42-type model, human cortex nuclei, developing cortex, and organoid transfer). Pair the weighting with class-frequency-sensitive metrics such as Macro F1, since weighted training aims to lift performance on minority classes that plain accuracy can mask. Keep the weighting in place even when subsampling (e.g., the 3,285-cell ablation) because small per-class sample counts amplify imbalance.

## Related tools

- **SIMS** (Provides sims.weights (inverse label-frequency factors) consumed by setup_model to weight the loss; scsims Python package) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying model architecture whose training loss is weighted by class frequency (n_a=64, n_d=64)) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Training framework in which the weighted loss is optimized with EarlyStopping on val_loss (patience=50) and up to 100 epochs)

## Examples

```
sims = SIMS(data=adata, class_label='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.setup_trainer(max_epochs=100); sims.train()
```

## Evaluation signals

- Training run uses setup_model(..., weights=sims.weights) rather than default unweighted loss
- Macro F1 improves or stays close to accuracy on imbalanced splits (e.g., 0.983 Macro F1 at 97.6% accuracy on Allen mouse data), indicating rare classes are not ignored
- Rare cell types receive non-trivial predictions in held-out folds instead of being collapsed into majority labels
- The weighting remains effective in low-data regimes: 7% training ablation (3,285 cells) still exceeds 95% accuracy and ~0.90 Macro F1

## Limitations

- Weighting compensates for label imbalance but cannot fix missing or inconsistent cell-type naming across labs, which the authors note makes combining datasets manually intensive
- Supervised weighting depends on pre-existing annotations; errors introduced upstream by normalization or differential expression propagate into the weighted loss
- Class-frequency weighting is described as 'recommended' rather than mandatory; the article does not report an ablation isolating its individual contribution

## Evidence

- [methods] Set up model with architecture parameters and class-frequency weighting: "sims.setup_model(n_a=64, n_d=64, weights=sims.weights)  # weighting loss by label freq"
- [readme] Weighting loss inversely by label frequency helps learn rare cell types: [evidence span withheld — anchor status: external_doc]
- [intro] Accuracy of automated classifiers degrades with many cell types and few samples per label: "the accuracy of these automated classifiers often degrades as the number of cell types increase, and the number of samples per label becomes small."
