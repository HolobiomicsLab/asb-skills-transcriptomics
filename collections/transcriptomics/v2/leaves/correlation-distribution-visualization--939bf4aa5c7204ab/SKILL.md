---
name: correlation-distribution-visualization
description: Use when a gene-expression (GTEx/TCGA-style) feature table is available
  and the goal is to see how one focal feature correlates with all others, including
  the shape of the correlation distribution.
when_to_use_negative:
- The analysis goal is deep pairwise analysis of two specific features with scatter,
  linear regression, and density visualization — use FeatureCorr::pair_scatter instead
- Multiple features must be correlated pairwise with a clustered heatmap — use FeatureCorr::pairwise_corr
  instead
- Single-cell data with non-viable cells and low-abundance genes requiring read count
  filtration — FeatureCorr's transformations do not address this limitation
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: FeatureCorr
  role: R package providing data_transform and primefeature_corr functions for log2
    transformation and prime-feature correlation with distribution plotting
  repo: https://github.com/DKundnani/FeatureCorr
- name: devtools
  role: Used to install the FeatureCorr development version from GitHub
- name: ggplot2
  role: Rendering backend for the correlation coefficient distribution histogram produced
    by primefeature_corr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:ed8d7814f2585d5e74606dcb7c0130460352591b93e4dea50460f7c61aef8828
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
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# correlation-distribution-visualization

## Summary

Computes the correlation of one prime feature (e.g., the RNASEH2A gene) against all other features in an expression matrix using FeatureCorr::primefeature_corr, returning a coefficient/p-value table plus a histogram of the correlation coefficient distribution.

## When to use

Use when a gene-expression (GTEx/TCGA-style) feature table is available and the goal is to see how one focal feature correlates with all others, including the shape of the correlation distribution.

## When NOT to use

- The analysis goal is deep pairwise analysis of two specific features with scatter, linear regression, and density visualization — use FeatureCorr::pair_scatter instead
- Multiple features must be correlated pairwise with a clustered heatmap — use FeatureCorr::pairwise_corr instead
- Single-cell data with non-viable cells and low-abundance genes requiring read count filtration — FeatureCorr's transformations do not address this limitation

## Inputs

- Packaged GTEX_8000 Gene-Tissue Expression data frame (~8000 genes across 53 tissues) with a Description column used as featurelist
- Prime feature identifier matching a rowname of the transformed data frame (e.g., 'RNASEH2A')

## Outputs

- Data frame of correlation coefficients and p-values for the prime feature vs. all other features
- ggplot2 histogram of the correlation coefficient distribution (prime-feature-correlation-workflow figure)
- Distribution statistics computed by primefeature_corr

## How to apply

Install FeatureCorr from GitHub with devtools::install_github('DKundnani/FeatureCorr') and load the packaged GTEX_8000 Gene-Tissue Expression dataset (~8000 genes across 53 tissues). First preprocess with data_transform(df=GTEX[-1], transformation='log2', featurelist=GTEX$Description, medianthres=1), because log2 transformation reduces PCR artifacts and heteroskedasticity in sequencing data; extract the transformed data frame as the first item of the returned list. Then call primefeature_corr(df=inputdf, featurelist=rownames(inputdf), primefeature='RNASEH2A'), which calculates correlation coefficients, computes distribution statistics, and plots the correlation coefficient distribution. Write the coefficient/p-value table (one row per non-prime feature) to CSV and save the rendered histogram, then validate both outputs against the README's prime-feature-correlation-workflow figure.

## Related tools

- **FeatureCorr** (R package providing data_transform and primefeature_corr functions for log2 transformation and prime-feature correlation with distribution plotting) — https://github.com/DKundnani/FeatureCorr
- **devtools** (Used to install the FeatureCorr development version from GitHub)
- **ggplot2** (Rendering backend for the correlation coefficient distribution histogram produced by primefeature_corr)

## Examples

```
transdf <- data_transform(df=GTEX[-1],transformation='log2', featurelist=GTEX$Description, medianthres=1); inputdf<-transdf[[1]]; primefcorr <- primefeature_corr(df=inputdf,featurelist=rownames(inputdf),primefeature="RNASEH2A")
```

## Evaluation signals

- Output CSV exists and contains exactly one row per non-prime feature, with correlation coefficient and p-value columns
- Saved histogram PNG renders the correlation coefficient distribution and matches the package README prime-feature-correlation-workflow figure
- primefeature_corr console messages appear in order: 'Calculating Correlation Coefficients', 'Calculating Distribution Statistics', 'Plotting Correlation Coefficient Distribution'
- Input data frame was log2-transformed via data_transform with medianthres=1, and the transformed frame (first list item) is what is passed to primefeature_corr

## Limitations

- Single-cell experiments remain limited by non-viable cells and low-abundance genes that require read count filtration, which this workflow does not handle
- Log2 transformation is intended mainly for sequencing data to reduce PCR artifacts and heteroskedasticity; microarray data may warrant a different transformation
- The function observes one prime feature vs. many features only; pairwise or two-feature in-depth analysis requires pairwise_corr or pair_scatter

## Evidence

- [abstract] prime feature correlation and distribution histogram: "This method or function allows the user to observe the correlation of one feature with many features by providing a prime feature to be observed."
- [abstract] log2 rationale: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal"
- [methods] transformation call: "transdf<- data_transform(df=GTEX[-1],transformation='log2', featurelist=GTEX$Description, medianthres=1)"
- [readme] README prime feature call: "primefcorr <- primefeature_corr(df=inputdf,featurelist=rownames(inputdf) ,primefeature="RNASEH2A")"
- [readme] README transformed data extraction: "inputdf<-transdf[[1]] #First item in the list returned is the transformed Data"
