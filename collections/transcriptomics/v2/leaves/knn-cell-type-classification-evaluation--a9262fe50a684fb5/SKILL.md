---
name: knn-cell-type-classification-evaluation
description: Use when comparing feature selection methods on single-cell data with
  known ground-truth cluster labels (e.g., Splatter simulations) and you need a quantitative
  metric — k-NN classification accuracy — of how well the top-ranked selected features
  represent cell types.
when_to_use_negative:
- The dataset has no ground-truth cell-type annotations (e.g., unannotated real tissues)
  — the article notes supervised-like evaluation relies on labels that 'are often
  unavailable'; use trajectory-based metrics (e.g., diffusion pseudotime Kendall correlation)
  instead
- You need to evaluate how features preserve continuous trajectory/progression structure
  rather than discrete cell-type identity — use pseudotime/Kendall rank correlation
  or precision@k against ground-truth differentially expressed genes
- The goal is model performance itself rather than a comparative benchmark of feature
  selection methods (k-NN accuracy is an evaluation proxy, not a final classifier)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scikit-learn
  role: Provides the KNeighborsClassifier (k = 3) used to train on selected features
    and score cell-type classification accuracy against ground-truth labels
- name: delve_benchmark
  role: fs class with evaluate_select() performs feature selection and evaluates the
    method on the cell-type classification trajectory task; scikit-learn-based k-NN
    evaluation is implemented in its evaluation utilities
  repo: https://github.com/jranek/delve_benchmark
- name: Splatter (via scprep SplatSimulate)
  role: Generates simulated single-cell RNA-seq data with known ground-truth cluster
    labels (1500 cells, 500 genes, 6 clusters) against which k-NN accuracy is scored
- name: DELVE
  role: The feature selection strategy whose top-100 selected features are evaluated
    by the k-NN classification metric in the benchmark
  repo: https://github.com/jranek/delve
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:979d47f3c6e5e315bffa3d0b2f7f17d6bcb18baaaef2d54f02d4778bde5ade8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve_benchmark
  tools:
  - DELVE
  - Python
  - Scanpy
  - Splatter (via scprep SplatSimulate)
  - delve_benchmark
  - scikit-learn
  - scprep (SplatSimulate)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# knn-cell-type-classification-evaluation

## Summary

A benchmark evaluation that quantifies how well a subset of selected molecular features preserves cell-type identity by training a k-nearest neighbor classifier (k = 3, scikit-learn) on the selected features and recording cell-type classification accuracy against ground-truth cluster labels, as used to compare twelve feature selection methods on simulated single-cell RNA-seq data.

## When to use

Use when comparing feature selection methods on single-cell data with known ground-truth cluster labels (e.g., Splatter simulations) and you need a quantitative metric — k-NN classification accuracy — of how well the top-ranked selected features represent cell types.

## When NOT to use

- The dataset has no ground-truth cell-type annotations (e.g., unannotated real tissues) — the article notes supervised-like evaluation relies on labels that 'are often unavailable'; use trajectory-based metrics (e.g., diffusion pseudotime Kendall correlation) instead
- You need to evaluate how features preserve continuous trajectory/progression structure rather than discrete cell-type identity — use pseudotime/Kendall rank correlation or precision@k against ground-truth differentially expressed genes
- The goal is model performance itself rather than a comparative benchmark of feature selection methods (k-NN accuracy is an evaluation proxy, not a final classifier)

## Inputs

- Preprocessed single-cell annotated data object (.h5ad) or data matrix (cells x features) with ground-truth cluster/cell-type labels in adata.obs (labels_key) or a supplied labels array
- Ranked list of selected features (top 100 by the feature selection strategy under evaluation)
- Train/test cell splits (e.g., 3-fold cross-validation with stratified random sampling)
- Optionally, simulated data ground truth from Splatter (scprep SplatSimulate) with known cluster annotations across noise conditions (BCV = 0.1, 0.25, 0.5)

## Outputs

