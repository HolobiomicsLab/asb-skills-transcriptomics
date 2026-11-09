---
name: cross-entropy-loss-evaluation
description: Use when training or evaluating cross-tissue cell type classifiers (scTab/TabNet)
  on the CELLxGENE Merlin store and you need the cross-entropy loss and macro F1-score
  on unseen-donor test data, e.g. to compare training-dataset-size scaling regimes.
when_to_use_negative:
- Evaluating linear reference baselines for comparisons where the expected scTab-over-linear
  gap (0.0447 ± 0.0008 macro F1 at full data) is the validation target — the gap definition
  requires the nonlinear scTab model specifically
- Datasets whose cell types fail the corpus filters (e.g. fewer than 5000 cells per
  cell type, fewer than 30 donors per cell type, or labels not subtypes of 'native
  cell'), where the observed loss may be confounded by coarse or missing Cell Ontology
  annotations
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scTab
  role: Training pipeline and model checkpoints used to produce class-probability
    outputs for cross-entropy loss and macro F1 evaluation
  repo: github.com__theislab__scTab
- name: TabNet
  role: Tabular classifier architecture (19,331 input genes, cross-entropy loss, class
    probabilities output) whose loss curves are evaluated
- name: NumPy
  role: Implements size-factor normalization (sf_normalize) to 10,000 counts and log1p
    transform that precede loss evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:8ec9c63bd3f448e6893950f369b82e52d49a50c3b5f58edf8461ea571b122367
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NumPy
  - TabNet
  - scTab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-entropy-loss-evaluation

## Summary

Quantify cell-type classification quality of scTab/TabNet models by cross-entropy loss (paired with macro F1-score) on donor-held-out test splits, enabling comparison of scaling regimes and model checkpoints in the scTab CELLxGENE corpus.

## When to use

Use when training or evaluating cross-tissue cell type classifiers (scTab/TabNet) on the CELLxGENE Merlin store and you need the cross-entropy loss and macro F1-score on unseen-donor test data, e.g. to compare training-dataset-size scaling regimes.

## When NOT to use

- Evaluating linear reference baselines for comparisons where the expected scTab-over-linear gap (0.0447 ± 0.0008 macro F1 at full data) is the validation target — the gap definition requires the nonlinear scTab model specifically
- Datasets whose cell types fail the corpus filters (e.g. fewer than 5000 cells per cell type, fewer than 30 donors per cell type, or labels not subtypes of 'native cell'), where the observed loss may be confounded by coarse or missing Cell Ontology annotations

## Inputs

- On-disk Merlin-format CELLxGENE store (22,189,056 cells x 19,331 protein-coding genes, size-factor normalized to 10,000 counts and log1p-transformed; census version 2023-05-15)
- Donor-based train/val/test split (0.7/0.15/0.15) with donor-based and cell-based subsample definitions (15%, 30%, 50%, 70%, 100% of training donors)
- Trained scTab/TabNet model checkpoint outputs (class probabilities per cell)

## Outputs

- Cross-entropy loss on the unseen-donor test set for each trained model/subsample
- Macro F1-score (F1 per cell type averaged over cell types) on the unseen-donor test set
- Tabulated loss and macro F1 versus training set size for donor-based vs. cell-based subsampling regimes, and loss curves

## How to apply

Preprocess each cell by size-factor normalization to 10,000 counts followed by log1p transformation, then read the 22,189,056 x 19,331-gene on-disk Merlin-format store (merlin_cxg_2023_05_15_sf-log1p) with the Nvidia Merlin dataloader. Train the TabNet classifier (19,331 input genes, cross-entropy loss, class-probability output) on train/val/test splits that are donor-based (0.7/0.15/0.15) so test cells come from unseen donors. After training, evaluate on the fixed donor-held-out test set, computing cross-entropy loss over predicted class probabilities and the macro F1-score (F1 per cell type averaged over cell types). When studying data scaling, run this under both donor-based subsampling (15%, 30%, 50%, 70%, 100% of training donors) and cell-based subsampling, then tabulate loss and F1 versus training size; a correct implementation reproduces strong loss/F1 improvement with size under donor-based subsampling but only weak improvement under cell-based subsampling, isolating batch/donor diversity — not cell count — as the scaling driver. The scTab repository provides dedicated notebooks for training, evaluation, and loss-curve comparison ('loss_curve_plotting').

## Related tools

- **scTab** (Training pipeline and model checkpoints used to produce class-probability outputs for cross-entropy loss and macro F1 evaluation) — github.com__theislab__scTab
- **TabNet** (Tabular classifier architecture (19,331 input genes, cross-entropy loss, class probabilities output) whose loss curves are evaluated)
- **NumPy** (Implements size-factor normalization (sf_normalize) to 10,000 counts and log1p transform that precede loss evaluation)

## Examples

```
import numpy as np; X = np.log1p(X / X.sum(axis=1, keepdims=True) * 10000)  # sf_normalize before TabNet training/eval
```

## Evaluation signals

- Loss and macro F1 improve strongly with training size under donor-based subsampling but only weakly under cell-based subsampling
- At full 15.2M-cell training size, scTab exceeds the linear reference model by 0.0447 ± 0.0008 macro F1
- Donor-based scaling from 2.1M to 15.2M cells yields a minimum organ-system macro F1 difference of 0.0454 ± 0.0083 (median 0.1219 ± 0.0212)
- Test-set evaluation is restricted to cells from unseen donors (donor-based 0.15 test split kept fixed across subsamples)
- Reproduced loss curves match those generated by the repository's loss_curve_plotting notebooks

## Limitations

- Plain macro F1-scores may not properly reflect the complexity of unbalanced datasets, which are imbalanced not only by cell type but also by tissue and sequencing protocol distribution
- Cell Ontology relationships are still under active discussion, which can affect the classification metrics; not all cell types (especially rare ones) match an ontology term
- Pre-trained scTab models still contain some fairly coarsely annotated cell type labels, and current ontologies lack resolution for brain tissue

## Evidence

- [abstract] scaling finding: "the performance of scTab scales both in terms of training dataset size and model size"
- [intro] TabNet architecture choice: [evidence span withheld — anchor status: not_found]
- [readme] loss curve notebooks: "``loss_curve_plotting``: Notebooks to plot and compare loss curves"
- [readme] evaluation notebooks: "``model_evaluation``: Notebooks containing all evaluation code from this paper"
- [methods] preprocessing steps: "1. Normalize each cell to have `10000` counts (size factor normalization)"
