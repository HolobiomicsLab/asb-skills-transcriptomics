---
name: precision-at-k-feature-recovery-evaluation
description: Use when benchmarking a feature selection strategy that returns a ranked
  top-p feature list and you have ground truth labels (e.g. cell cycle phase) to build
  a supervised reference ranking.
when_to_use_negative:
- When no ground truth labels exist to train the supervised random forest reference
  — precision@k cannot be computed without reference features
- When evaluating a supervised method trained on the same ground truth labels used
  to build the reference (inflated overlap, as with random forest itself)
- When the goal is trajectory preservation (pseudotime Kendall correlation) rather
  than recovery of a specific labeled feature set — use classification accuracy, NMI,
  or pseudotime metrics instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_2229
tools:
- name: delve_benchmark
  role: Provides the fs class with select() and evaluate_select() methods and the
    random-forest-based reference feature computation used to compute precision@30
    per feature selection strategy
  repo: https://github.com/jranek/delve_benchmark
- name: DELVE
  role: Unsupervised feature selection method whose top-30 ranked features are scored
    by precision@30 against the reference set; run with k=10, num_subsamples=1000,
    n_clusters=10 (PDAC) or n_clusters=5 (RPE)
  repo: https://github.com/jranek/delve
- name: scikit-learn
  role: Supplies the random forest classifier whose Gini importance ranking defines
    the reference top-30 phase-predictive features
- name: Python
  role: Implementation language for the benchmarking pipeline and metric aggregation
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:6f21d2c473ac01fb8c97c9059930610488b76255c9cae1ce09191af485c951a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve_benchmark
  tools:
  - DELVE
  - Hotspot
  - Python
  - Slingshot
  - UMAP (uniform manifold approximation and projection)
  - delve_benchmark
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# precision-at-k-feature-recovery-evaluation

## Summary

Evaluate an unsupervised feature selection method by computing precision@30 against a reference feature set ranked by random forest Gini importance on ground truth labels, as used to benchmark DELVE and eleven alternatives on 4i protein imaging data.

## When to use

Use when benchmarking a feature selection strategy that returns a ranked top-p feature list and you have ground truth labels (e.g. cell cycle phase) to build a supervised reference ranking.

## When NOT to use

- When no ground truth labels exist to train the supervised random forest reference — precision@k cannot be computed without reference features
- When evaluating a supervised method trained on the same ground truth labels used to build the reference (inflated overlap, as with random forest itself)
- When the goal is trajectory preservation (pseudotime Kendall correlation) rather than recovery of a specific labeled feature set — use classification accuracy, NMI, or pseudotime metrics instead

## Inputs

- Preprocessed single-cell annotated data object (.h5ad, e.g. adata_RPE.h5ad or PDAC 4i files from Zenodo record 10534873) with cells x features matrix (e.g. 2759 cells x 241 features for RPE; d=253 for PDAC)
- Ground truth cell labels in adata.obs (e.g. cell cycle phase annotations from manual gating of core cell cycle markers)
- Candidate method's ranked top-30 selected feature list (or let fs.select() produce it)
- feature_threshold (p=30) and feature_threshold_reference (30) settings

## Outputs

- precision@30 score per feature selection method (fraction of the top-30 selected features found in the random-forest Gini-importance reference top-30)
- score_df dataframe from evaluate_select() containing precision@30 together with classification accuracy and NMI per method
- Aggregated summary table (CSV) of precision@30 across methods and cell lines/datasets

## How to apply

First build the reference list: train a random forest classifier on the ground truth labels (e.g. manually gated cell cycle phases G0/G1/S/G2/M in the PDAC 4i or RPE 4i datasets) using the full feature matrix from the preprocessed .h5ad files, and rank features by Gini importance, taking the top 30. Then take the candidate method's top 30 selected features (e.g. DELVE with k=10, num_subsamples=1000, n_clusters=10 for PDAC or n_clusters=5 for RPE) and compute precision@30 as the fraction of selected features overlapping the reference top-30. In the delve_benchmark package this is handled by the fs class: instantiate with fs_method and labels_key, then call select() and evaluate_select(), which returns the ranked predicted_features and a score_df containing precision@30 alongside classification accuracy and NMI. Aggregate the precision@30 per method across cell lines (e.g. all nine PDAC lines) to compare feature recovery quality; DELVE and hotspot are expected to achieve the highest precision@30 across cell lines, with DELVE reaching 0.800 on RPE 4i.

## Related tools

- **delve_benchmark** (Provides the fs class with select() and evaluate_select() methods and the random-forest-based reference feature computation used to compute precision@30 per feature selection strategy) — https://github.com/jranek/delve_benchmark
- **DELVE** (Unsupervised feature selection method whose top-30 ranked features are scored by precision@30 against the reference set; run with k=10, num_subsamples=1000, n_clusters=10 (PDAC) or n_clusters=5 (RPE)) — https://github.com/jranek/delve
- **scikit-learn** (Supplies the random forest classifier whose Gini importance ranking defines the reference top-30 phase-predictive features)
- **Python** (Implementation language for the benchmarking pipeline and metric aggregation)

## Examples

```
from delve_benchmark import fs; model = fs(adata, fs_method=run_delve_fs, fs_method_params={'k':10,'num_subsamples':1000,'n_clusters':5}, labels_key='phase', feature_threshold=30, feature_threshold_reference=30); model.evaluate_select()
```

## Evaluation signals

- DELVE achieves precision@30 ≈ 0.800 on RPE 4i; verify the computed value matches the reported median
- DELVE and hotspot should attain the highest median precision@30 across all nine PDAC cell lines relative to the other ten methods
- precision@30 must lie in [0, 1]; a value of 0 for a similarity-based method like DELVE or Laplacian score suggests a labels_key or feature-naming mismatch
- The reference feature list must contain exactly the top 30 features by random forest Gini importance — verify its length and that it was derived from ground truth labels, not from the candidate method
- Precision@30 for the random features baseline should approximate the chance overlap level (30/total_features), confirming the reference set is discriminative

## Limitations

- Precision@30 only measures overlap with a supervised reference ranking; features informative for the trajectory but outside the Gini top-30 are scored as misses
- The metric is threshold-sensitive: results at p=30 may not reflect performance at other feature budgets
- Feature selection can greatly bias interpretation of the underlying cellular trajectory; careful consideration should be made when performing feature selection for trajectory analysis
- Reference features depend on the quality of manual gating annotations of cell cycle markers; noisy or inconsistent labels degrade the reference ranking

## Evidence

- [other] Compute precision@30 by comparing each method's selected features against the reference set of top-30 phase-predictive features ranked by random forest Gini importance on ground truth phase annotations.: [evidence span withheld — anchor status: not_found]
- [readme] fs class API: [evidence span withheld — anchor status: external_doc]
- [readme] reference feature threshold parameter: [evidence span withheld — anchor status: external_doc]
- [results] reported DELVE precision on RPE: [evidence span withheld — anchor status: not_found]
- [discussion] cross-dataset validation: [evidence span withheld — anchor status: not_found]
