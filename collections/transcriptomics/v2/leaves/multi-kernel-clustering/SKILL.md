---
name: multi-kernel-clustering
description: Use when you need unsupervised clustering or sample stratification from
  a gene expression matrix where noise/dropout weaken standard distances, and the
  number of subpopulations is unknown.
when_to_use_negative:
- When input is not a raw expression matrix but an already-computed similarity/feature
  table — SIMLR learns its own multi-kernel metric from expression data and is unsupervised,
  so precomputed distances bypass its core step
- When clusters or cell types are already known and labeled — a supervised classification
  approach is more appropriate than SIMLR's de novo discovery
- When the dataset is very large-scale and standard SIMLR is too slow — use the SIMLR_Large_Scale
  variant instead of the standard implementation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SIMLR
  role: Core multi-kernel similarity-learning framework used for clustering, dimension
    reduction, and 2-D/3-D visualization of heterogeneous samples
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: SIMLR_Large_Scale
  role: Large-scale variant of SIMLR for handling large datasets within the same similarity-learning
    workflow
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: SIMLR_Estimate_Number_of_Clusters
  role: Script to estimate the number of clusters from the data before running SIMLR
    clustering
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: R
  role: Implementation environment; standard scripts SIMLR.R, SIMLR_Large_Scale.R,
    and SIMLR_Estimate_Number_of_Clusters.R run in R
  repo: https://github.com/BatzoglouLabSU/SIMLR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:e7d755e8a3d8438f74a7f8db7ecc31e52d3b5a66ed8fcaff0f17ef6b8b63141f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/BatzoglouLabSU/SIMLR
  tools:
  - R
  - SIMLR
  - SIMLR_Estimate_Number_of_Clusters
  - SIMLR_Large_Scale
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# multi-kernel-clustering

## Summary

Learn a sample-to-sample similarity measure by combining multiple kernels, as implemented in SIMLR, to cluster heterogeneous samples (single cells or tumor cohorts) and estimate the number of populations directly from expression data.

## When to use

Use when you need unsupervised clustering or sample stratification from a gene expression matrix where noise/dropout weaken standard distances, and the number of subpopulations is unknown.

## When NOT to use

- When input is not a raw expression matrix but an already-computed similarity/feature table — SIMLR learns its own multi-kernel metric from expression data and is unsupervised, so precomputed distances bypass its core step
- When clusters or cell types are already known and labeled — a supervised classification approach is more appropriate than SIMLR's de novo discovery
- When the dataset is very large-scale and standard SIMLR is too slow — use the SIMLR_Large_Scale variant instead of the standard implementation

## Inputs

- Gene expression matrix (rows = samples/cells, columns = genes), e.g., scRNA-seq count matrix or TCGA diffuse lower-grade glioma expression matrix
- Optional expected or candidate number of clusters (when not estimating from data)

## Outputs

- Learned multi-kernel sample-to-sample similarity matrix
- Cluster assignments per sample/cell
- Estimated number of clusters (from SIMLR_Estimate_Number_of_Clusters)
- 2-D (or 3-D) visualization of the learned similarity structure, e.g., simlr_tcga_lgg_2d.png
- Ranked features via SIMLR_Feature_Ranking (optional)

## How to apply

Load the expression matrix (e.g., single-cell scRNA-seq count data or a TCGA cohort such as the diffuse lower-grade glioma matrix from reference [12]) into R and install SIMLR from GitHub with install_github('BatzoglouSuperLab/SIMLR', ref='master'); before running, compile the external C code with 'R CMD SHLIB -c projsplx_R.c'. SIMLR learns a distance metric that best fits the data structure by combining multiple kernels, then applies graph diffusion to strengthen weak similarities arising from noise or dropout events. If the number of populations is unknown, run the SIMLR_Estimate_Number_of_Clusters.R script first, as done for the TCGA lower-grade glioma example, then run SIMLR on the expression matrix to learn the multi-kernel patient-to-patient (or cell-to-cell) similarity matrix. Produce 2-D (or 3-D) visualizations of the learned similarity structure to interpret the clusters; for large-scale datasets use the SIMLR_Large_Scale variant (requires Rcpp, pracma, RcppAnnoy, RSpectra). Validate cluster assignments against known labels or a published reference stratification, e.g., matching the SI-reported tumor stratification.

## Related tools

- **SIMLR** (Core multi-kernel similarity-learning framework used for clustering, dimension reduction, and 2-D/3-D visualization of heterogeneous samples) — https://github.com/BatzoglouLabSU/SIMLR
- **SIMLR_Large_Scale** (Large-scale variant of SIMLR for handling large datasets within the same similarity-learning workflow) — https://github.com/BatzoglouLabSU/SIMLR
- **SIMLR_Estimate_Number_of_Clusters** (Script to estimate the number of clusters from the data before running SIMLR clustering) — https://github.com/BatzoglouLabSU/SIMLR
- **R** (Implementation environment; standard scripts SIMLR.R, SIMLR_Large_Scale.R, and SIMLR_Estimate_Number_of_Clusters.R run in R) — https://github.com/BatzoglouLabSU/SIMLR

## Examples

```
install_github('BatzoglouLabSU/SIMLR', ref='master'); results <- SIMLR(X, c=n_clusters); SIMLR_Estimate_Number_of_Clusters(X)
```

## Evaluation signals

- Estimated number of clusters matches the known or SI-reported number of populations (e.g., the TCGA lower-grade glioma stratification in the Supplementary Materials)
- Cluster assignments reproduce a reference stratification (output_matches_reference against published labels or SI-reported clusters)
- Normalized mutual information against known subpopulations is computed (the demos require the igraph package for this metric) and is higher than baselines such as PCA or t-SNE
- Similarity matrix is symmetric with entries in [0,1] and the 2-D plot separates known subpopulations visually
- External C code compiled successfully (projsplx_R.so present) and the run completes without the compilation issues noted in the README

## Limitations

- SIMLR's multi-kernel approach is motivated by the large noise, outliers, and dropout events of single-cell data; performance claims are for such data, and example datasets in the repo are pre-processed and 'should not be used in place of the ones provided in the respective publications'
- The external C file projsplx_R.c may fail to compile; the README advises removing pre-compiled files (projsplx_R.o and projsplx_R.so) if issues occur
- SIMLR is unsupervised and does not use pre-selected gene subsets of known function, so it cannot directly incorporate prior biological annotations into clustering
- Required R packages (Matrix, parallel; plus Rcpp, pracma, RcppAnnoy, RSpectra for large scale) must be installed via install.packages or the install_R_libraries.R script before running

## Evidence

- [supplementary] Patient stratification finding on TCGA LGG: [evidence span withheld — anchor status: not_found]
