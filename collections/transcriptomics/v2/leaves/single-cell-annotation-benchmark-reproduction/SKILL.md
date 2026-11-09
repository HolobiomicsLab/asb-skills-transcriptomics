---
name: single-cell-annotation-benchmark-reproduction
description: Use when reproducing or extending the active learning benchmark for single-cell
  cell type annotation, specifically comparing RF vs LR classifiers with random initial
  20-cell training selection.
when_to_use_negative:
- When the goal is adaptive reweighting / LEADER cell selection (cluster-based or
  marker-based subsampling) rather than RF vs LR active learning comparison
- When ground-truth cell type labels are unavailable, since the loop labels the top-10
  uncertain cells using ground truth at each iteration
- When comparing self-training (most confident predictions) rather than uncertainty-based
  active learning search strategies
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: leader
  role: R package implementing the heuristic cell selection / active learning workflow
    used for the benchmark
  repo: https://github.com/camlab-bioml/leader
- name: caret
  role: Trains the random forest and logistic regression classifiers on the first
    20 principal components of scaled and centered logcount data
- name: SingleR
  role: Single-cell annotation algorithm whose labels are used as one of the annotation
    methods averaged in the relative improvement scores
- name: scmap
  role: Single-cell annotation algorithm whose labels are used as one of the annotation
    methods averaged in the relative improvement scores
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:4baccb4d2fc7f56eb738ef3f6883fceaf28c001fe9cf0b67217c923f040cabb6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - SingleR
  - caret
  - leader
  - scmap
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

# single-cell-annotation-benchmark-reproduction

## Summary

Reproduces the random forest vs logistic regression active learning comparison from Geuenich et al. 2024 (Nature Communications), in which random-forest active learning outperforms logistic regression when the initial 20-cell training set is selected randomly under realistic single-cell conditions.

## When to use

Use when reproducing or extending the active learning benchmark for single-cell cell type annotation, specifically comparing RF vs LR classifiers with random initial 20-cell training selection.

## When NOT to use

- When the goal is adaptive reweighting / LEADER cell selection (cluster-based or marker-based subsampling) rather than RF vs LR active learning comparison
- When ground-truth cell type labels are unavailable, since the loop labels the top-10 uncertain cells using ground truth at each iteration
- When comparing self-training (most confident predictions) rather than uncertainty-based active learning search strategies

## Inputs

- Processed single-cell expression matrices (logcount, scaled/centered)
- Ground-truth cell type labels (e.g., SingleR or scmap annotations) from Zenodo deposit 10.5281/zenodo.10403475
- Seurat objects with log-normalized expression and variable features identified

## Outputs

- relative_f1_improvement.csv with one row per search-strategy/threshold combination and averaged relative F1 improvement values
- Per-iteration F1 score trajectories for RF and LR active learning runs

## How to apply

Load processed single-cell expression matrices and ground-truth cell type labels from Zenodo deposit 10.5281/zenodo.10403475. Randomly split each dataset into ten 50:50 train/test splits stratified by cell type, and initialize each active learning run by randomly selecting 20 cells as the initial training set. Using the leader package workflow, train a random forest and a logistic regression classifier with the caret R package on the first 20 principal components of scaled and centered logcount expression data. At each loop iteration, rank unlabeled cells by predictive uncertainty (entropy and maximum-probability search) at the highest, 95th percentile, and 75th percentile thresholds, label the 10 cells with the highest uncertainty using ground truth, and retrain. Compute F1 scores across iterations and derive the relative improvement score as (RF F1 minus LR F1) standardized by the LR F1, averaged over the ten splits and annotation methods (SingleR, scmap).

## Related tools

- **leader** (R package implementing the heuristic cell selection / active learning workflow used for the benchmark) — https://github.com/camlab-bioml/leader
- **caret** (Trains the random forest and logistic regression classifiers on the first 20 principal components of scaled and centered logcount data)
- **SingleR** (Single-cell annotation algorithm whose labels are used as one of the annotation methods averaged in the relative improvement scores)
- **scmap** (Single-cell annotation algorithm whose labels are used as one of the annotation methods averaged in the relative improvement scores)

## Examples

```
devtools::install_github("camlab-bioml/leader"); library(leader)
```

## Evaluation signals

- relative_f1_improvement.csv contains exactly one row per search-strategy/threshold combination
- RF relative F1 improvement values are positive under random initial 20-cell selection, matching the ordering in Fig. 2A
- Magnitudes of relative improvement match those reported in Fig. 2A and Supplementary Figs. 4-5
- Classifiers are trained on exactly 20 principal components of scaled and centered logcount data
- Each run uses ten 50:50 stratified train/test splits with a random initial training set of 20 cells

## Limitations

- Ground truth cell type labels were taken from previous studies rather than manual annotation, so what a ground-truth label means is ambiguous
- Manual annotation was only simulated using existing cell type annotations, not performed
- No statistical method was used to predetermine sample size, experiments were not randomized, and investigators were not blinded
- Benchmarking chose many hyperparameters and annotation tools at the expense of investigating a large number of datasets

## Evidence

- [methods] we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package"
- [methods] Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [methods] we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats.: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [abstract] An R package implementing the heuristic approaches introduced in this work may be found at https://github.com/camlab-bioml/leader: "An R package implementing the heuristic approaches introduced in this work may be found at https://github.com/camlab-bioml/leader"
- [discussion] we relied here on “ground truth” cell type labels derived from previous studies: "we relied here on “ground truth” cell type labels derived from previous studies"
