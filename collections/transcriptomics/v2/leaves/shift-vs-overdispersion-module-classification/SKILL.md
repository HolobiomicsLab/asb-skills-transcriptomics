---
name: shift-vs-overdispersion-module-classification
description: Use when quantifying gene set activity from a normalized expression matrix
  and needing to statistically distinguish coordinated-variance (overdispersed) from
  mean-shift (shifted) pathway activation in omics data.
when_to_use_negative:
- The input expression matrix is raw/unnormalized — rRoma works with normalized data
  (e.g. cpm or vst) before classification
- The goal is differential-expression-per-gene testing rather than gene-set/module-level
  activity quantification
- The gene set sizes are so heterogeneous that shared null distributions (ApproxSample
  log-spaced grid) would be a poor approximation and full per-size nulls are required
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: rROMA
  role: R package implementing the shifted vs overdispersed significance procedure
    (rRoma.R), producing the ModuleMatrix with ppv and q values
  repo: https://github.com/sysbio-curie/rROMA
- name: irlba
  role: Efficient partial SVD computing only the first principal component (PC1) for
    each gene set, yielding the L1 statistic
- name: mice
  role: Imputes missing values in the global expression matrix via iterative PCA before
    module scoring
- name: R
  role: Runtime environment for the rRoma package and its vignette workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:384382c1ac3d677cabc4c69165fb429131dfbda6da351eefc03af7bf6e5eb1a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - R
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# shift-vs-overdispersion-module-classification

## Summary

rROMA's empirical significance procedure that classifies each gene set as either 'shifted' (samples displaced along PC1, tested via Median Exp) or 'overdispersed' (coordinated variance, tested via L1 % variance explained), using null distributions from randomly generated gene sets of matched size and BH-corrected ppv q-values.

## When to use

Use when quantifying gene set activity from a normalized expression matrix and needing to statistically distinguish coordinated-variance (overdispersed) from mean-shift (shifted) pathway activation in omics data.

## When NOT to use

- The input expression matrix is raw/unnormalized — rRoma works with normalized data (e.g. cpm or vst) before classification
- The goal is differential-expression-per-gene testing rather than gene-set/module-level activity quantification
- The gene set sizes are so heterogeneous that shared null distributions (ApproxSample log-spaced grid) would be a poor approximation and full per-size nulls are required

## Inputs

- Normalized gene expression matrix (genes as rows, samples as columns), e.g. cpm-normalized from edgeR or vst from DESeq2
- Module list loaded from MSigDB (e.g. SelectFromMSIGdb('HALLMARK')) or a GMT file via ReadGMTFile
- Sample labels (e.g. sample_labels with sample_id and Type)

## Outputs

- ModuleMatrix table with L1, Median Exp, ppv L1, ppv Median Exp, q L1, q Median Exp per module
- Module classification labels: 'shifted' (ppv Median Exp <= 0.05) or 'overdispersed' (ppv L1 <= 0.05)
- rRoma.output object for downstream exploration (SelectGeneSets, GetTopContrib, PlotOutliers)

## How to apply

Center the normalized expression matrix globally (every gene mean zero across samples), since rROMA's PCA interpretation requires centered data. For each module (e.g. the 50 MSigDB v6.2 Hallmark sets), extract compatible genes and compute PC1 by SVD using irlba (PCADims = 2), deriving the L1 statistic (% variance explained by PC1) and the Median Exp statistic (median of sample projections on PC1). Build null distributions by sampling nSamples = 100 random gene sets of the same size — sharing background sets across modules whose sizes differ by less than the ApproxSamples percentage, on a log-spaced grid of sizes via the ApproxSample parameter. Compute empirical ppv p-values for overdispersion (ppv L1) and shift (ppv Median Exp) by comparing observed statistics against the null, then apply Benjamini-Hochberg correction (q L1, q Median Exp). Classify a module as shifted if ppv Median Exp <= 0.05 and as overdispersed if ppv L1 <= 0.05, assembling all statistics into the ModuleMatrix table.

## Related tools

- **rROMA** (R package implementing the shifted vs overdispersed significance procedure (rRoma.R), producing the ModuleMatrix with ppv and q values) — https://github.com/sysbio-curie/rROMA
- **irlba** (Efficient partial SVD computing only the first principal component (PC1) for each gene set, yielding the L1 statistic)
- **mice** (Imputes missing values in the global expression matrix via iterative PCA before module scoring)
- **R** (Runtime environment for the rRoma package and its vignette workflow)

## Examples

```
rRoma.output <- rRoma.R(MatData, AllHall)
```

## Evaluation signals

- Run rRoma.R on MatData with the 50 MSigDB v6.2 Hallmark sets and confirm 3 of 50 pathways (FATTY_ACID_METABOLISM, APICAL_SURFACE, COAGULATION) are called significant at p < 0.05
- Check that ModuleMatrix contains the expected columns: L1, Median Exp, ppv L1, ppv Median Exp, q L1, q Median Exp
- Verify classification rule: modules flagged shifted have ppv Median Exp <= 0.05, modules flagged overdispersed have ppv L1 <= 0.05
- Confirm ppv values are BH-corrected into q values and no module is classified on raw uncorrected ppv
- Check that null distributions were generated from nSamples = 100 random gene sets matched in size to each module

## Limitations

- Default hyperparameter values (e.g. extreme-weight percentage, outlier z-score threshold, ApproxSample) are not reported in the article and must be set by the user
- The CF case study lacked statistical power; authors note additional transcriptional datasets and multiple gene set reference databases would be needed
- rRoma relies on the mice package for missing-value imputation, but the README recommends using datasets with missing values already imputed
- Results depend on random gene set sampling, so the seed should be fixed for reproducibility (as done in the vignette)

## Evidence

- [intro] For each gene set, rROMA computes L1 and Median Exp via SVD-based PCA against null distributions from random gene sets; shifted if ppv Median Exp < 0.05, overdispersed if ppv L1 < 0.05, BH-corrected.: "estimate the statistical significance of the distribution of samples along the first component for a gene set in two ways: it distinguishes between shifted and over-dispersed gene sets"
- [discussion] rROMA uses a random gene set procedure to generate a null distribution for L1 and calculates p-values by comparison to the null.: "uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1 to the null distribution."
- [discussion] Null distributions are built for representative gene set sizes on a log-spaced grid via the ApproxSample parameter.: "rROMA constructs null distributions for a representative list of gene set sizes. These are selected to be uniformly distributed in the logarithmic scale"
- [discussion] rROMA starts by centering the global data matrix before computation.: "The rROMA algorithm therefore starts by centering the data of the global matrix."
- [readme] The vignette runs rRoma.R on MatData with the HALLMARK gene sets, fixing the random seed.: "rRoma.output <- rRoma.R(MatData, AllHall)"
- [discussion] Out of 50 hallmark pathways tested in the CF dataset, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION.: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
- [readme] rRoma works with normalized data; cpm from edgeR was used for normalisation.: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
