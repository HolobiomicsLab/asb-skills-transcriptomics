---
name: cell-cycle-phase-classification-svm
description: Use when you have single-cell imaging (4i) or single-cell RNA-seq data
  with a top-k ranked feature subset and ground-truth cell cycle phase labels, and
  need an accuracy-based comparison of feature selection methods.
when_to_use_negative:
- Ground-truth cell cycle phase annotations (manual gating labels) are unavailable
  — the classifier is supervised and needs labels_key in adata.obs for cross-validation
- The goal is unsupervised feature selection or trajectory inference itself rather
  than evaluating a ranked feature subset's phase-predictive performance
- Input data lacks an expression matrix of candidate features (e.g. already reduced
  to a feature table or non-tabular raw microscopy images)
edam_operation: http://edamontology.org/operation_2995
tools:
- name: DELVE
  role: Feature selection method producing the top-30 ranked feature subsets that
    the SVM classifier evaluates; expected to achieve the highest phase-classification
    accuracy
  repo: https://github.com/jranek/delve
- name: delve_benchmark
  role: Python package providing the fs class whose select()/evaluate_select() workflow
    ranks features and scores them on the trajectory/classification task; includes
    run_delve_fs and alternative fs functions
  repo: https://github.com/jranek/delve_benchmark
- name: Python
  role: Implementation environment for the SVM classifiers, nested cross-validation,
    and summary tables (scikit-learn dependency listed in the delve README)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:e90e8002ffa2fb0b0c2556675a8ea0267723ad4d7a93e11a616eb32921b6675c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - Python
  - UMAP (uniform manifold approximation and projection)
  - delve_benchmark
  edam_operation: http://edamontology.org/operation_2995
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# cell-cycle-phase-classification-svm

## Summary

Trains support vector machine classifiers with nested 10-fold cross-validation on top-30 feature-selected subsets to predict cell cycle phase labels (G0, G1, S, G2, M) in PDAC 4i protein immunofluorescence data, serving as the classification-accuracy criterion for benchmarking feature selection methods.

## When to use

Use when you have single-cell imaging (4i) or single-cell RNA-seq data with a top-k ranked feature subset and ground-truth cell cycle phase labels, and need an accuracy-based comparison of feature selection methods.

## When NOT to use

- Ground-truth cell cycle phase annotations (manual gating labels) are unavailable — the classifier is supervised and needs labels_key in adata.obs for cross-validation
- The goal is unsupervised feature selection or trajectory inference itself rather than evaluating a ranked feature subset's phase-predictive performance
- Input data lacks an expression matrix of candidate features (e.g. already reduced to a feature table or non-tabular raw microscopy images)

## Inputs

- Preprocessed single-cell .h5ad object per PDAC cell line (cells x 253 imaging-derived 4i features, from Zenodo record 10534873)
- Ranked list of top 30 selected features from one of the twelve feature selection methods (e.g. DELVE, Laplacian Score, Hotspot, HVG, MCFS, SCMER, random forest, all/random features, dynamic seed)
- Ground-truth cell cycle phase labels per cell (G0, G1, S, G2, M) from manual gating of core cell cycle markers, stored in adata.obs (labels_key)

## Outputs

- Per-cell predicted cell cycle phase labels (G0/G1/S/G2/M)
- Classification accuracy score for each (cell line, feature selection method) pair
- Aggregated summary table of classification accuracy across all 9 cell lines and 12 methods

## How to apply

Load a preprocessed PDAC 4i .h5ad file (d = 253 imaging-derived features per cell line from Zenodo record 10534873, e.g. BxPC3, CFPAC, HPAC, MiaPaCa, Pa01C, Pa02C, Pa16C, PANC1, UM53). Restrict the expression matrix to the top p = 30 features returned by a feature selection strategy (DELVE with k=10, num_subsamples=1000, n_clusters=10 for PDAC; or alternatives like Laplacian Score, Hotspot, HVG, MCFS, SCMER, random/all/random features). Train an SVM classifier using nested 10-fold cross-validation to predict the cell cycle phase labels (G0, G1, S, G2, M) annotated by manual gating of core cell cycle markers. Repeat for all twelve feature selection strategies and all nine cell lines, then aggregate accuracy into a summary table to determine which methods (expected: DELVE and Hotspot) best recover the cell cycle from noisy 4i data.

## Related tools

- **DELVE** (Feature selection method producing the top-30 ranked feature subsets that the SVM classifier evaluates; expected to achieve the highest phase-classification accuracy) — https://github.com/jranek/delve
- **delve_benchmark** (Python package providing the fs class whose select()/evaluate_select() workflow ranks features and scores them on the trajectory/classification task; includes run_delve_fs and alternative fs functions) — https://github.com/jranek/delve_benchmark
- **Python** (Implementation environment for the SVM classifiers, nested cross-validation, and summary tables (scikit-learn dependency listed in the delve README))

## Evaluation signals

- Nested 10-fold cross-validation must be used — accuracy must not be computed on a single held-out split
- Verify that DELVE and Hotspot achieve the highest median classification accuracy across the nine PDAC cell lines, consistent with the reported finding
- Each feature selection method must contribute exactly its top p = 30 features as classifier input
- Accuracy scores for each (cell line, method) pair must be recorded in the aggregated summary table covering 9 cell lines x 12 methods
- Phase label set must contain exactly G0, G1, S, G2, M as annotated by manual gating

## Limitations

- Feature selection can greatly bias interpretation of the underlying cellular trajectory, so careful consideration should be made when performing feature selection for trajectory analysis (article discussion)
- Classification accuracy depends on the quality of manually gated phase labels, which the article notes are expensive or laborious manual annotations often unavailable in practice
- Results are validated on PDAC 4i data with d = 253 features; accuracy rankings may not transfer to datasets with different feature counts or noise structure

## Evidence

- [abstract] feature selection quality criterion: "we present DELVE, an unsupervised feature selection method for identifying a representative subset of molecular features which robustly recapitulate cellular trajectories"
- [discussion] benchmark scope: "we benchmarked twelve feature selection methods29,49,53,55–58 on their ability to identify biologically relevant features for trajectory analysis from single-cell RNA sequencing data and proteomic"
- [discussion] expected ranking of similarity-based methods: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes within a cell type or along a cellular lineage"
- [readme] data access for .h5ad inputs: "You can download all of the preprocessed single-cell datasets (`.h5ad` files) from the [Zenodo](https://zenodo.org/records/10534873) repository."
- [readme] evaluation workflow via fs class: [evidence span withheld — anchor status: external_doc]
