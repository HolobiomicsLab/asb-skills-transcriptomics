---
name: tabular-cell-type-classifier-training
description: Use when training or benchmarking a cross-tissue cell type annotation
  model on normalized tabular scRNA-seq data, or testing whether data augmentation
  improves holdout-donor generalization.
when_to_use_negative:
- Input data is not normalized tabular cell-by-gene data (e.g. raw count matrices,
  image or embedding-based representations) — the TabNet pipeline expects 19331-gene
  size-factor-normalized, log1p-transformed inputs
- Cell types or donors fail the corpus filters (e.g. fewer than 5000 cells per type,
  fewer than 30 donors per type), so generalization to new donors cannot be reliably
  quantified
- The goal is annotating modalities other than transcriptomics — scTab currently uses
  only transcriptomic input
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scTab
  role: Cell type prediction model for tabular scRNA-seq data, trained with the donor-difference
    augmentation scheme
  repo: https://github.com/theislab/scTab
- name: TabNet
  role: Tabular deep-learning architecture used as the scTab classifier backbone (TabnetClassifier
    with cross-entropy loss)
- name: NVIDIA-Merlin/dataloader
  role: GPU-optimized dataloader that reads the on-disk parquet store for large-than-memory
    training
  repo: https://github.com/NVIDIA-Merlin/dataloader
- name: cellxgene-census
  role: Standardised Python API to query and subset the CELLxGENE corpus for training
    data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:507c06ce6deca4295b8dc1b52e9f452cff4b57ca72b208bfe1ae6fa5f438e282
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NVIDIA-Merlin/dataloader
  - TabNet
  - cellxgene-census
  - scTab
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# tabular-cell-type-classifier-training

## Summary

Train and evaluate the scTab TabNet-based cell type classifier on tabular scRNA-seq data from the CELLxGENE census, including donor-difference data augmentation to improve cross-donor generalization.

## When to use

Use when training or benchmarking a cross-tissue cell type annotation model on normalized tabular scRNA-seq data, or testing whether data augmentation improves holdout-donor generalization.

## When NOT to use

- Input data is not normalized tabular cell-by-gene data (e.g. raw count matrices, image or embedding-based representations) — the TabNet pipeline expects 19331-gene size-factor-normalized, log1p-transformed inputs
- Cell types or donors fail the corpus filters (e.g. fewer than 5000 cells per type, fewer than 30 donors per type), so generalization to new donors cannot be reliably quantified
- The goal is annotating modalities other than transcriptomics — scTab currently uses only transcriptomic input

## Inputs

- Donor-subsampled train/val/test store in Merlin on-disk format (e.g. merlin_cxg_2023_05_15_sf-log1p), CxG census version 2023-05-15
- Gene list of 19331 protein-coding genes (notebooks/store_creation/features.parquet)
- Cell Ontology child-node lookup matrix for subtype-aware scoring
- Donor-difference augmentation vectors (notebooks/data_augmentation)

## Outputs

- Trained TabNet scTab model checkpoints with and without augmentation, one per random seed
- Per-seed and aggregated macro F1-score and test loss (e.g. augmentation_ablation_metrics.csv)

## How to apply

Load the donor-subsampled train/val/test store built from the CELLxGENE census (version 2023-05-15), with donor-based 70/15/15 splits so each donor falls entirely in one split to prevent donor-level leakage. Inputs are 19331 protein-coding gene expression values, size-factor normalized to 10000 counts and log1p-transformed, served via the Nvidia Merlin dataloader. Train the TabNet-based scTab classifier (TabnetClassifier, cross-entropy loss) as the no-augmentation baseline over multiple random seeds, then retrain the identical architecture with donor-difference augmentation vectors applied during training using the same seeds. Evaluate on the unseen-donor holdout test set, scoring a prediction as correct when it matches the annotator label or one of its subtypes via the Cell Ontology child-node lookup matrix; compute macro F1-score and test loss and run paired significance tests. Reproduction is confirmed when loss drops from 0.797±0.05 to 0.659±0.04 (p=0.0039) and macro F1 rises from 0.7755±0.0020 to 0.7841±0.0030 (p=0.0016).

## Related tools

- **scTab** (Cell type prediction model for tabular scRNA-seq data, trained with the donor-difference augmentation scheme) — https://github.com/theislab/scTab
- **TabNet** (Tabular deep-learning architecture used as the scTab classifier backbone (TabnetClassifier with cross-entropy loss))
- **NVIDIA-Merlin/dataloader** (GPU-optimized dataloader that reads the on-disk parquet store for large-than-memory training) — https://github.com/NVIDIA-Merlin/dataloader
- **cellxgene-census** (Standardised Python API to query and subset the CELLxGENE corpus for training data)

## Examples

```
pip install -e . && python notebooks/training/train_tabnet.py --data merlin_cxg_2023_05_15_sf-log1p --augmentation donor-difference --seed 0
```

## Evaluation signals

- Holdout test loss decreases from 0.797±0.05 (baseline) to 0.659±0.04 (augmented), paired p=0.0039
- Macro F1-score increases from 0.7755±0.0020 to 0.7841±0.0030, paired p=0.0016
- Donor-based split check: every donor appears in exactly one of train/val/test (no donor-level leakage)
- Prediction counted correct only if it matches the annotator label or a subtype via the Cell Ontology child-node lookup matrix
- Metrics file (augmentation_ablation_metrics.csv) contains per-seed and aggregated values for both conditions over identical seeds

## Limitations

- Cell Ontology incompleteness: not all cell types, especially rare ones, match an ontology term, and ontology relationships are still under active discussion, affecting classification metrics
- Plain macro F1 may not reflect complexity of datasets imbalanced across cell types, tissues, and sequencing protocols
- Current benchmark excludes uncommon assay technologies and finer cell subtypes due to data availability; pre-trained models still contain some coarsely annotated cell type labels

## Evidence

- [abstract] data augmentation improves generalization: "we show that the proposed data augmentation schema improves model generalization"
- [methods] size factor normalization to 10000 counts: "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [methods] donor-based split requirement: "2. `02_create_train_val_test_split.ipynb`: Split data into train, val and test sets"
- [methods] donor filter for generalization quantification: "5. Each cell type has to be observed in at least `30` donors to reliably quantify whether the classifier can"
- [intro] TabNet architecture selection: [evidence span withheld — anchor status: not_found]
- [readme] training data download for Merlin dataloader: "Training data (compatible with Merlin Dataloader infrastructure): https://pklab.med.harvard.edu/felix/data/merlin_cxg_2023_05_15_sf-log1p.tar.gz (164GB)"
