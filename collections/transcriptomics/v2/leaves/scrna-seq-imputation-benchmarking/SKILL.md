---
name: scrna-seq-imputation-benchmarking
description: Use when evaluating whether an scRNA-Seq imputation method recovers cell-type
  structure or differential expression signal from dropout/NB-noise matrices, against
  AutoClass and control methods on known-truth data.
when_to_use_negative:
- Input data has no ground-truth labels or paired true counts, so ASW against known
  cell types or median DE t-statistics over truly DE genes cannot be computed
- The goal is applying one imputation method to new data rather than comparing multiple
  imputation methods head-to-head under controlled noise
- Data is not single-cell RNA-Seq count data (e.g., bulk RNA-Seq or proteomics), where
  the dropout/NB-noise simulation and imputation benchmark design do not apply
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: AutoClass
  role: Primary imputation method under benchmark; composite autoencoder + classifier
    network run via AutoClassImpute(dropout_rate=0.3)
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Control imputation method (deep count autoencoder with ZINB loss) in the head-to-head
    comparison
  repo: https://github.com/theislab/dca
- name: SAVER
  role: Control imputation method (regularized regression with empirical Bayes) in
    the comparison
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Control imputation method (Gaussian-Gamma mixture model) in the comparison
  repo: https://github.com/Vivianstats/scImpute
- name: MAGIC
  role: Control imputation method used in the Tian et al. cell-mixture benchmark (Fig.
    8)
  repo: https://github.com/KrishnaswamyLab/MAGIC
- name: Splatter
  role: 'Simulator generating paired true/noisy count matrices (Dataset 1: 1000 genes,
    500 cells, six groups; Dataset 8: 161 DE genes)'
- name: TensorFlow >=2.0
  role: Backend for building the AutoClass composite network and the classifier-free
    regular-autoencoder ablation
- name: numpy
  role: Numerical array handling for normalized matrices, PCA, and silhouette computation
- name: LuyiTian/sc_mixology
  role: Source of the Tian et al. cell mixture dataset (305 cells, 13,426 genes) used
    for the 1000/5000-gene noise benchmark
  repo: https://github.com/LuyiTian/sc_mixology
merged_aliases:
- scrnaseq-imputation-benchmarking
merged_alias_records:
- alias: scrna-imputation-method-benchmarking
  slug: scrna-imputation-method-benchmarking
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
- alias: scrna-seq-data-imputation
  slug: scrna-seq-data-imputation
  cosine_score: 0.8862
  method: embedding-cosine
  decision: needs_review
- alias: single-cell-rna-seq-imputation
  slug: single-cell-rna-seq-imputation
  cosine_score: 0.7629
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_008
  - task_004
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:37ad0e9e175cc3a355941013eaee94c5df09e78258b56199584c4648e8dc221f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - AutoImpute
  - DCA
  - LuyiTian/sc_mixology
  - MAGIC
  - Python
  - SAVER
  - Splatter
  - TensorFlow >=2.0
  - matplotlib
  - numpy
  - pandas
  - scImpute
  - scikit-learn
  - sklearn
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# scrna-seq-imputation-benchmarking

## Summary

A benchmarking workflow for comparing AutoClass against control imputation methods (DCA, SAVER, scImpute, MAGIC, regular autoencoder) on simulated Splatter datasets and the Tian et al. sc_mixology cell-mixture data, scoring recovery by average silhouette width (ASW) on t-SNE embeddings and median DE t-statistics.

## When to use

Use when evaluating whether an scRNA-Seq imputation method recovers cell-type structure or differential expression signal from dropout/NB-noise matrices, against AutoClass and control methods on known-truth data.

## When NOT to use

- Input data has no ground-truth labels or paired true counts, so ASW against known cell types or median DE t-statistics over truly DE genes cannot be computed
- The goal is applying one imputation method to new data rather than comparing multiple imputation methods head-to-head under controlled noise
- Data is not single-cell RNA-Seq count data (e.g., bulk RNA-Seq or proteomics), where the dropout/NB-noise simulation and imputation benchmark design do not apply

## Inputs

- Simulated Splatter scRNA-Seq count matrices with paired true (pre-noise) counts, e.g., Dataset1_raw.csv and Dataset1_true.csv (1000 genes, 500 cells, six groups)
- Dropout-containing or negative-binomial-noise count matrices on Tian et al. sc_mixology data (305 cells, 13,426 genes), subset to top-1000 and top-5000 highly variable genes
- Known cell-type/group labels for silhouette and DE evaluation

## Outputs

- asw_results.csv with average silhouette width scores per method, noise type, and gene-set size
- dataset8_de_benchmark_results.csv with median DE-gene t-statistics and sensitivity at specificity = 0.90 per method
- 2-D t-SNE embeddings (PCA-50 -> t-SNE-2) colored by group labels for each imputed matrix

