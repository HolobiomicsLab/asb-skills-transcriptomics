---
name: self-training-semi-supervised-cell-annotation
description: Use when a small ground-truth-annotated single-cell training set (e.g.
  100 actively selected cells) exists but many unlabeled cells remain, and you want
  to test whether adding confidently self-labeled cells improves F1 cell type annotation.
when_to_use_negative:
- When the full dataset is already manually annotated and there are no unlabeled training
  cells left to pseudo-label
- When the goal is to select which NEW cells a human should annotate (that is active
  learning / adaptive reweighting with leader, not self-training)
- When you need an optimal certainty threshold for pseudo-label selection — the article
  explicitly leaves identifying the best certainty threshold to future work
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: leader (R package)
  role: R package accompanying the paper that supplies the active-learning-selected
    training sets and annotation/selection framework in which self-training is evaluated
  repo: https://github.com/camlab-bioml/leader
- name: caret
  role: Trains the random forest and logistic regression classifiers on the first
    20 PCs of scaled/centered logcount expression, used both to generate pseudo-labels
    and for retraining
- name: scuttle
  role: Computes logcounts and size factors for transcriptomic data prior to PCA and
    classification
- name: SingleR
  role: One of the annotation methods retrained on the combined ground-truth plus
    self-labeled training set
- name: scmap
  role: One of the annotation methods retrained on the combined ground-truth plus
    self-labeled training set
- name: scran
  role: Used for marker-based analysis (findMarkers) supporting the marker-aware variants
    benchmarked alongside self-training
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:68c2e469fb622478ae2db59ee90cb51da9c8542580be9cd6d7fc89b792f4b8cb
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# self-training-semi-supervised-cell-annotation

## Summary

A semi-supervised self-training (pseudo-labelling) procedure from Geuenich et al. 2024 (Nat Commun) in which a cell type classifier trained on a small annotated set predicts the remaining unlabeled cells, and its most confident predictions (lowest entropy) are added as pseudo-labeled training data before retraining annotation methods such as SingleR, scmap, SingleCellNet, logistic regression and random forest.

## When to use

Use when a small ground-truth-annotated single-cell training set (e.g. 100 actively selected cells) exists but many unlabeled cells remain, and you want to test whether adding confidently self-labeled cells improves F1 cell type annotation.

## When NOT to use

- When the full dataset is already manually annotated and there are no unlabeled training cells left to pseudo-label
- When the goal is to select which NEW cells a human should annotate (that is active learning / adaptive reweighting with leader, not self-training)
- When you need an optimal certainty threshold for pseudo-label selection — the article explicitly leaves identifying the best certainty threshold to future work

## Inputs

- Annotated training set (e.g. active-learning-selected 100-cell training sets with ground-truth cell type labels)
- Remaining unlabeled training cells from the same dataset
- Stratified 50:50 held-out test set with ground-truth labels
- Processed log-normalized single-cell expression data (SingleCellExperiment or Seurat object) with unique cell IDs

## Outputs

- self_training_f1_improvement.csv — one row per annotation method x percentage (10/50/100%) of self-labeled cells, matching Fig. 5B and Supplementary Figs. 34-39
- self_training_gain_correlation.csv — correlation between self-training F1 gain and baseline (ground-truth-only) F1 across methods, matching Fig. 5C

## How to apply

Train each annotation method on the ground-truth training cells (e.g. the active-learning-selected 100-cell sets), compute logcounts/size factors with scuttle, and for caret-based methods train random forest and logistic regression on the first 20 principal components of scaled and centered logcount expression. Predict the unlabeled training cells, rank them by prediction entropy, and select the top 10%, 50% and 100% most confident cells, assigning them their predicted labels. Retrain each method (SingleR, scmap, SingleCellNet, logistic regression, random forest) on the combined ground-truth plus pseudo-labeled set at each percentage level. Evaluate F1 on a stratified 50:50 held-out test set and record the F1 improvement over ground-truth-only training, per method x percentage of self-labeled cells (self_training_f1_improvement.csv, matching Fig. 5B), plus the correlation between self-training gain and baseline F1 across methods (self_training_gain_correlation.csv, Fig. 5C). The rationale is that confident pseudo-labels enlarge the effective training set at little annotation cost, and gains are expected especially when cell types are dissimilar and few cells are annotated.

## Related tools

- **leader (R package)** (R package accompanying the paper that supplies the active-learning-selected training sets and annotation/selection framework in which self-training is evaluated) — https://github.com/camlab-bioml/leader
- **caret** (Trains the random forest and logistic regression classifiers on the first 20 PCs of scaled/centered logcount expression, used both to generate pseudo-labels and for retraining)
- **scuttle** (Computes logcounts and size factors for transcriptomic data prior to PCA and classification)
- **SingleR** (One of the annotation methods retrained on the combined ground-truth plus self-labeled training set)
- **scmap** (One of the annotation methods retrained on the combined ground-truth plus self-labeled training set)
- **scran** (Used for marker-based analysis (findMarkers) supporting the marker-aware variants benchmarked alongside self-training)

## Examples

```
seu <- ScaleData(seu, features = all.genes); fit <- train(pcs_20, labels, method = "rf"); top <- order(pred_entropy)[1:round(0.1 * length(pred_entropy))]
```

## Evaluation signals

- self_training_f1_improvement.csv reproduces the direction and relative ordering of bars/curves in Fig. 5B: positive F1 gain for the most effective methods at 10-50% self-labeling
- self_training_gain_correlation.csv reproduces the trend in Fig. 5C (relationship between self-training gain and baseline F1 across methods)
- Pseudo-label selection must be entropy-ranked, restricted to the top 10%, 50% and 100% most confidently predicted unlabeled cells
- For caret classifiers, input must be the first 20 principal components of scaled and centered logcount expression (not raw counts)
- Evaluation must use the stratified 50:50 held-out test set, never the unlabeled cells used for pseudo-labeling

## Limitations

- Ground truth cell type labels were taken from previous studies rather than manual annotation; there is fundamental ambiguity in what a ground-truth cell type label means in single-cell biology
- The optimal certainty threshold for cell selection in self-training was not identified and is left to future work
- Self-training gains are conditional: improvement is expected especially for datasets with dissimilar cell types and when few cells are annotated
- Circularity in cell type marker definition, especially for cell line data where markers derive from differential expression between cell lines, may affect annotation baselines

## Evidence

- [discussion] Self-training selection by entropy with pseudo-labels: "We then selected the top 10, 50 and 100% most confidently predicted cells based on their entropy and labeled these using the prediction values obtained."
- [discussion] Caret classifiers on first 20 PCs: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package"
- [intro] SingleR and scmap as annotation algorithms combined with learning strategies: "what performance boost do active learning approaches deliver when combined with a range of popular single-cell annotation algorithms such as SingleR23 and scmap22?"
- [discussion] Ground truth labels derived from previous studies: "we relied here on “ground truth” cell type labels derived from previous studies"
- [readme] Leader package output used as training data: "The output of the `adaptive_reweighting` call can then be saved and the cell_id's can be used as more balanced training data."
