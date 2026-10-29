---
name: paired-significance-testing-of-model-metrics
description: Use when comparing two model-training conditions (e.g. augmentation on/off)
  evaluated on the same holdout test set across matched random seeds, and you must
  report statistically significant differences in loss or macro F1.
when_to_use_negative:
- Metrics are not paired — models were evaluated on different test sets or trained
  with unmatched seeds, so pairing assumptions are violated
- Comparing more than two conditions simultaneously; the article's ablation design
  is a two-condition augmented vs. non-augmented comparison
- Evaluating a single training run without per-seed replication, which provides no
  variance estimate for paired testing
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scTab
  role: Cell type classifier trained under both conditions (with and without donor-difference
    augmentation) to generate the paired metric sets being tested
  repo: https://github.com/theislab/scTab
- name: TabNet
  role: Tabular architecture underlying the scTab classifier (TabnetClassifier, cross-entropy
    loss) trained across multiple random seeds
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:dad90eaf1942e5f2bd01ea34cd8e6c637c062f6a111e2073e5412fedd4a93341
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - TabNet
  - scTab
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# paired-significance-testing-of-model-metrics

## Summary

Runs paired statistical tests comparing per-seed metrics (test loss and macro F1-score) from scTab models trained with and without donor-difference augmentation, to establish that augmentation significantly improves holdout generalization (loss 0.797±0.05 → 0.659±0.04, p=0.0039; macro F1 0.7755±0.0020 → 0.7841±0.0030, p=0.0016).

## When to use

Use when comparing two model-training conditions (e.g. augmentation on/off) evaluated on the same holdout test set across matched random seeds, and you must report statistically significant differences in loss or macro F1.

## When NOT to use

- Metrics are not paired — models were evaluated on different test sets or trained with unmatched seeds, so pairing assumptions are violated
- Comparing more than two conditions simultaneously; the article's ablation design is a two-condition augmented vs. non-augmented comparison
- Evaluating a single training run without per-seed replication, which provides no variance estimate for paired testing

## Inputs

- Per-seed test loss values for baseline (no-augmentation) and augmented scTab runs on the same holdout test set
- Per-seed macro F1-score values for both conditions
- Cell Ontology child node lookup matrix for subtype-aware correctness scoring
- Donor-based 70/15/15 train/val/test split store (19331 protein-coding genes, 10,000-count normalization, log1p-transformed)

## Outputs

- augmentation_ablation_metrics.csv with per-seed and aggregated metrics for both conditions
- Paired significance test results (p-values for test loss and macro F1 differences)

## How to apply

Train both model conditions (baseline TabNet-based scTab classifier and the identical architecture with donor-difference augmentation) over multiple random seeds, evaluating each on the same donor-disjoint holdout test set so that per-seed metrics are paired. Score predictions as correct when they match the annotator label or one of its Cell Ontology child-node subtypes using the child node lookup matrix. Compute test loss (cross-entropy) and macro F1-score (per-cell-type F1 averaged) for each seed of each condition. Run paired significance tests on the matched per-seed metric pairs to compare augmented vs. non-augmented runs. Confirm significance against the reported expectations: test loss p=0.0039 and macro F1 p=0.0016. Write per-seed and aggregated metrics for both conditions to augmentation_ablation_metrics.csv.

## Related tools

- **scTab** (Cell type classifier trained under both conditions (with and without donor-difference augmentation) to generate the paired metric sets being tested) — https://github.com/theislab/scTab
- **TabNet** (Tabular architecture underlying the scTab classifier (TabnetClassifier, cross-entropy loss) trained across multiple random seeds)

## Examples

```
import pandas as pd; from scipy.stats import wilcoxon; m = pd.read_csv('augmentation_ablation_metrics.csv'); stat, p_loss = wilcoxon(m['baseline_test_loss'], m['augmented_test_loss']); stat, p_f1 = wilcoxon(m['baseline_f1'], m['augmented_f1'])
```

## Evaluation signals

- Test loss decreases significantly from 0.797 ± 0.05 (no augmentation) to 0.659 ± 0.04 (augmentation) with p=0.0039
- Macro F1-score increases significantly from 0.7755 ± 0.0020 to 0.7841 ± 0.0030 with p=0.0016
- Per-seed metric pairs are matched: same seeds and same holdout test set used for both conditions
- augmentation_ablation_metrics.csv contains per-seed and aggregated metrics for both augmented and non-augmented conditions
- Correctness scoring uses the Cell Ontology child-node lookup so subtype predictions count as correct

## Limitations

- Plain macro F1-scores may not reflect the complexity of datasets imbalanced across cell types, tissues, and sequencing protocols; more detailed evaluation metrics may be needed
- Cell Ontology relationships are still actively discussed, which can affect classification metrics and thus the significance conclusions
- Correctness depends on cell types being mappable to ontology terms; rare cell types may not match an ontology term

## Evidence

- [results] loss decrease: [evidence span withheld — anchor status: not_found]
- [results] f1 increase: [evidence span withheld — anchor status: not_found]
- [methods] split rationale: [evidence span withheld — anchor status: not_found]
- [methods] subtype scoring: "Create child node lookup matrix to find subtypes based on cell type ontology"
- [readme] augmentation findings: [evidence span withheld — anchor status: not_found]
