---
name: training-subset-scaling-curve-analysis
description: Use when you need to determine whether model performance scaling with
  training corpus size reflects added cells or added donor/batch diversity, by comparing
  subsampling regimes on a donor-stratified dataset.
when_to_use_negative:
- The dataset has no donor/batch identifier, so a donor-based split (the variable
  being tested) cannot be constructed
- The goal is overall benchmark comparison of annotation models rather than attribution
  of data-scaling effects — use a standard benchmarking protocol instead
- Only a single fixed training set is available and subsampling to 2.1M–15.2M cells
  is infeasible
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scTab
  role: Training pipeline and model family used to fit the TabNet classifier on each
    training subsample
  repo: https://github.com/theislab/scTab
- name: TabNet
  role: Tabular-data architecture of the classifier trained on each subsample (19,331
    input genes, cross-entropy loss, class probabilities output)
- name: NVIDIA-Merlin/dataloader
  role: GPU dataloader used to read the on-disk Merlin-format training store for each
    subsample
  repo: https://github.com/NVIDIA-Merlin/dataloader
- name: NumPy
  role: Used in the sf_normalize size-factor normalization (10,000 counts + log1p)
    underlying the training data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:d6e03bc64d7ddae7a81c94c04bf539fb4dbcc118853fbac1b51ece8953c09a8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NVIDIA-Merlin/dataloader
  - NumPy
  - TabNet
  - scTab
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# training-subset-scaling-curve-analysis

## Summary

Trains a classifier (scTab's TabNet) on nested training-set subsamples under two regimes — donor/batch-based and cell-based subsampling — and fits macro F1-score and cross-entropy loss against training size to disentangle whether data scaling is driven by cell count or by donor/batch diversity.

## When to use

Use when you need to determine whether model performance scaling with training corpus size reflects added cells or added donor/batch diversity, by comparing subsampling regimes on a donor-stratified dataset.

## When NOT to use

- The dataset has no donor/batch identifier, so a donor-based split (the variable being tested) cannot be constructed
- The goal is overall benchmark comparison of annotation models rather than attribution of data-scaling effects — use a standard benchmarking protocol instead
- Only a single fixed training set is available and subsampling to 2.1M–15.2M cells is infeasible

## Inputs

- On-disk Merlin-format data store (merlin_cxg_2023_05_15_sf-log1p, 164GB; 22,189,056 cells x 19,331 protein-coding genes, size-factor normalized to 10,000 counts and log1p-transformed)
- Donor-based train/val/test split (0.7/0.15/0.15) with fixed donor-held-out val and test sets
- Trained scTab checkpoints (scTab-checkpoints.tar.gz, 8.1GB) or capacity to retrain TabNet on each subsample

## Outputs

- Table of macro F1-score and cross-entropy loss versus training set size for donor-based and cell-based subsampling regimes
- Estimated scaling gap between the 2.1M-cell and 15.2M-cell donor-based regimes per organ system
- scTab-vs-linear reference model macro F1 difference at full training data

## How to apply

Load the on-disk Merlin-format store (22,189,056 cells x 19,331 protein-coding genes, each cell size-factor normalized to 10,000 counts and log1p-transformed) with the Nvidia Merlin dataloader. Keep donor-held-out val (0.15) and test (0.15) splits fixed; from the 0.7 train fraction create donor-based subsamples containing 15%, 30%, 50%, 70%, and 100% of training donors, plus matched cell-based subsamples for comparison. Train the TabNet classifier (19,331 input genes, cross-entropy loss, class-probability output) via the scTab training pipeline on each subsample, then evaluate each model on the unseen-donor test set computing macro F1 (per-cell-type F1 averaged over cell types) and cross-entropy loss. Tabulate both metrics versus training size for both regimes and compare against the linear reference model at the full 15.2M-cell training size. Correct application is judged by whether donor-based subsampling yields steep metric improvement while cell-based subsampling improves only weakly, and by the quantitative anchors below.

## Related tools

- **scTab** (Training pipeline and model family used to fit the TabNet classifier on each training subsample) — https://github.com/theislab/scTab
- **TabNet** (Tabular-data architecture of the classifier trained on each subsample (19,331 input genes, cross-entropy loss, class probabilities output))
- **NVIDIA-Merlin/dataloader** (GPU dataloader used to read the on-disk Merlin-format training store for each subsample) — https://github.com/NVIDIA-Merlin/dataloader
- **NumPy** (Used in the sf_normalize size-factor normalization (10,000 counts + log1p) underlying the training data)

## Examples

```
dataset = merlin.io.Dataset(MERLIN_STORE_PATH, engine="parquet"); loader = Loader(dataset, batch_size=65536)  # then train TabNet per donor-fraction subsample via scTab pipeline
```

## Evaluation signals

- At full training data, scTab outperforms the linear reference model by 0.0447 ± 0.0008 macro F1-score
- Donor-based scaling from 2.1M to 15.2M cells yields a minimum macro F1-score difference of 0.0454 ± 0.0083 (median 0.1219 ± 0.0212) across organ systems
- Macro F1 improves strongly with training size under donor-based subsampling but only weakly under cell-based subsampling
- Val and test splits remain fixed donor-held-out sets (0.15/0.15) across all subsample sizes, so all points are evaluated on the same unseen-donor test set
- Training sizes correspond to 15%, 30%, 50%, 70%, and 100% of training donors (train split = 0.7 of the full dataset)

## Limitations

- Plain macro F1-scores may not properly reflect the complexity of unbalanced datasets which are imbalanced across cell types, tissues, and sequencing protocols
- Cell Ontology incompleteness: not all cell types (especially rare ones) can be matched to an ontology term, which affects classification metrics
- Annotation granularity inconsistency across CELLxGENE datasets represents missing information that cannot be fixed by model architecture

## Evidence

- [results] Scaling driven by batch diversity: [evidence span withheld — anchor status: not_found]
- [methods] Data store dimensions: [evidence span withheld — anchor status: not_found]
- [methods] Normalization procedure: "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [readme] Training data download: "Training data (compatible with Merlin Dataloader infrastructure): https://pklab.med.harvard.edu/felix/data/merlin_cxg_2023_05_15_sf-log1p.tar.gz (164GB)"
- [abstract] Data scaling finding: "the performance of scTab scales both in terms of training dataset size and model size"
