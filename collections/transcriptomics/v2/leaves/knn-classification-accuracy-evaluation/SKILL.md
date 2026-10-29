---
name: knn-classification-accuracy-evaluation
description: Use when you have selected a feature subset from single-cell data (e.g.,
  top p = 50–2000 genes per method) and need a quantitative, label-supervised metric
  of how well the subset preserves cell types or cell-type transitions.
when_to_use_negative:
- No ground-truth cell-type labels exist for the data — the kNN evaluation is label-supervised
  and cannot be computed (use pseudotime correlation or unsupervised evaluation instead)
- You want to evaluate feature selection on continuous trajectory progression rather
  than discrete cell-type identity (Kendall correlation against diffusion pseudotime
  vs. latent time is the appropriate metric)
- The input is already a performance/feature table rather than an expression matrix
  with labels
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: delve_benchmark
  role: Provides the fs class with evaluate_select() to perform feature selection
    and evaluate performance via classification against reference labels; includes
    run_delve_fs, laplacian_score_fs, hotspot_fs, hvg, variance_score, and other feature
    selection functions
  repo: https://github.com/jranek/delve_benchmark
- name: DELVE
  role: Unsupervised feature selection method whose output feature subsets are scored
    by kNN classification accuracy; installable as delve-fs
  repo: https://github.com/jranek/delve
- name: Python
  role: Implementation environment for the kNN (k = 3) stratified 3-fold cross-validation
    evaluation and PCA preprocessing (scikit-learn)
- name: SymSim
  role: R package (v0.0.0.9000) used to simulate tree differentiation trajectories
    (10,000 cells, 20,000 genes, 4 cell types) that supply ground-truth labels for
    the classification evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:0b10696c74809b01049a72c869d99d9559dc0ba116bae5537ac2a37200948905
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve_benchmark
  tools:
  - DELVE
  - Python
  - SymSim
  - delve_benchmark
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# knn-classification-accuracy-evaluation

## Summary

Evaluate whether selected molecular features preserve cell-type structure by training a k-nearest-neighbor classifier (k = 3) with 3-fold stratified cross-validation on the feature-selected data and recording classification accuracy, as used to benchmark twelve feature selection methods in the DELVE paper.

## When to use

Use when you have selected a feature subset from single-cell data (e.g., top p = 50–2000 genes per method) and need a quantitative, label-supervised metric of how well the subset preserves cell types or cell-type transitions.

## When NOT to use

- No ground-truth cell-type labels exist for the data — the kNN evaluation is label-supervised and cannot be computed (use pseudotime correlation or unsupervised evaluation instead)
- You want to evaluate feature selection on continuous trajectory progression rather than discrete cell-type identity (Kendall correlation against diffusion pseudotime vs. latent time is the appropriate metric)
- The input is already a performance/feature table rather than an expression matrix with labels

## Inputs

- Preprocessed single-cell annotated data object (.h5ad, cells x features) with ground-truth cell-type labels in adata.obs
- Ranked feature list from a feature selection method applied to a PCA embedding (n_pcs = 50)
- Feature threshold p (number of top features to select: 2000, 1000, 500, 250, 100, or 50)

## Outputs

- Per-method, per-threshold classification accuracy scores (e.g., a score_df dataframe from fs.evaluate_select())
- Aggregated median accuracy across datasets per method x feature-count combination (e.g., symsim_feature_selection_results.csv)

## How to apply

Take the ranked feature list output by a feature selection method (DELVE, Laplacian Score, Hotspot, neighborhood variance, MCFS, SCMER, max variance, HVG, all, or random) applied to a PCA embedding (n_pcs = 50) of a preprocessed .h5ad single-cell dataset, and subset the expression matrix to the top p features (p = 2000, 1000, 500, 250, 100, or 50). Train a k-nearest-neighbor classifier with k = 3 using 3-fold stratified cross-validation against ground-truth cell-type labels stored in adata.obs (labels_key of the delve_benchmark fs class), and record the median classification accuracy. Repeat across feature thresholds p to produce an accuracy-vs-subset-size curve per method, since the article shows similarity-based methods (DELVE, Laplacian Score, Hotspot) remain accurate at small subsets (e.g., 100 genes) while variance-based approaches need ~2000 genes. In the delve_benchmark package this corresponds to instantiating the fs class with the chosen fs_method, feature_threshold, and labels_key, then calling evaluate_select(), which returns a score_df of evaluation metrics. Aggregate median accuracy across datasets (e.g., five SymSim tree trajectories) and write results to a CSV such as symsim_feature_selection_results.csv.

## Related tools

- **delve_benchmark** (Provides the fs class with evaluate_select() to perform feature selection and evaluate performance via classification against reference labels; includes run_delve_fs, laplacian_score_fs, hotspot_fs, hvg, variance_score, and other feature selection functions) — https://github.com/jranek/delve_benchmark
- **DELVE** (Unsupervised feature selection method whose output feature subsets are scored by kNN classification accuracy; installable as delve-fs) — https://github.com/jranek/delve
- **Python** (Implementation environment for the kNN (k = 3) stratified 3-fold cross-validation evaluation and PCA preprocessing (scikit-learn))
- **SymSim** (R package (v0.0.0.9000) used to simulate tree differentiation trajectories (10,000 cells, 20,000 genes, 4 cell types) that supply ground-truth labels for the classification evaluation)

## Examples

```
from delve_benchmark.fs import fs; model = fs(adata=adata, fs_method=run_delve_fs, feature_threshold=100, labels_key='cell_type'); feats, score_df = model.evaluate_select()
```

## Evaluation signals

- Median kNN (k = 3, 3-fold stratified CV) accuracy for similarity-based methods (DELVE, Laplacian Score, Hotspot) at small subsets (e.g., p = 100) should exceed that of variance-based methods (max variance, HVG) at the same threshold
- Variance-based approaches should approach similar accuracy to similarity-based methods only near p = 2000 features
- Accuracy should degrade monotonically (or near-monotonically) as p shrinks from 2000 to 50 for weak feature selection methods such as random feature selection
- The 'all' and 'random' baselines should bracket the range of accuracies, with the 'all' (unselected) baseline providing the reference ceiling
- Output CSV (e.g., symsim_feature_selection_results.csv) must contain a row for every method x alpha_mean x p combination with finite accuracy values

## Limitations

- Feature selection can greatly bias interpretation of the underlying cellular trajectory; careful consideration should be made when performing feature selection for trajectory analysis
- Classification accuracy against cell-type labels requires supervised annotations that are often unavailable for real datasets
- Results depend on preprocessing choices (PCA with n_pcs = 50) and the number of selected features p; conclusions about method robustness are threshold-dependent

## Evidence

- [abstract] we present DELVE, an unsupervised feature selection method for identifying a representative subset of molecular features which robustly recapitulate cellular trajectories: "we present DELVE, an unsupervised feature selection method for identifying a representative subset of molecular features which robustly recapitulate cellular trajectories"
- [discussion] similarity-based methods achieve highest classification accuracy: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes within a cell type or along a cellular lineage"
- [readme] fs class evaluation entry point: [evidence span withheld — anchor status: not_found]
- [readme] evaluation uses reference labels via classification: [evidence span withheld — anchor status: external_doc]
- [readme] feature threshold controls subset size: [evidence span withheld — anchor status: external_doc]
