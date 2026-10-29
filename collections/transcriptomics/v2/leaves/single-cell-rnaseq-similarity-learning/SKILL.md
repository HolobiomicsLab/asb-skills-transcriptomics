---
name: single-cell-rnaseq-similarity-learning
description: Use when you have a noisy single-cell RNA-seq expression matrix (with
  dropouts and outliers) and need a cell-to-cell similarity/distance metric for de
  novo clustering, dimension reduction, or 2-D/3-D visualization of heterogeneous
  populations.
when_to_use_negative:
- The goal is supervised analysis with pre-selected gene subsets of known function
  — SIMLR is unsupervised for de novo discovery
- The dataset is too large for the standard implementation without switching to SIMLR_Large_Scale
  (which additionally requires Rcpp, pracma, RcppAnnoy, RSpectra)
- The example .RData datasets are treated as substitutes for the original publication
  data — the README warns they are pre-processed examples only
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3525
- http://edamontology.org/topic_2229
tools:
- name: SIMLR
  role: core multi-kernel similarity-learning framework (SIMLR.R) combining kernels
    into a learned distance metric with graph diffusion
  repo: github.com/BatzoglouLabSU/SIMLR
- name: SIMLR_Large_Scale
  role: large-scale variant of the similarity-learning pipeline for high-throughput
    datasets (SIMLR_Large_Scale.R)
- name: SIMLR_Estimate_Number_of_Clusters
  role: estimates the number of clusters C from the eigenspectrum of the learned similarity
    via the eigenvalue-gap approach
- name: R
  role: implementation language; requires Matrix (sparse matrices) and parallel (parallel
    kernel estimation) packages, and igraph for NMI
  repo: https://bioconductor.org/packages/release/bioc/html/SIMLR.html
- name: SIMLR_Feature_Ranking
  role: ranks the most important features (genes) for the learned similarities
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:d26467ae3de5ca15b51f1c9e87569ec73d94ccd1a22d6770c58235630262cc2f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - SIMLR
  - SIMLR_Estimate_Number_of_Clusters
  - SIMLR_Feature_Ranking
  - SIMLR_Large_Scale
  - igraph
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3525
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# single-cell-rnaseq-similarity-learning

## Summary

Learn a sample-to-sample similarity matrix from single-cell RNA-seq expression data by combining multiple Gaussian kernels in SIMLR's optimization framework, then refining weak similarities via graph diffusion for clustering and visualization.

## When to use

Use when you have a noisy single-cell RNA-seq expression matrix (with dropouts and outliers) and need a cell-to-cell similarity/distance metric for de novo clustering, dimension reduction, or 2-D/3-D visualization of heterogeneous populations.

## When NOT to use

- The goal is supervised analysis with pre-selected gene subsets of known function — SIMLR is unsupervised for de novo discovery
- The dataset is too large for the standard implementation without switching to SIMLR_Large_Scale (which additionally requires Rcpp, pracma, RcppAnnoy, RSpectra)
- The example .RData datasets are treated as substitutes for the original publication data — the README warns they are pre-processed examples only

## Inputs

- single-cell RNA-seq expression matrix (.RData, e.g. Test_1_mECS.RData, Test_2_Kolod.RData, Test_3_Pollen.RData, Test_4_Usoskin.RData)
- target number of cell populations C (or estimated via SIMLR_Estimate_Number_of_Clusters)

## Outputs

- learned sample-to-sample similarity matrix (e.g. simlr_similarity_matrix.csv) with approximate block-diagonal structure
- dimension reduction / clustering / 2-D and 3-D visualization of the samples
- estimated number of clusters (e.g. estimated_number_of_clusters.csv)

## How to apply

Load the expression matrix (e.g. Test_1_mECS.RData from the data directory, using the Matrix package for sparsity) and set the target number of cell populations C. Compute multiple Gaussian kernel representations of cell-to-cell distances with kernel estimation parallelized via the parallel package, then run the SIMLR optimization loop (SIMLR.R) that combines the kernels into a learned distance metric under the rank constraint; compile the external C routine with 'R CMD SHLIB -c projsplx_R.c' for the projection step. Apply graph diffusion to the learned similarity to strengthen weak cell-to-cell similarities caused by noise and dropout events. If C is unknown, first run SIMLR_Estimate_Number_of_Clusters, which applies the eigenvalue-gap heuristic to the eigenspectrum of the learned similarity. Validate by checking the resulting sample-to-sample similarity matrix displays an approximate block-diagonal structure with C blocks corresponding to the populations.

## Related tools

- **SIMLR** (core multi-kernel similarity-learning framework (SIMLR.R) combining kernels into a learned distance metric with graph diffusion) — github.com/BatzoglouLabSU/SIMLR
- **SIMLR_Large_Scale** (large-scale variant of the similarity-learning pipeline for high-throughput datasets (SIMLR_Large_Scale.R))
- **SIMLR_Estimate_Number_of_Clusters** (estimates the number of clusters C from the eigenspectrum of the learned similarity via the eigenvalue-gap approach)
- **R** (implementation language; requires Matrix (sparse matrices) and parallel (parallel kernel estimation) packages, and igraph for NMI) — https://bioconductor.org/packages/release/bioc/html/SIMLR.html
- **SIMLR_Feature_Ranking** (ranks the most important features (genes) for the learned similarities)

## Examples

```
R CMD SHLIB -c projsplx_R.c; Rscript R_main_demo_SIMLR.R
```

## Evaluation signals

- The resulting sample-to-sample similarity matrix displays an approximate block-diagonal structure with C blocks corresponding to the known populations
- The estimated number of clusters from the eigenvalue-gap approach matches the known number of subpopulations in a validation dataset (e.g. Test_3_Pollen)
- projsplx_R.c compiles successfully via 'R CMD SHLIB -c projsplx_R.c' (remove pre-compiled projsplx_R.o/.so if compilation issues occur)
- Normalized mutual information (computed with the igraph package) confirms subpopulation separation better than PCA/t-SNE baselines
- The pipeline runs to completion on the demo script R_main_demo_SIMLR.R after installing Matrix and parallel packages

## Limitations

- Example datasets are pre-processed and 'should not be used in place of the ones provided in the respective publications'
- Single-cell data noise, outliers and dropouts weaken cell-to-cell similarities; the graph diffusion step mitigates but does not remove this
- If compilation of projsplx_R.c fails, pre-compiled files (projsplx_R.o, projsplx_R.so) must be removed; an OS X pre-compiled file is provided
- Standard implementation does not scale to very large datasets — SIMLR_Large_Scale with 4 additional packages (Rcpp, pracma, RcppAnnoy, RSpectra) is required (e.g. Test_5_Zeisel)

## Evidence