- Per-strategy, per-split cell-type classification accuracy of a k-NN (k = 3) classifier trained on selected features
- Aggregated median classification accuracy per feature selection method per noise condition (e.g., written to a results CSV)
- Ranked comparison of 12 feature selection strategies by k-NN cell-type classification performance

## How to apply

Within each 3-fold cross-validation split, first run the feature selection strategy of interest and retain only the top 100 ranked features. Train a k-nearest neighbor classifier (k = 3, scikit-learn, stratified random sampling) on the cells' selected features using the training folds and predict the held-out fold's cell-type labels; record accuracy against the ground truth cluster labels from adata.obs (or supplied labels array). Apply this identically to all strategies (e.g., DELVE, random forest, Laplacian Score, hotspot, MCFS, SCMER, max variance, HVG, all features, random) so scores are comparable. For Splatter simulations generated via scprep's SplatSimulate wrapper (1500 cells, 500 genes, 6 clusters per linear path, BCV = 0.1/0.25/0.5), use the true cluster annotations as ground truth and aggregate the median accuracy across the datasets per noise condition. The rationale is that features preserving cell-type identity should let a simple k-NN classifier recover labels even at low signal-to-noise ratios; DELVE achieved median accuracies of 0.937 (high), 0.882 (medium), and 0.734 (low SNR), the most robust among unsupervised methods.

## Related tools

- **scikit-learn** (Provides the KNeighborsClassifier (k = 3) used to train on selected features and score cell-type classification accuracy against ground-truth labels)
- **delve_benchmark** (fs class with evaluate_select() performs feature selection and evaluates the method on the cell-type classification trajectory task; scikit-learn-based k-NN evaluation is implemented in its evaluation utilities) — https://github.com/jranek/delve_benchmark
- **Splatter (via scprep SplatSimulate)** (Generates simulated single-cell RNA-seq data with known ground-truth cluster labels (1500 cells, 500 genes, 6 clusters) against which k-NN accuracy is scored)
- **DELVE** (The feature selection strategy whose top-100 selected features are evaluated by the k-NN classification metric in the benchmark) — https://github.com/jranek/delve

## Examples

```
from sklearn.neighbors import KNeighborsClassifier; knn = KNeighborsClassifier(n_neighbors=3); knn.fit(X_train[:, top100_features], y_train); acc = knn.score(X_test[:, top100_features], y_test)
```

## Evaluation signals

- Accuracy is computed on held-out folds from stratified random-sampling 3-fold cross-validation, so training and test sets must be label-stratified and disjoint
- Exactly the top 100 ranked features per strategy are used — a run selecting a different feature count is not comparable
- Identical classifier settings across all 12 methods: KNeighborsClassifier with k = 3 from scikit-learn; any hyperparameter drift invalidates the comparison
- Expected ranges on Splatter linear trajectories: median k-NN accuracy should decrease with noise (DELVE: 0.937 high SNR, 0.882 medium, 0.734 low SNR); the 'random' feature baseline should fall at or below similarity-based methods
- Median accuracy aggregated across the 10 simulation datasets per noise condition (30 datasets per condition across BCV levels) should be reported per method for a reproducible results table

## Limitations

- Evaluation requires ground-truth cell-type labels, which the article notes are 'often unavailable' for real data, precluding this evaluation outside simulated or annotated settings
- Feature selection 'can greatly bias the interpretation of the underlying cellular trajectory', so accuracy on selected features reflects the feature subset as much as the classifier
- Results are specific to the simulation regime (linear Splatter paths, 1500 cells, 500 genes, 6 clusters, BCV = 0.1–0.5) and may not transfer to other data modalities or trajectory shapes

## Evidence

- [methods] k-NN classifier trained on selected features: [evidence span withheld — anchor status: not_found]
- [readme] fs class evaluate_select for trajectory evaluation: [evidence span withheld — anchor status: not_found]
- [discussion] similarity-based methods achieved highest classification accuracy: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes within a cell type or along a cellular lineage"
- [discussion] DELVE benchmarked twelve feature selection methods: "we benchmarked twelve feature selection methods29,49,53,55–58 on their ability to identify biologically relevant features for trajectory analysis from single-cell RNA sequencing data"
