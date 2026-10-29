---
name: cell-type-probability-classification
description: Use when single-cell expression data has a small labeled training set
  and you need probabilistic cell type predictions plus predictive entropy to guide
  cell selection or evaluate annotation uncertainty.
when_to_use_negative:
- When the goal is de novo clustering rather than supervised probability-based annotation
  of known or partially known cell types
- When no ground truth or training labels exist at all — entropy-based novelty detection
  here still requires a labeled training set to train the classifiers
- When a cell type is so rare that no cells can be sampled into the 20-cell training
  set — interpretation of entropy as active-learning signal rather than absence must
  then be verified
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: caret
  role: Trains the random forest and logistic regression classifiers on the first
    20 PCs of scaled and centered logcount expression data and produces cell type
    probability predictions
- name: leader
  role: R package framework for the cell selection heuristics (adaptive reweighting)
    that construct the balanced initial training sets used for classification
  repo: https://github.com/camlab-bioml/leader
- name: scuttle
  role: Calculates logcounts and size-factor normalization of the expression data
    prior to PCA and classification
- name: scran
  role: findMarkers function used for differential-expression-based marker definition
    between cell types
- name: Seurat
  role: Clustering used in adaptive reweighting to sample equal numbers of cells per
    cluster for building training sets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:26eb76383e754faa50b9e1ca1caef9b01a1f1ee704273e95011b554f2c7bdef6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - Seurat
  - caret
  - leader
  - scran
  - scuttle
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cell-type-probability-classification

## Summary

Train supervised classifiers (random forest and logistic regression via caret) on the first 20 principal components of scaled, centered logcount single-cell expression data to produce per-cell-type probability predictions, whose uncertainty (predictive entropy) can be scored against ground truth labels for active learning evaluation.

## When to use

Use when single-cell expression data has a small labeled training set and you need probabilistic cell type predictions plus predictive entropy to guide cell selection or evaluate annotation uncertainty.

## When NOT to use

- When the goal is de novo clustering rather than supervised probability-based annotation of known or partially known cell types
- When no ground truth or training labels exist at all — entropy-based novelty detection here still requires a labeled training set to train the classifiers
- When a cell type is so rare that no cells can be sampled into the 20-cell training set — interpretation of entropy as active-learning signal rather than absence must then be verified

## Inputs

- Log-normalized single-cell expression matrix (logcounts, e.g. SingleCellExperiment or Seurat object)
- Ground truth cell type labels for the labeled cells
- Small initial training set (e.g., 20 cells per cohort, with target cell type present 0-3 times)
- Optional marker gene file (yaml with positive/negative markers) for marker-informed training set construction

## Outputs

- Per-cell cell type probability vectors from random forest and logistic regression classifiers
- Scaled predictive entropy values per cell against ground truth labels
- Summary table (CSV) of median scaled entropies for removed versus present cell types, by cohort and classifier

## How to apply

Log-normalize the expression counts (e.g., with scuttle), scale and center them, and compute the first 20 principal components. Train a random forest and a logistic regression classifier on these PCs using the caret R package, then predict cell type probability vectors for unlabeled cells. Compute scaled predictive entropy from the predicted probabilities and compare against ground truth cell type labels. Decision point: cells (or cell types) with the highest entropy are candidates for labeling in an active learning loop — entropies rise sharply when a cell type is absent from the initial training set (most drastically for logistic regression), so entropy can flag completely unlabeled novel cell types.

## Related tools

- **caret** (Trains the random forest and logistic regression classifiers on the first 20 PCs of scaled and centered logcount expression data and produces cell type probability predictions)
- **leader** (R package framework for the cell selection heuristics (adaptive reweighting) that construct the balanced initial training sets used for classification) — https://github.com/camlab-bioml/leader
- **scuttle** (Calculates logcounts and size-factor normalization of the expression data prior to PCA and classification)
- **scran** (findMarkers function used for differential-expression-based marker definition between cell types)
- **Seurat** (Clustering used in adaptive reweighting to sample equal numbers of cells per cluster for building training sets)

## Examples

```
library(caret); fit <- train(pcs[,1:20], y = train_labels, method = "rf"); probs <- predict(fit, newdata = test_pcs, type = "prob"); entropy <- -rowSums(probs * log(probs))
```

## Evaluation signals

- Predictive entropies for a cell type excluded from the 20-cell training set should generally be higher than when 1, 2, or 3 cells of that type are included (most drastically for logistic regression)
- Predicted probability vectors must sum to 1 across cell types for each classified cell
- Median scaled entropy values aggregated by cohort and classifier should be reproducible and writable as a summary CSV
- Classifiers trained on 20-cell sets should achieve plausible accuracy when evaluated against stratified 50:50 train/test splits by cell type

## Limitations

- Ground truth cell type labels were taken from previous studies rather than manual annotation, creating ambiguity in what a ground-truth label means in single-cell biology
- Dropout at the single-cell level could complicate annotation when specific cell type markers are not expressed
- Circularity in marker definition for cell line data, as markers derive from differential expression between the cell lines

## Evidence

- [discussion] caret training on PCs: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package"
- [results] entropy detects unlabeled cell types (finding): [evidence span withheld — anchor status: not_found]
- [discussion] active learning loop selection rule: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [discussion] train/test split protocol: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [readme] leader package description (README): "Leader requires a processed seurat object with gene expression data and an optional marker file specifying known marker genes for known cell types."
- [discussion] ground truth caveat: "we relied here on “ground truth” cell type labels derived from previous studies"
