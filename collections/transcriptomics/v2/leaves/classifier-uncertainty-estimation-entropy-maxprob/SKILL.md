---
name: classifier-uncertainty-estimation-entropy-maxprob
description: Use when running an active learning loop for single-cell type annotation
  where you must decide which unlabeled cells (e.g., next 10 of many predicted on
  first 20 PCs) an expert should label next, using entropy or max-probability uncertainty
  from caret-trained RF/LR classifiers.
when_to_use_negative:
- The goal is selecting an initial balanced training set before any classifier exists
  (use leader adaptive reweighting, cluster-based or marker-based, instead of uncertainty
  ranking)
- Running self-training, where the most confidently predicted cells (top 10/50/100%
  by entropy) are pseudo-labeled — that uses low uncertainty, not high uncertainty
  ranking
- The expression matrix has no ground-truth labels available to annotate the selected
  10 cells at each iteration, since the loop requires labeling with ground truth
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: leader
  role: R package implementing the active learning workflow in which uncertainty-ranked
    cells are selected, labeled, and classifiers retrained
  repo: https://github.com/camlab-bioml/leader
- name: caret
  role: Trains the random forest and logistic regression classifiers whose predictions
    feed the entropy and max-probability uncertainty search
- name: SingleR
  role: Single-cell annotation method whose performance boost from uncertainty-driven
    active learning is quantified in the F1 comparisons
- name: scmap
  role: Single-cell annotation method combined with the active learning approaches
    for the F1-based evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:13332d505c2e365fa3b72312bf4c97eb2d2f120913a71c5e3d167a3d9d0f68b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - SingleR
  - caret
  - leader
  - scmap
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Classifier uncertainty estimation with entropy and maximum-probability search

## Summary

Ranks unlabeled single-cell predictions from random forest and logistic regression classifiers by predictive uncertainty (entropy and maximum-probability search) at highest, 95th-percentile, and 75th-percentile thresholds, so that the 10 most uncertain cells per active learning loop iteration are labeled with ground truth and used to retrain the classifiers.

## When to use

Use when running an active learning loop for single-cell type annotation where you must decide which unlabeled cells (e.g., next 10 of many predicted on first 20 PCs) an expert should label next, using entropy or max-probability uncertainty from caret-trained RF/LR classifiers.

## When NOT to use

- The goal is selecting an initial balanced training set before any classifier exists (use leader adaptive reweighting, cluster-based or marker-based, instead of uncertainty ranking)
- Running self-training, where the most confidently predicted cells (top 10/50/100% by entropy) are pseudo-labeled — that uses low uncertainty, not high uncertainty ranking
- The expression matrix has no ground-truth labels available to annotate the selected 10 cells at each iteration, since the loop requires labeling with ground truth

## Inputs

- Processed single-cell expression matrix (logcount, scaled and centered) with unique cell IDs
- Ground-truth cell type labels for the dataset (e.g., from Zenodo deposit 10.5281/zenodo.10403475)
- First 20 principal components of the scaled logcount expression data
- caret-trained random forest and logistic regression classifiers (initial 20-cell training set)
- Unlabeled cell predictions from each classifier at the current loop iteration

## Outputs

- Per-iteration F1 scores for each classifier/search-strategy/threshold combination
- Relative F1 improvement scores ((RF F1 - LR F1) / LR F1) averaged over ten splits and annotation methods
- relative_f1_improvement.csv with one row per search-strategy/threshold combination (entropy and max-probability; highest, 95th, 75th percentile)

## How to apply

Train a random forest and a logistic regression classifier with the caret R package on the first 20 principal components of scaled and centered logcount expression data, initialized from a random selection of 20 training cells after a stratified 50:50 train/test split. At each loop iteration, compute predictive uncertainty (entropy and maximum-probability search) over all unlabeled cells, apply each of the three thresholds (highest, 95th percentile, 75th percentile), and select the set of 10 cells with the highest uncertainty to label using ground truth data. Label those cells, retrain the classifier, and repeat, computing F1 scores per iteration. Derive the relative improvement score as (RF F1 minus LR F1) standardized by the LR F1, then average across the ten train-test splits and the SingleR/scmap annotation contexts to reproduce the Fig. 2A ordering of search-strategy/threshold combinations.

## Related tools

- **leader** (R package implementing the active learning workflow in which uncertainty-ranked cells are selected, labeled, and classifiers retrained) — https://github.com/camlab-bioml/leader
- **caret** (Trains the random forest and logistic regression classifiers whose predictions feed the entropy and max-probability uncertainty search)
- **SingleR** (Single-cell annotation method whose performance boost from uncertainty-driven active learning is quantified in the F1 comparisons)
- **scmap** (Single-cell annotation method combined with the active learning approaches for the F1-based evaluation)

## Examples

```
devtools::install_github("camlab-bioml/leader")
```

## Evaluation signals

- relative_f1_improvement.csv contains exactly one row per search-strategy/threshold combination (entropy and maximum-probability at highest, 95th-percentile, and 75th-percentile thresholds)
- With random 20-cell initialization, random forest F1 improvement over logistic regression is positive on average, matching the Fig. 2A ordering and magnitudes (Supplementary Figs. 4 and 5)
- Exactly 10 cells (the highest-uncertainty set) are labeled with ground truth per loop iteration before retraining
- F1 scores are averaged over ten stratified 50:50 train/test splits and both annotation methods (SingleR, scmap)
- Improvement values reproduce when initial cells are selected randomly under realistic conditions without guaranteed coverage of all cell types

## Limitations

- Ground-truth cell type labels came from previous studies rather than manual annotation, creating ambiguity in what a ground-truth label means; manual annotation was only simulated using existing annotations
- The benchmark prioritized breadth of hyperparameters and annotation tools over a large number of datasets, and experiments were not randomized or blinded
- Optimal certainty thresholds for the related self-training selection step were not identified and left to future work

## Evidence

- [discussion] Uncertainty selection drives the loop: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [discussion] Classifier training details: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package"
- [discussion] Split design: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [intro] Motivating research question: "what performance boost do active learning approaches deliver when combined with a range of popular single-cell annotation algorithms such as SingleR23 and scmap22?"
- [abstract] Tool availability: "An R package implementing the heuristic approaches introduced in this work may be found at https://github.com/camlab-bioml/leader"
