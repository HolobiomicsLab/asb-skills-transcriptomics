---
name: entropy-based-confident-cell-selection
description: Use when a small actively- or randomly-selected labeled training set
  exists for cell type annotation and additional unlabeled cells are available to
  boost classifier F1 via pseudo-labeling.
when_to_use_negative:
- No unlabeled cells remain — the training set already contains all available cells,
  so there is nothing to pseudo-label
- Cell types in the dataset are highly similar, where self-labeled cells may propagate
  mislabels rather than improve F1
- The goal is choosing which cells a human should manually annotate (active learning
  with ground-truth labeling) rather than automatic pseudo-labeling
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: caret
  role: Trains the logistic regression and random forest classifiers on the first
    20 principal components used to generate predictions and entropies for pseudo-labeling
- name: scuttle
  role: Computes logcounts and size factors as the expression normalization step before
    PCA and classifier training
- name: SingleR
  role: One of the annotation methods retrained on the combined ground-truth plus
    self-labeled training set
- name: scmap
  role: One of the annotation methods retrained on the combined ground-truth plus
    self-labeled training set
- name: leader (R package)
  role: R package implementing the heuristic cell selection approaches from the paper,
    within which this self-training step is reproduced
  repo: https://github.com/camlab-bioml/leader
- name: scran
  role: Supporting Bioconductor package used in the analysis workflow for single-cell
    preprocessing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:24064171f2f4623b3cc5426f75b3b4df25a33d8809ab0ef8fbdbc99a6c791332
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - SingleR
  - caret
  - leader
  - leader (R package)
  - scmap
  - scran
  - scuttle
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# entropy-based-confident-cell-selection

## Summary

A self-training (pseudo-labelling) step for single-cell annotation in which the remaining unlabeled training cells are predicted by trained classifiers and the top 10%, 50% or 100% most confidently predicted cells — ranked by prediction entropy — are relabeled with their predicted values and added to the ground-truth training set.

## When to use

Use when a small actively- or randomly-selected labeled training set exists for cell type annotation and additional unlabeled cells are available to boost classifier F1 via pseudo-labeling.

## When NOT to use

- No unlabeled cells remain — the training set already contains all available cells, so there is nothing to pseudo-label
- Cell types in the dataset are highly similar, where self-labeled cells may propagate mislabels rather than improve F1
- The goal is choosing which cells a human should manually annotate (active learning with ground-truth labeling) rather than automatic pseudo-labeling

## Inputs

- SingleCellExperiment or Seurat object with log-normalized expression, active-learning-selected ~100-cell training set, remaining unlabeled training cells, and held-out test set
- Ground-truth cell type labels for the initial training and test cells

## Outputs

- self_training_f1_improvement.csv (one row per annotation method x percentage of self-labeled cells)
- self_training_gain_correlation.csv (correlation between self-training F1 gain and baseline F1 across annotation methods)

## How to apply

Compute logcounts and size factors with scuttle, then train logistic regression and random forest classifiers in caret on the first 20 principal components of scaled and centered logcount expression. Predict the unlabeled training cells and rank them by prediction entropy, selecting the top 10%, 50% and 100% most confidently predicted cells, labeling them with the predicted values. Retrain each annotation method (SingleR, scmap, SingleCellNet, logistic regression, random forest) on the combined ground-truth plus self-labeled training set at each percentage level. Evaluate F1 on a stratified 50:50 held-out test set and compute the improvement relative to training on ground-truth labels only. Rationale: confidently self-labeled cells increase predictive performance especially when cell types are dissimilar and few cells are annotated.

## Related tools

- **caret** (Trains the logistic regression and random forest classifiers on the first 20 principal components used to generate predictions and entropies for pseudo-labeling)
- **scuttle** (Computes logcounts and size factors as the expression normalization step before PCA and classifier training)
- **SingleR** (One of the annotation methods retrained on the combined ground-truth plus self-labeled training set)
- **scmap** (One of the annotation methods retrained on the combined ground-truth plus self-labeled training set)
- **leader (R package)** (R package implementing the heuristic cell selection approaches from the paper, within which this self-training step is reproduced) — https://github.com/camlab-bioml/leader
- **scran** (Supporting Bioconductor package used in the analysis workflow for single-cell preprocessing)

## Examples

```
preds <- predict(rf_fit, newdata = unlabeled_pcs); ent <- apply(preds$votes, 1, function(p) -sum(p*log(p))); top <- names(sort(ent))[1:ceiling(0.1*length(ent))]
```

## Evaluation signals

- self_training_f1_improvement.csv contains one row per annotation method x percentage level (10%, 50%, 100% of self-labeled cells) and reproduces the direction and relative ordering of bars/curves in Fig. 5B (positive F1 gain for the most effective methods at 10-50% self-labeling)
- self_training_gain_correlation.csv reproduces the correlation trend in Fig. 5C between self-training F1 gain and baseline (ground-truth-only) F1
- F1 is computed on the stratified 50:50 held-out test set, not on the self-labeled training cells
- Improvement values are relative to training on ground-truth labels only with the same test set
- Most-confident cells are selected by lowest prediction entropy, and pseudo-labels are the classifier's predicted values (not ground truth)

## Limitations

- The optimal certainty threshold for selecting cells in self-training was not identified and is left to future work
- Ground truth cell type labels were taken from previous studies rather than manual annotation, introducing ambiguity in what a ground-truth label means
- Benchmarking favored breadth of tools and hyperparameters at the expense of investigating a large number of datasets

## Evidence

- [discussion] Self-training: predict remaining training cells, label most confident predictions, add to training set: "We then selected the top 10, 50 and 100% most confidently predicted cells based on their entropy and labeled these using the prediction values obtained."
- [discussion] Classifier training detail: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package"
- [discussion] Normalization step: "we calculated logcounts and normalized with the size factor using the scuttle R package"
- [abstract] Finding: [evidence span withheld — anchor status: not_found]
- [discussion] Future work limitation: "future work could focus on identifying the best certainty threshold to select cells to label using self-training"
