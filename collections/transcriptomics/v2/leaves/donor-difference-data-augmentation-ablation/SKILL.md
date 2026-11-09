---
name: donor-difference-data-augmentation-ablation
description: Use when evaluating whether data augmentation improves generalization
  of a tabular single-cell cell type classifier to unseen donors, or when reproducing
  the scTab augmentation ablation (test loss 0.797→0.659, macro F1 0.7755→0.7841).
when_to_use_negative:
- The research goal is scaling behavior with training set or model size, not the effect
  of augmentation — use the scaling-law evaluation instead
- The data split is not donor-based (e.g., random cell-level splits), since donor-level
  leakage would invalidate the generalization comparison
- Input data are not sf-log1p normalized tabular scRNA-seq in the 19331 protein-coding
  gene space expected by the scTab/TabNet pipeline
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scTab
  role: Cell type prediction model whose augmented and non-augmented training runs
    constitute the ablation
  repo: https://github.com/theislab/scTab
- name: TabNet
  role: Tabular deep-learning architecture used for the TabnetClassifier trained in
    both ablation conditions
- name: NVIDIA-Merlin/dataloader
  role: Reads the on-disk Cell-by-Gene store into GPU memory for training the classifier
    in both conditions
  repo: https://github.com/NVIDIA-Merlin/dataloader
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:b8d24d2541ee9fe873e1f74d69d64124183cffae281cd2abe2fdb66df5a7a439
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NVIDIA-Merlin/dataloader
  - TabNet
  - scTab
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# donor-difference-data-augmentation-ablation

## Summary

Reproduce the scTab donor-difference data augmentation ablation: train the TabNet-based cell type classifier with and without augmentation vectors over multiple seeds and compare holdout test loss and macro F1 on unseen donors. This quantifies whether augmentation improves cross-donor generalization of the cross-tissue classifier.

## When to use

Use when evaluating whether data augmentation improves generalization of a tabular single-cell cell type classifier to unseen donors, or when reproducing the scTab augmentation ablation (test loss 0.797→0.659, macro F1 0.7755→0.7841).

## When NOT to use

- The research goal is scaling behavior with training set or model size, not the effect of augmentation — use the scaling-law evaluation instead
- The data split is not donor-based (e.g., random cell-level splits), since donor-level leakage would invalidate the generalization comparison
- Input data are not sf-log1p normalized tabular scRNA-seq in the 19331 protein-coding gene space expected by the scTab/TabNet pipeline

## Inputs

- Donor-subsampled train/val/test store in Merlin Dataloader on-disk format (merlin_cxg_2023_05_15_sf-log1p, 164GB; minimal 0.5GB subset available)
- Donor-difference augmentation vectors (notebooks/data_augmentation)
- Cell Ontology child-node lookup matrix for subtype-aware scoring

## Outputs

- Per-seed and aggregated test loss and macro F1-score for augmented vs. non-augmented conditions
- augmentation_ablation_metrics.csv
- Paired significance test results (p-values) comparing augmented vs. non-augmented runs

## How to apply

Load the donor-subsampled train/val/test store built from the CELLxGENE census (version 2023-05-15, sf-log1p normalized, 19331 protein-coding genes) with a donor-based 70/15/15 split so each donor falls entirely in one split and no donor-level leakage occurs. Train the TabNet-based scTab classifier (TabnetClassifier, cross-entropy loss) as the no-augmentation baseline over multiple random seeds, then retrain the identical architecture with donor-difference augmentation vectors applied during training using the same seeds. Evaluate both model sets on the unseen-donor holdout test set, scoring a prediction as correct when it matches the annotator label or a Cell Ontology subtype via the child-node lookup matrix. Compute macro F1 and test loss per seed, run paired significance tests (expected p=0.0039 for loss, p=0.0016 for macro F1), and write per-seed and aggregated metrics to augmentation_ablation_metrics.csv.

## Related tools

- **scTab** (Cell type prediction model whose augmented and non-augmented training runs constitute the ablation) — https://github.com/theislab/scTab
- **TabNet** (Tabular deep-learning architecture used for the TabnetClassifier trained in both ablation conditions)
- **NVIDIA-Merlin/dataloader** (Reads the on-disk Cell-by-Gene store into GPU memory for training the classifier in both conditions) — https://github.com/NVIDIA-Merlin/dataloader

## Examples

```
from cellnet import TabnetClassifier; model = TabnetClassifier(); model.fit(train_loader, augmentation_vectors=aug_vectors)
```

## Evaluation signals

- Holdout test loss decreases from 0.797 ± 0.05 (baseline) to 0.659 ± 0.04 (augmented) with paired p=0.0039
- Macro F1-score increases from 0.7755 ± 0.0020 to 0.7841 ± 0.0030 with paired p=0.0016
- Each donor appears in exactly one of the 70/15/15 train/val/test splits (no donor-level leakage)
- Same random seeds used for paired augmented vs. non-augmented runs; per-seed metrics present in augmentation_ablation_metrics.csv
- Predictions scored correct if matching annotator label or a Cell Ontology subtype via the child-node lookup matrix

## Limitations

- Macro F1-scores may not reflect dataset complexity across imbalanced cell types, tissues, and sequencing protocols
- Cell Ontology incompleteness: not all cell types (especially rare ones) match an ontology term, affecting subtype-aware scoring
- The benchmark excludes uncommon assay technologies and finer cell subtypes due to data availability

## Evidence

- [abstract] augmentation improves generalization: "we show that the proposed data augmentation schema improves model generalization"
- [methods] donor split rationale: "Each cell type has to be observed in at least `30` donors to reliably quantify whether the classifier can generalize to new donors."
- [methods] preprocessing normalization: "1. Normalize each cell to have `10000` counts (size factor normalization)"
- [readme] augmentation evaluation notebooks: "``data_augmentation``: Notebooks related to data augmentation &rarr; calculation of augmentation vectors + evaluation"
