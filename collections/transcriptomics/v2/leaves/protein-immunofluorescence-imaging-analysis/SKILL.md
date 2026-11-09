---
name: protein-immunofluorescence-imaging-analysis
description: Use when analyzing single-cell protein immunofluorescence imaging measurements
  (stored as .h5ad, e.g. adata_RPE.h5ad) where the full feature panel includes noisy/irrelevant
  measurements and the goal is to identify features tied to cell cycle progression
  or other trajectories without labels.
when_to_use_negative:
- The input is already a curated, noise-free feature table where all measurements
  are known to be trajectory-relevant — unsupervised selection adds little over direct
  modeling.
- Supervised annotations are available and inexpensive, and the goal is pure prediction
  rather than discovering an unsupervised feature subset.
- The data are bulk measurements or lack per-cell resolution, since the methods rely
  on single-cell cell-similarity graphs.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0121
tools:
- name: DELVE
  role: Unsupervised feature selection (k=10, num_subsamples=1000, n_clusters=5) on
    the 241 imaging features to recover trajectory-preserving cell-cycle features
  repo: https://github.com/jranek/delve
- name: delve_benchmark
  role: fs class (select()/evaluate_select()) implementing the twelve feature-selection
    strategies and trajectory-preservation evaluations on the RPE 4i data
  repo: https://github.com/jranek/delve_benchmark
- name: scanpy
  role: Diffusion pseudotime trajectory inference (20 diffusion map components, k=10
    kNN graph) on the selected features
- name: scikit-learn
  role: SVM classifier/regressor with nested 10-fold cross-validation and KMeans++
    (c=4, 25 initializations) used to score selected features
- name: Hotspot
  role: One of the compared unsupervised similarity-based feature-selection baselines
- name: Slingshot
  role: R-based trajectory inference used alongside pseudotime evaluation of selected
    features
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:d4e1caa2b75af0377d20fc89cdf17584230433eb7353aa7c6b0332caa41b1589
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - Hotspot
  - Python
  - Slingshot
  - delve_benchmark
  - scanpy
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# protein-immunofluorescence-imaging-analysis

## Summary

Unsupervised feature selection and trajectory analysis on iterative protein immunofluorescence imaging data (e.g. the RPE 4i cell-cycle dataset of 2759 cells x 241 imaging features), using DELVE to pick trajectory-preserving features from noisy measurement panels and evaluating them against cell-cycle phase and age annotations.

## When to use

Use when analyzing single-cell protein immunofluorescence imaging measurements (stored as .h5ad, e.g. adata_RPE.h5ad) where the full feature panel includes noisy/irrelevant measurements and the goal is to identify features tied to cell cycle progression or other trajectories without labels.

## When NOT to use

- The input is already a curated, noise-free feature table where all measurements are known to be trajectory-relevant — unsupervised selection adds little over direct modeling.
- Supervised annotations are available and inexpensive, and the goal is pure prediction rather than discovering an unsupervised feature subset.
- The data are bulk measurements or lack per-cell resolution, since the methods rely on single-cell cell-similarity graphs.

## Inputs

- Preprocessed single-cell proteomic imaging data as .h5ad (adata_RPE.h5ad, 2759 cells x 241 features) from Zenodo record 10534873
- Ground-truth cell-cycle phase and age annotations in adata.obs
- Reference feature lists (random-forest-derived phase-specific features) for precision@30 evaluation

## Outputs

- Ranked top-30 feature list per feature-selection method (20 trials for DELVE reproducibility)
- Per-method metrics: median SVM cell-cycle phase classification accuracy, precision@30, NMI (KMeans++ c=4 vs. phase), age-prediction RMSE in hours, and Kendall correlation of diffusion pseudotime to age for proliferation (pRB-high) and arrest (pRB-low) lineages
- Summary CSV aggregating median metric values per method

## How to apply

