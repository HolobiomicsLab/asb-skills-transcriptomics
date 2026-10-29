---
name: active-learning-uncertainty-sampling
description: 'Use when annotating single-cell expression data with a limited labeling
  budget: you have ground-truth labels available on demand, a training classifier
  (e.'
when_to_use_negative:
- When no ground-truth labels can be obtained for the cells selected for labeling
  — the loop requires labeling the highest-uncertainty cells with ground truth data
- When the goal is to compare selection strategies without labeled data, e.g. purely
  unsupervised subset selection — adaptive reweighting via leader/Seurat clustering
  would be the appropriate comparison
- When the input is not log-normalized expression with identified/scaled variable
  features — the classifiers expect scaled, centered logcount data (first 20 PCs)
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: caret
  role: Trains the random forest and logistic regression classifiers on the first
    20 principal components of scaled and centered logcount expression, whose uncertainty
    drives cell selection
- name: scuttle
  role: Calculates logcounts and normalizes with size factors to produce the expression
    features used by the classifiers
- name: Seurat
  role: Used by the adaptive reweighting baseline (cluster-based sampling of equal
    numbers of cells per cluster) against which uncertainty sampling is compared;
    also provides object structure for leader
- name: leader
  role: R package implementing adaptive reweighting (cluster-based and marker-based
    cell selection), the baseline compared against active learning uncertainty sampling
  repo: https://github.com/camlab-bioml/leader
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:97c57abd138f0d98c53aed504561132eee7e935ffafa26718d9df651bcbaadcf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - Seurat
  - caret
  - leader
  - scuttle
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# active-learning-uncertainty-sampling

## Summary

An iterative cell-labeling strategy for single-cell type annotation in which, starting from a small training set, a classifier (random forest or logistic regression trained on scaled, centered logcount features) repeatedly selects the 10 cells with the highest prediction uncertainty to label, retrains, and re-evaluates. It maximizes annotation accuracy per labeled cell and was benchmarked against random selection and adaptive reweighting (leader/Seurat) under cell type imbalance.

## When to use

Use when annotating single-cell expression data with a limited labeling budget: you have ground-truth labels available on demand, a training classifier (e.g. random forest/logistic regression via caret), and want to prioritize labeling uncertain cells to boost balanced accuracy, especially in imbalanced datasets.

## When NOT to use

- When no ground-truth labels can be obtained for the cells selected for labeling — the loop requires labeling the highest-uncertainty cells with ground truth data
- When the goal is to compare selection strategies without labeled data, e.g. purely unsupervised subset selection — adaptive reweighting via leader/Seurat clustering would be the appropriate comparison
- When the input is not log-normalized expression with identified/scaled variable features — the classifiers expect scaled, centered logcount data (first 20 PCs)

## Inputs

- Processed single-cell expression matrix (log-normalized counts, e.g. Seurat object or SingleCellExperiment) with ground-truth cell type labels
- Train/test splits stratified by cell type (ten 50:50 splits per dataset)
- Optional artificial imbalanced dataset constructions (e.g. 450/50 two-type or 400/25x4 five-type per cohort) for imbalance benchmarking

## Outputs

- Per-iteration balanced accuracy on held-out test cells for each classifier (random forest, logistic regression) and each selection method
- Change in balanced accuracy between imbalanced and balanced datasets, standardized by the balanced dataset (e.g. imbalance_benchmark.csv matching Fig. 3C and Supplementary Figs. 13-30)

## How to apply

Randomly split the dataset into ten 50:50 train/test splits stratified by cell type. Initialize training with ~100 labeled cells, then at each iteration train random forest and logistic regression classifiers on the first 20 principal components of scaled and centered logcount expression (using the caret R package), select the set of 10 cells with the highest classifier uncertainty to label using ground truth, add them to the training set, retrain, and repeat. Evaluate by computing balanced accuracy on the held-out test split at each round. To quantify robustness to imbalance, compare the change in balanced accuracy between artificial imbalanced datasets (e.g. 450 vs 50 cells, or 100 majority vs 25 per minority type) and their balanced counterparts, standardized by the balanced dataset; active learning generally outperformed random selection and adaptive reweighting (Seurat cluster-based sampling via the leader package) in imbalanced settings, with larger drops for similar than distinct cell type pairs.

## Related tools

- **caret** (Trains the random forest and logistic regression classifiers on the first 20 principal components of scaled and centered logcount expression, whose uncertainty drives cell selection)
- **scuttle** (Calculates logcounts and normalizes with size factors to produce the expression features used by the classifiers)
- **Seurat** (Used by the adaptive reweighting baseline (cluster-based sampling of equal numbers of cells per cluster) against which uncertainty sampling is compared; also provides object structure for leader)
- **leader** (R package implementing adaptive reweighting (cluster-based and marker-based cell selection), the baseline compared against active learning uncertainty sampling) — https://github.com/camlab-bioml/leader

## Examples

```
seu <- NormalizeData(seu); seu <- FindVariableFeatures(seu, selection.method='vst', nfeatures=2000); seu <- ScaleData(seu, features=all.genes); model <- caret::train(pcs, labels, method='rf'); idx <- order(uncertainty, decreasing=TRUE)[1:10]
```

## Evaluation signals

- Balanced accuracy on each stratified 50:50 test split should increase monotonically (or near-monotonically) as uncertain cells are labeled and the classifier is retrained
- The active learning curve should reach a given balanced accuracy with fewer labeled cells than random selection on the same splits
- In imbalanced vs balanced dataset comparisons, the standardized change in balanced accuracy for active learning should be smaller (i.e. more robust) than for random and adaptive reweighting baselines
- The performance drop should be larger for artificial datasets built from similar cell type pairs than from distinct pairs
- Output file (e.g. imbalance_benchmark.csv) should reproduce the layout of Fig. 3C and Supplementary Figs. 13-30 across all selection methods, dataset constructions, and classifiers

## Limitations

- Ground truth cell type labels were derived from previous studies rather than manual annotation, so what a 'ground-truth' label means is fundamentally ambiguous in single-cell biology
- Dropout at the single-cell level could complicate annotation if specific cell type markers are not expressed; this was not addressed in the benchmarking
- Benchmarking prioritized breadth of hyperparameters and annotation tools at the expense of investigating a large number of datasets
- Active learning performance drops more for similar cell types than distinct ones under imbalance

## Evidence

- [methods] we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats.: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [methods] we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package"
- [methods] Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [methods] we created two imbalanced datasets with 450 cells of one type and 50 of another, with cell types that were similar and distinct from each other: "we created two imbalanced datasets with 450 cells of one type and 50 of another, with cell types that were similar and distinct from each other"
- [readme] An R package implementing the heuristic approaches introduced in this work may be found at https://github.com/camlab-bioml/leader: "An R package implementing the heuristic approaches introduced in this work may be found at https://github.com/camlab-bioml/leader"
