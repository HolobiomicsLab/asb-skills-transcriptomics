---
name: variance-explained-quantification
description: Use when PCA on a multi-study microarray or expression matrix is suspected
  to be dominated by batch (e.g. study of origin) and you must measure variance explained
  per PC and covariate–PC correlations to confirm removal.
when_to_use_negative:
- Data are single-series with no batch/study-of-origin variable, so PC dominance by
  covariates cannot be compared before vs after adjustment
- Expression values are raw probe intensities that have not been log-transformed and
  limma-normalized across studies, making cross-study PCA meaningless
- The analysis goal is RNA-seq or ChIP-seq (curatedAdipoRNA/curatedAdipoChIP count
  matrices) rather than the microarray data this batch-removal workflow was demonstrated
  on
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: curatedAdipoArray
  role: Source of the curated SummarizedExperiment microarray data on which PCA is
    computed
  repo: https://travis-ci.org/MahShaaban/curatedAdipoArray.svg?branch=master
- name: ExperimentHub
  role: Retrieves the clean and processed SummarizedExperiment data objects into R
- name: limma
  role: Log-transformation and cross-study normalization of gene intensities prior
    to PCA
- name: sva
  role: Empirical Bayes (ComBat-style) removal of known batch effects (data series
    and platform) before recomputing PCA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:f33dffa2b0b197dde3bff084305c57712ca31064033f942390cf5cdce103876e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ExperimentHub
  - R
  - R/Bioconductor
  - curatedAdipoArray
  - limma
  - sva
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# variance-explained-quantification

## Summary

Quantify per-principal-component variance explained and sample correlations with covariates (e.g. study of origin, stage of differentiation) on a normalized gene-expression matrix, before and after batch removal, to verify which variables dominate the embedding.

## When to use

Use when PCA on a multi-study microarray or expression matrix is suspected to be dominated by batch (e.g. study of origin) and you must measure variance explained per PC and covariate–PC correlations to confirm removal.

## When NOT to use

- Data are single-series with no batch/study-of-origin variable, so PC dominance by covariates cannot be compared before vs after adjustment
- Expression values are raw probe intensities that have not been log-transformed and limma-normalized across studies, making cross-study PCA meaningless
- The analysis goal is RNA-seq or ChIP-seq (curatedAdipoRNA/curatedAdipoChIP count matrices) rather than the microarray data this batch-removal workflow was demonstrated on

## Inputs

- SummarizedExperiment object from curatedAdipoArray retrieved via ExperimentHub (log-normalized, probe-to-gene collapsed expression matrix)
- Sample metadata (colData) with study/series of origin, platform, and stage of differentiation

## Outputs

- Per-principal-component variance-explained and covariate-correlation table (before vs after batch removal)
- Two PCA plots: pre-adjustment (study of origin dominant) and post-adjustment (stage of differentiation separating samples on PC2/PC3)

## How to apply

Load the curatedAdipoArray SummarizedExperiment object via ExperimentHub and subset to the series of interest (here the Pparg-knockdown datasets GSE12929, n=48, and GSE14004, n=9), retaining stage-of-differentiation metadata. Run PCA on the limma-normalized, probe-to-gene-collapsed (collapseRows) log expression matrix and record variance explained per PC plus correlations of samples with study of origin and stage of differentiation. Then remove known batch effects (data series and platform) with sva's empirical Bayes (ComBat-style) adjustment and recompute PCA on the corrected matrix. Correct application is judged by the reported pattern: study of origin explains the most variance before adjustment, while stage of differentiation corresponds to PC2/PC3 and separates samples after adjustment.

## Related tools

- **curatedAdipoArray** (Source of the curated SummarizedExperiment microarray data on which PCA is computed) — https://travis-ci.org/MahShaaban/curatedAdipoArray.svg?branch=master
- **ExperimentHub** (Retrieves the clean and processed SummarizedExperiment data objects into R)
- **limma** (Log-transformation and cross-study normalization of gene intensities prior to PCA)
- **sva** (Empirical Bayes (ComBat-style) removal of known batch effects (data series and platform) before recomputing PCA)

## Examples

```
BiocManager::install("curatedAdipoArray"); se <- curatedAdipoArray::... # via ExperimentHub; pca <- prcomp(t(log_norm_matrix)); summary(pca)$importance[, 1:3]
```

## Evaluation signals

- Before adjustment, study of origin explains the most variance across principal components in the GSE12929/GSE14004 subset
- After empirical Bayes batch removal, stage of differentiation corresponds to PC2 and PC3 and successfully separates the samples
- Variance-explained/correlation table contains per-PC values for both the original and batch-corrected matrices, enabling a side-by-side diff
- PCA is computed on the log-normalized, probe-to-gene-collapsed matrix (missing and low-intensity genes removed), matching the curation pipeline
- Both pre- and post-adjustment PCA plots are generated and saved alongside the summary table

## Limitations

- Batch removal targets known batch effects only (data series and platform); unknown batch factors are not addressed by the sva adjustment
- Successful stage-of-differentiation separation was demonstrated on a specific subset (two Pparg-knockdown series, n=48 and n=9); it is not guaranteed for other subsets
- The demonstration depends on the curated metadata: two datasets were excluded from the resource for missing probe annotations, and missing/low-intensity genes were removed before normalization

## Evidence

- [full_text] Before batch removal, study of origin explained the most variance; after applying empirical Bayes adjustment, stage of differentiation corresponded to the second and third principal components and successfully separated the samples.: [evidence span withheld — anchor status: not_found]
- [intro] the known batch effects (data series and platform) were removed using `sva`.: "the known batch effects (data series and platform) were removed using `sva`."
- [intro] missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`.: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [readme] Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`.: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [abstract] Finally, we used a subset of the data to effectively remove batch effects and reproduce biological observations.: "Finally, we used a subset of the data to effectively remove batch effects and reproduce biological observations."