## How to apply

Simulate paired true/noisy count matrices with Splatter (e.g., Dataset 1: 1000 genes, 500 cells, six groups; Dataset 8: two groups with 161 truly DE genes) or download Tian et al. sc_mixology data (305 cells, 13,426 genes) and subset to top 1000/5000 highly variable genes. Impute each noisy matrix with AutoClassImpute (dropout_rate=0.3) and controls (DCA, SAVER, scImpute, MAGIC; a classifier-free regular autoencoder for ablation). Normalize each matrix with take_norm, reduce with PCA(n_components=50), embed with TSNE(n_components=2), and compute average silhouette width (ASW) against known cell-type labels; for DE tasks, run per-gene two-sample T-tests between groups and report the median t-statistic over truly DE genes plus sensitivity at specificity=0.90. Accept the benchmark only if AutoClass attains the highest ASW across noise types and gene-set sizes, and a median DE t-statistic near the true-data value (e.g., ~5.86 vs 2.11 raw, matching true 5.79).

## Related tools

- **AutoClass** (Primary imputation method under benchmark; composite autoencoder + classifier network run via AutoClassImpute(dropout_rate=0.3)) — https://github.com/datapplab/AutoClass
- **DCA** (Control imputation method (deep count autoencoder with ZINB loss) in the head-to-head comparison) — https://github.com/theislab/dca
- **SAVER** (Control imputation method (regularized regression with empirical Bayes) in the comparison) — https://github.com/mohuangx/SAVER
- **scImpute** (Control imputation method (Gaussian-Gamma mixture model) in the comparison) — https://github.com/Vivianstats/scImpute
- **MAGIC** (Control imputation method used in the Tian et al. cell-mixture benchmark (Fig. 8)) — https://github.com/KrishnaswamyLab/MAGIC
- **Splatter** (Simulator generating paired true/noisy count matrices (Dataset 1: 1000 genes, 500 cells, six groups; Dataset 8: 161 DE genes))
- **TensorFlow >=2.0** (Backend for building the AutoClass composite network and the classifier-free regular-autoencoder ablation)
- **numpy** (Numerical array handling for normalized matrices, PCA, and silhouette computation)
- **LuyiTian/sc_mixology** (Source of the Tian et al. cell mixture dataset (305 cells, 13,426 genes) used for the 1000/5000-gene noise benchmark) — https://github.com/LuyiTian/sc_mixology

## Examples

```
X_norm = take_norm(X); res = AutoClassImpute(X, dropout_rate=0.3); pca = PCA(n_components=50).fit_transform(X_norm); tsne = TSNE(n_components=2).fit_transform(pca)
```

## Evaluation signals

- AutoClass must achieve the highest ASW across both the 1000-gene and 5000-gene subsets and both dropout and negative-binomial noise conditions on the Tian et al. data
- On Dataset 8, AutoClass should attain a median DE-gene t-statistic of ~5.86 (vs 2.11 raw, 5.79 true) and sensitivity at specificity = 0.90 of ~0.61 vs ~0.30 for raw data
- On the classifier-necessity ablation (Dataset 1), the AutoClass t-SNE embedding should show six visually separated groups with higher ASW than the regular autoencoder
- Imputed matrix must have the same gene-by-cell dimensions as the raw input and non-negative values
- Each method is evaluated on identically simulated noisy inputs after take_norm -> PCA(50) -> t-SNE(2), so scores are directly comparable

## Limitations

- Existing control methods focus largely on dropouts and carry strong distribution assumptions, which limits fair generalization of their performance across noise types
- Supplementary Tables 1-3 (hyperparameter and simulation parameters) are referenced but not included, so exact simulation settings must be sourced separately
- Method ranking may depend on the chosen evaluation metric (ASW on t-SNE, median DE t-statistic at specificity 0.90) rather than a universally accepted standard

## Evidence

- [abstract] AutoClass outperforms the state-of-art methods in multiple types of scRNA-Seq data analyses, including data recovery, differential expression analysis, clustering analysis, and batch effect removal: "AutoClass outperforms the state-of-art methods in multiple types of scRNA-Seq data analyses, including data recovery, differential expression analysis, clustering analysis"
- [methods] AutoClass runs with Python 3 and TensorFlow requirement: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed"
- [intro] we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups: "we simulated a scRNA-Seq Dataset 1 (see “Methods” and Supplementary Table 2) using Splatter13 with 1000 genes and 500 cells in six groups"
- [results] AutoClassImpute call with dropout rate: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [results] PCA then t-SNE reduction step: "PCA_raw = PCA(n_components=50).fit_transform(X_norm)
tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)"
- [readme] README overview of AutoClass features: "AutoClass integrates two deep neural network components, an autoencoder and a classifier, as to maximize both noise removal and signal retention"
