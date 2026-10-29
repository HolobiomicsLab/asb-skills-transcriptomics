---
name: dimension-reduction-visualization
description: Use when input is a single-cell or cohort expression matrix (e.g., scRNA-seq
  with noise/dropout, or TCGA patient expression) and you need unsupervised clustering
  plus 2-D visualization without pre-selecting gene subsets.
when_to_use_negative:
- Input is already a similarity/distance matrix or a reduced-dimension embedding rather
  than a raw expression matrix
- The sample-example .RData files in the repo are placeholders — do not use them in
  place of the datasets from the respective publications
- A supervised analysis with pre-selected gene subsets of known function is required;
  SIMLR is unsupervised and de novo
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: SIMLR
  role: Core multi-kernel similarity-learning, dimension reduction, clustering, and
    2-D/3-D visualization of the expression matrix
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: SIMLR_Large_Scale
  role: Scalable variant used for large datasets such as the GemCode PBMC matrix (hundreds
    of thousands of cells)
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: SIMLR_Feature_Ranking
  role: Ranks the most important features (genes) for the learned similarities
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: SIMLR_Estimate_Number_of_Clusters
  role: Estimates the number of populations/clusters from the data before stratification
    (e.g., on the TCGA diffuse lower-grade glioma cohort)
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: R
  role: Runtime environment; requires Matrix and parallel packages, plus Rcpp/pracma/RcppAnnoy/RSpectra
    for the large-scale mode
provenance:
  source_task_ids:
  - task_008
  - task_003
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:0ec11c8034197e4b7b0b24c42ca4501ef92345cb3dd8093568c0ec47d576683d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/BatzoglouLabSU/SIMLR
  tools:
  - R
  - SIMLR
  - SIMLR_Estimate_Number_of_Clusters
  - SIMLR_Feature_Ranking
  - SIMLR_Large_Scale
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# dimension-reduction-visualization

## Summary

Learn a sample-to-sample similarity measure via multi-kernel learning (SIMLR) and use it for dimension reduction, clustering, and 2-D/3-D visualization of heterogeneous populations, including large-scale single-cell data.

## When to use

Use when input is a single-cell or cohort expression matrix (e.g., scRNA-seq with noise/dropout, or TCGA patient expression) and you need unsupervised clustering plus 2-D visualization without pre-selecting gene subsets.

## When NOT to use

- Input is already a similarity/distance matrix or a reduced-dimension embedding rather than a raw expression matrix
- The sample-example .RData files in the repo are placeholders — do not use them in place of the datasets from the respective publications
- A supervised analysis with pre-selected gene subsets of known function is required; SIMLR is unsupervised and de novo

## Inputs

- scRNA-seq or cohort gene expression matrix in R (e.g., .RData example datasets such as Test_2_Kolod.RData, Test_3_Pollen.RData, or 10x Genomics GemCode PBMC matrix)
- Optionally, an expected or estimated number of clusters

## Outputs

- Cluster assignment per cell/patient (e.g., simlr_pbmc_clusters.csv, simlr_tcga_lgg_results.csv)
- 2-D embedding coordinates for visualization (e.g., simlr_tcga_lgg_2d.png)
- Learned multi-kernel sample-to-sample similarity matrix

## How to apply

Load the expression matrix into R and compile the external C routine with 'R CMD SHLIB -c projsplx_R.c' (remove stale projsplx_R.o/.so if compilation fails). Run SIMLR, which learns a distance metric by combining multiple kernels — a better fit to the noisy, dropout-laden statistical structure of scRNA-seq than PCA or t-SNE — and applies graph diffusion to strengthen weak similarities caused by noise or dropout events. If the number of populations is unknown, first run SIMLR_Estimate_Number_of_Clusters.R; for very large data (hundreds of thousands of cells, e.g., 10x Genomics GemCode PBMC), use SIMLR_Large_Scale instead, which requires the Rcpp, pracma, RcppAnnoy and RSpectra packages. Finally apply SIMLR's dimension reduction to embed cells (or patients) in 2-D for interpretation, and write cluster assignments, coordinates, and the similarity matrix to output files (e.g., CSV plus a 2-D PNG).

## Related tools

- **SIMLR** (Core multi-kernel similarity-learning, dimension reduction, clustering, and 2-D/3-D visualization of the expression matrix) — https://github.com/BatzoglouLabSU/SIMLR
- **SIMLR_Large_Scale** (Scalable variant used for large datasets such as the GemCode PBMC matrix (hundreds of thousands of cells)) — https://github.com/BatzoglouLabSU/SIMLR
- **SIMLR_Feature_Ranking** (Ranks the most important features (genes) for the learned similarities) — https://github.com/BatzoglouLabSU/SIMLR
- **SIMLR_Estimate_Number_of_Clusters** (Estimates the number of populations/clusters from the data before stratification (e.g., on the TCGA diffuse lower-grade glioma cohort)) — https://github.com/BatzoglouLabSU/SIMLR
- **R** (Runtime environment; requires Matrix and parallel packages, plus Rcpp/pracma/RcppAnnoy/RSpectra for the large-scale mode)

## Examples

```
library("devtools"); install_github("BatzoglouLabSU/SIMLR", ref = 'master'); library("SIMLR")
```

## Evaluation signals

- Clustering separates known subpopulations more accurately than existing dimension reduction methods (PCA, t-SNE), consistent with reported sensitivity/accuracy on PBMC data
- simlr_pbmc_clusters.csv (or equivalent) exists and contains a cluster label and 2-D coordinates for every input cell
- Estimated number of clusters from SIMLR_Estimate_Number_of_Clusters matches the number reported in the reference (e.g., SI-reported TCGA LGG stratification)
- Learned similarity matrix and 2-D plot are produced; normalized mutual information can be computed with the igraph package for demos

## Limitations

- Single-cell data's high level of noise, outliers and dropouts challenges conventional similarity measures; SIMLR mitigates but does not eliminate this
- The repo's example .RData datasets are pre-processed examples and 'should not be used in place of the ones provided in the respective publications'
- External C compilation (projsplx_R.c) can fail; stale projsplx_R.o and projsplx_R.so files may need to be removed

## Evidence

- [methods] SIMLR is capable of separating known subpopulations more accurately in single-cell data sets than do existing dimension reduction methods: "SIMLR is capable of separating known subpopulations more accurately in single-cell data sets than do existing dimension reduction methods"
- [readme] by employing graph diffusion, which improves weak similarity measures that are likely to result from noise or dropout events: "by employing graph diffusion, which improves weak similarity measures that are likely to result from noise or dropout events"
- [readme] we use SIMLR to provide 2-D and 3-D visualizations that assist with the interpretation of single-cell data: "we use *SIMLR* to provide 2-D and 3-D visualizations that assist with the interpretation of single-cell data derived from several diverse technologies"
- [readme] In order to compite the program, one needs to run on the shell the command R CMD SHLIB -c projsplx_R.c: "In order to compite the program, one needs to run on the shell the command *R CMD SHLIB -c projsplx_R.c*"
- [readme] high-throughput peripheral blood mononuclear cells (PBMC) data sets generated by the GemCode single-cell technology from 10x Genomics: "SIMLR demonstrates high sensitivity and accuracy on high-throughput peripheral blood mononuclear cells (PBMC) data sets generated by the GemCode single-cell technology"