Load the preprocessed .h5ad object (RPE 4i: 2759 cells x 241 features with ground-truth cell-cycle phase and age annotations) and verify dimensions before analysis. Run the feature-selection strategies (random forest, DELVE, dynamic seed, Laplacian Score, neighborhood variance, Hotspot, MCFS, SCMER, max variance, HVG, all features, random), selecting the top p=30 features each; run DELVE with k=10, num_subsamples=1000, n_clusters=5 across 20 random trials. Evaluate selected features by training an SVM classifier with nested 10-fold cross-validation to predict cell-cycle phase, computing precision@30 against random-forest-derived reference features, NMI between KMeans++ clusters (c=4, 25 initializations) and phase labels, and SVM-regressor RMSE (in hours) for cell-cycle age. Infer trajectories with diffusion pseudotime in Scanpy (20 diffusion map components, k=10 kNN graph, youngest-age root cells) over pRB-defined proliferation and arrest lineages and correlate pseudotime to age via Kendall rank correlation. The rationale is that filtering noisy, information-poor, or irrelevant imaging features prior to trajectory inference prevents them from dominating variance and distorting inferred dynamics.

## Related tools

- **DELVE** (Unsupervised feature selection (k=10, num_subsamples=1000, n_clusters=5) on the 241 imaging features to recover trajectory-preserving cell-cycle features) — https://github.com/jranek/delve
- **delve_benchmark** (fs class (select()/evaluate_select()) implementing the twelve feature-selection strategies and trajectory-preservation evaluations on the RPE 4i data) — https://github.com/jranek/delve_benchmark
- **scanpy** (Diffusion pseudotime trajectory inference (20 diffusion map components, k=10 kNN graph) on the selected features)
- **scikit-learn** (SVM classifier/regressor with nested 10-fold cross-validation and KMeans++ (c=4, 25 initializations) used to score selected features)
- **Hotspot** (One of the compared unsupervised similarity-based feature-selection baselines)
- **Slingshot** (R-based trajectory inference used alongside pseudotime evaluation of selected features)

## Examples

```
pip install delve-fs; import anndata; adata = anndata.read_h5ad('adata_RPE.h5ad')
```

## Evaluation signals

- Dataset integrity check: adata_RPE.h5ad loads with exactly 2759 cells x 241 features plus phase and age annotations in adata.obs
- DELVE should reproduce the reported median metrics on the RPE 4i data: SVM classification accuracy 0.960, precision@30 0.800, NMI 0.599, age RMSE 1.806 h, pseudotime Kendall correlation 0.656 (proliferation) / 0.405 (arrest)
- DELVE's median classification accuracy should approach the supervised random-forest ground-truth performance (0.957) and exceed all other unsupervised methods and the all-features baseline
- DELVE results across 20 random trials should show low spread, indicating reproducibility of the selected feature set
- Summary CSV must contain a median metric row per each of the twelve feature-selection strategies

## Limitations

- Feature selection can greatly bias interpretation of the underlying cellular trajectory, so careful consideration is needed when performing it for trajectory analysis
- DELVE could be improved by incorporating a set of previously established regulators
- All dynamic modules are currently leveraged equally when constructing the approximate cellular trajectory; excluding unwanted dynamic modules is possible but not implemented

## Evidence

- [abstract] DELVE abstract claim: "we present DELVE, an unsupervised feature selection method for identifying a representative subset of molecular features which robustly recapitulate cellular trajectories"
- [intro] Filtering rationale: "appropriate filtering of noisy, information-poor, or irrelevant features can serve as a crucial and necessary step for cell type identification, inference of dynamic phenotypes"
- [discussion] Benchmark scope including proteomic imaging: "we benchmarked twelve feature selection methods29,49,53,55–58 on their ability to identify biologically relevant features for trajectory analysis from single-cell RNA sequencing data and proteomic"
- [readme] Zenodo data access: "You can download all of the preprocessed single-cell datasets (`.h5ad` files) from the [Zenodo](https://zenodo.org/records/10534873) repository."
