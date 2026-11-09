---
name: f1-benchmark-comparison-across-annotation-methods
description: Use when comparing cell type annotation classifiers on single-cell expression
  data with ground-truth labels, especially to measure whether adding confidently
  self-labeled (pseudo-labeled) cells improves held-out F1 relative to ground-truth-only
  training.
when_to_use_negative:
- The dataset has no held-out test set or ground-truth labels, so F1 cannot be computed
  against reference annotations
- The goal is unsupervised clustering or marker discovery rather than supervised classification
  performance benchmarking
- There is no unlabeled cell pool for pseudo-labeling, so self-training comparison
  reduces to a plain baseline benchmark
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: leader (R package)
  role: Implements the cell selection heuristics underlying the balanced training
    sets used in the annotation benchmark
  repo: https://github.com/camlab-bioml/leader
- name: caret
  role: Trains the logistic regression and random forest classifiers on the first
    20 PCs of scaled, centered logcount expression for F1 evaluation and pseudo-labeling
- name: scuttle
  role: Computes logcounts and size factors prior to PCA and classifier training
- name: SingleR
  role: Reference-based cell type annotation method benchmarked with default parameters
- name: scmap
  role: Reference-based cell type annotation method benchmarked with default parameters
- name: SingleCellNet
  role: Cell type classification method benchmarked with default parameters
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:c895178d1aa81a3bd273ce4a49a746962f3991a2358ec1bcd1e3cd542c50f524
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - SingleCellNet
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

# f1-benchmark-comparison-across-annotation-methods

## Summary

Benchmark single-cell type annotation methods (SingleR, scmap, SingleCellNet, logistic regression, random forest) by F1-score on a stratified held-out test set, including retraining on ground-truth plus pseudo-labeled cells to quantify self-training F1 gains as in Geuenich et al. 2024 (Nature Communications).

## When to use

Use when comparing cell type annotation classifiers on single-cell expression data with ground-truth labels, especially to measure whether adding confidently self-labeled (pseudo-labeled) cells improves held-out F1 relative to ground-truth-only training.

## When NOT to use

- The dataset has no held-out test set or ground-truth labels, so F1 cannot be computed against reference annotations
- The goal is unsupervised clustering or marker discovery rather than supervised classification performance benchmarking
- There is no unlabeled cell pool for pseudo-labeling, so self-training comparison reduces to a plain baseline benchmark

## Inputs

- Single-cell expression data (SingleCellExperiment/Seurat object) with ground-truth cell type labels
- Active-learning-selected training set (e.g., 100 cells) plus remaining unlabeled training cells
- Stratified 50:50 held-out test set with ground-truth labels
- Optional marker file (YAML of positive/negative markers per cell type) for marker-aware methods

## Outputs

- self_training_f1_improvement.csv (one row per annotation method x percentage of self-labeled cells: 10%, 50%, 100%)
- self_training_gain_correlation.csv (correlation between self-training F1 gain and baseline ground-truth-only F1 across methods)

## How to apply

Load active-learning-selected training sets (e.g., 100 cells), the remaining unlabeled cells, and a stratified 50:50 held-out test set. Compute logcounts and size factors with scuttle, train logistic regression and random forest via caret on the first 20 principal components of scaled, centered logcount expression, and run SingleR, scmap, and SingleCellNet with default parameters. Predict the unlabeled cells, rank by prediction entropy, and pseudo-label the top 10%, 50%, and 100% most confident predictions with the predicted values. Retrain each annotation method on the combined ground-truth plus self-labeled training set per percentage level, evaluate F1 on the held-out test set, and record improvement versus ground-truth-only training (one row per method x percentage, e.g., self_training_f1_improvement.csv matching Fig. 5B). Also compute the correlation between self-training F1 gain and baseline F1 across methods (self_training_gain_correlation.csv, Fig. 5C).

## Related tools

- **leader (R package)** (Implements the cell selection heuristics underlying the balanced training sets used in the annotation benchmark) — https://github.com/camlab-bioml/leader
- **caret** (Trains the logistic regression and random forest classifiers on the first 20 PCs of scaled, centered logcount expression for F1 evaluation and pseudo-labeling)
- **scuttle** (Computes logcounts and size factors prior to PCA and classifier training)
- **SingleR** (Reference-based cell type annotation method benchmarked with default parameters)
- **scmap** (Reference-based cell type annotation method benchmarked with default parameters)
- **SingleCellNet** (Cell type classification method benchmarked with default parameters)

## Examples

```
devtools::install_github("camlab-bioml/leader"); adaptive_reweighting(seu, n_requested_cells = 40, mode = "cluster_based")
```

## Evaluation signals

- The improvement table reproduces the direction and relative ordering of bars/curves in Fig. 5B (positive F1 gain for the most effective methods at 10-50% self-labeling)
- The correlation table reproduces the trend in Fig. 5C between self-training F1 gain and baseline F1
- One output row exists per annotation method x percentage of self-labeled cells (10%, 50%, 100%)
- Held-out test set is a stratified 50:50 split by cell type, as in the article's split scheme
- Baseline (ground-truth-only) F1 values match retraining the same classifiers on the 100-cell active-learning training sets

## Limitations

- Ground-truth cell type labels derive from previous studies rather than manual annotation, creating ambiguity about what a ground-truth label means
- Manual annotation was simulated using existing cell type annotations, not performed
- The optimal certainty threshold for selecting cells in self-training was not identified and is left to future work
- Circularity in marker definition, especially for cell line data where markers derive from differential expression between cell lines

## Evidence

- [methods] caret PCs training: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package"
- [methods] self-training selection: "We then selected the top 10, 50 and 100% most confidently predicted cells based on their entropy and labeled these using the prediction values obtained."
- [readme] leader input requirements: "Leader requires a processed seurat object with gene expression data and an optional marker file specifying known marker genes for known cell types."
- [methods] train/test split: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
