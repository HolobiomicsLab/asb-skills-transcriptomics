---
name: r-bioinformatics-package-workflow
description: Use when the goal is to correlate one 'prime' feature (e.g., a gene like
  RNASEH2A) against all other features in an NGS or microarray expression matrix,
  or to run data transformation/normalization first in R.
when_to_use_negative:
- When the analysis requires pairwise correlation of multiple features with clustering/heatmap
  visualization or single-pair scatter with linear regression — use FeatureCorr::pairwise_corr
  or pair_scatter instead of primefeature_corr
- When the input is not a feature-by-sample expression matrix from sequencing or microarray
  experiments (e.g., single-cell data with non-viable cells and low-abundance genes
  requiring read count filtration beyond medianthres)
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: FeatureCorr
  role: Core R package providing data_transform (log2/z-score normalization with medianthres
    filtration) and primefeature_corr (one feature vs many features correlation with
    distribution statistics and plotting)
  repo: https://github.com/DKundnani/FeatureCorr
- name: devtools
  role: Installs the development version of FeatureCorr from GitHub
  repo: https://github.com/DKundnani/FeatureCorr
- name: ggplot2
  role: Underlying plotting dependency used by FeatureCorr to render the correlation
    coefficient distribution histogram
- name: R
  role: Runtime environment for executing the FeatureCorr workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:4d20d01c4eb542e1348610e9fa3a509392ed801a5540d9508b1bf8c63d03ffaf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DKundnani/FeatureCorr
  tools:
  - FeatureCorr
  - R
  - devtools
  - ggplot2
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# r-bioinformatics-package-workflow

## Summary

A reproducible R workflow using the FeatureCorr package to preprocess gene expression data (log2 transformation with median threshold filtration) and compute prime feature correlations (e.g., RNASEH2A against GTEx features), returning correlation coefficients, p-values, and distribution plots.

## When to use

Use when the goal is to correlate one 'prime' feature (e.g., a gene like RNASEH2A) against all other features in an NGS or microarray expression matrix, or to run data transformation/normalization first in R.

## When NOT to use

- When the analysis requires pairwise correlation of multiple features with clustering/heatmap visualization or single-pair scatter with linear regression — use FeatureCorr::pairwise_corr or pair_scatter instead of primefeature_corr
- When the input is not a feature-by-sample expression matrix from sequencing or microarray experiments (e.g., single-cell data with non-viable cells and low-abundance genes requiring read count filtration beyond medianthres)

## Inputs

- Gene expression data frame (e.g., packaged GTEX Gene-Tissue Expression dataset with ~8000 genes across 53 tissues, or TCGA Pan Cancer data with 40 genes)
- Feature identifier list (e.g., GTEX$Description or rownames of the data frame)
- Prime feature name (e.g., 'RNASEH2A')
- Transformation parameters: transformation='log2', medianthres=1

## Outputs

- Data frame of correlation coefficients and p-values for the prime feature vs. all other features (e.g., rnaseh2a_prime_correlation.csv)
- Histogram of the correlation coefficient distribution (e.g., rnaseh2a_corr_histogram.png)
- Transformed (log2) data frame returned as the first item of the data_transform list

## How to apply

Install the development version of FeatureCorr from GitHub with devtools::install_github('DKundnani/FeatureCorr') and load it in R. Preprocess the expression data with FeatureCorr::data_transform using transformation='log2' (log2 is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity), passing the feature identifiers (e.g., GTEX$Description) as featurelist and medianthres=1 to filter low-abundance features; the first item of the returned list is the transformed data frame. Then run FeatureCorr::primefeature_corr on the transformed data frame with featurelist=rownames(inputdf) and the chosen primefeature (e.g., 'RNASEH2A'); the function calculates correlation coefficients, computes distribution statistics, and plots the correlation coefficient distribution. Write the coefficient/p-value table and save the rendered histogram for validation.

## Related tools

- **FeatureCorr** (Core R package providing data_transform (log2/z-score normalization with medianthres filtration) and primefeature_corr (one feature vs many features correlation with distribution statistics and plotting)) — https://github.com/DKundnani/FeatureCorr
- **devtools** (Installs the development version of FeatureCorr from GitHub) — https://github.com/DKundnani/FeatureCorr
- **ggplot2** (Underlying plotting dependency used by FeatureCorr to render the correlation coefficient distribution histogram)
- **R** (Runtime environment for executing the FeatureCorr workflow)

## Examples

```
transdf <- data_transform(df=GTEX[-1],transformation='log2', featurelist=GTEX$Description, medianthres=1); primefcorr <- primefeature_corr(df=transdf[[1]],featurelist=rownames(transdf[[1]]),primefeature="RNASEH2A")
```

## Evaluation signals

- Both output files exist: the correlation coefficient/p-value table (e.g., rnaseh2a_prime_correlation.csv) and the distribution histogram (e.g., rnaseh2a_corr_histogram.png)
- The correlation table contains one row per non-prime feature with correlation coefficient and p-value columns
- The histogram renders the RNASEH2A correlation coefficient distribution consistent with the README's prime-feature-correlation workflow figure
- The transformed data frame is the first item returned by data_transform and its rownames match the featurelist passed to primefeature_corr
- Console output from primefeature_corr shows the expected stages: 'Calculating Correlation Coefficients', 'Calculating Distribution Statistics', 'Plotting Correlation Coefficient Distribution'

## Limitations

- Single-cell experiments remain limited by non-viable cells and low abundance genes that require read count filtration beyond what data_transform's medianthres handles
- The packaged GTEX_8000 dataset is a sampled subset (~8000 genes, 53 tissues) of a larger dataset with >42K genes, so correlation results may differ from full-dataset runs
- Transformation choice matters: log2 is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity; other data types may need z-score or other transformations

## Evidence

- [methods] devtools::install_github("DKundnani/FeatureCorr"): "devtools::install_github("DKundnani/FeatureCorr")"
- [readme] data_transform call with log2 and medianthres=1: "transdf<- data_transform(df=GTEX[-1],transformation='log2', featurelist=GTEX$Description, medianthres=1)"
- [readme] primefeature_corr call with RNASEH2A: "primefcorr <- primefeature_corr(df=inputdf,featurelist=rownames(inputdf) ,primefeature="RNASEH2A")"
- [abstract] Log2 transformation rationale: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal or read count."
- [abstract] Prime feature correlation function purpose: "This method or function allows the user to observe the correlation of one feature with many features by providing a prime feature to be observed."
- [abstract] Single-cell filtration limitation: "there are still limitations to single-cell experiments due to non-viable cells and low abundance genes that require read count filtration"
