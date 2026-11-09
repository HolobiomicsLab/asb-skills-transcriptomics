---
name: prime-feature-correlation-statistics
description: Use when you need to observe the correlation of a single feature of interest
  against many features in an NGS or microarray expression table, obtaining per-feature
  coefficients and p-values plus a distribution plot.
when_to_use_negative:
- When you need pairwise correlation among many features with clustering/heatmap output
  rather than one-vs-many — use FeatureCorr::pairwise_corr instead
- When a deep two-feature analysis with scatter, linear regression, and density plots
  is required — use FeatureCorr::pair_scatter instead
- When the input table has not been preprocessed/normalized; primefeature_corr expects
  a transformed data frame (e.g., log2, medianthres=1 filtered)
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: FeatureCorr
  role: Provides the primefeature_corr function that computes correlation coefficients
    and distribution statistics of one prime feature against all other features, and
    data_transform for the required log2 preprocessing
  repo: github.com/DKundnani/FeatureCorr
- name: devtools
  role: Used to install the development version of FeatureCorr from GitHub
  repo: github.com/DKundnani/FeatureCorr
- name: ggplot2
  role: Underlies the rendered correlation coefficient distribution histogram output
    by primefeature_corr
- name: R
  role: Runtime environment in which the FeatureCorr workflow is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:fd94019f912ef6329392a5802a154fca301a1001efcdf67f8678590938c80331
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureCorr
  - R
  - devtools
  - ggplot2
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# prime-feature-correlation-statistics

## Summary

Computes correlation coefficients, distribution statistics, and a histogram of the correlation-coefficient distribution for one 'prime' feature (e.g., the RNASEH2A gene) against all other features in a transformed expression dataset, using the FeatureCorr R package's primefeature_corr function.

## When to use

Use when you need to observe the correlation of a single feature of interest against many features in an NGS or microarray expression table, obtaining per-feature coefficients and p-values plus a distribution plot.

## When NOT to use

- When you need pairwise correlation among many features with clustering/heatmap output rather than one-vs-many — use FeatureCorr::pairwise_corr instead
- When a deep two-feature analysis with scatter, linear regression, and density plots is required — use FeatureCorr::pair_scatter instead
- When the input table has not been preprocessed/normalized; primefeature_corr expects a transformed data frame (e.g., log2, medianthres=1 filtered)

## Inputs

- Transformed gene expression data frame (output of FeatureCorr::data_transform, first item of returned list)
- Feature list of gene identifiers (e.g., rownames(inputdf) or GTEX$Description)
- Name of the prime feature to correlate against all others (e.g., 'RNASEH2A')

## Outputs

- Data frame of correlation coefficients and p-values for the prime feature vs. every other feature
- Histogram (ggplot2 figure) of the correlation coefficient distribution
- Distribution statistics for the coefficient distribution

## How to apply

Install FeatureCorr from GitHub with devtools::install_github("DKundnani/FeatureCorr") and load its packaged GTEX_8000 Gene-Tissue Expression dataset (~8000 genes across 53 tissues). First normalize with data_transform(df=GTEX[-1], transformation='log2', featurelist=GTEX$Description, medianthres=1); log2 transformation reduces PCR artifacts and heteroskedasticity in sequencing data, and medianthres=1 filters low-abundance features. Extract the transformed data frame as the first item of the list returned by data_transform, then run primefeature_corr(df=inputdf, featurelist=rownames(inputdf), primefeature="RNASEH2A"), which reports progress through 'Calculating Correlation Coefficients', 'Calculating Distribution Statistics', and 'Plotting Correlation Coefficient Distribution'. Write the returned coefficient/p-value table (one row per non-prime feature) to CSV and save the rendered ggplot2 histogram of the correlation coefficient distribution. The prime feature can also be explored further per-pair with pair_scatter using a chosen correlation method such as 'pearson'.

## Related tools

- **FeatureCorr** (Provides the primefeature_corr function that computes correlation coefficients and distribution statistics of one prime feature against all other features, and data_transform for the required log2 preprocessing) — github.com/DKundnani/FeatureCorr
- **devtools** (Used to install the development version of FeatureCorr from GitHub) — github.com/DKundnani/FeatureCorr
- **ggplot2** (Underlies the rendered correlation coefficient distribution histogram output by primefeature_corr)
- **R** (Runtime environment in which the FeatureCorr workflow is executed)

## Examples

```
inputdf<-transdf[[1]]; primefcorr <- primefeature_corr(df=inputdf, featurelist=rownames(inputdf), primefeature="RNASEH2A")
```

## Evaluation signals

- Both output files exist: the correlation coefficient/p-value CSV and the distribution histogram PNG
- The result table contains exactly one row per non-prime feature, with correlation coefficient and p-value columns
- The histogram renders the RNASEH2A correlation coefficient distribution as shown in the package README prime-feature-correlation-workflow figure
- Console messages confirm the three internal stages: 'Calculating Correlation Coefficients', 'Calculating Distribution Statistics', 'Plotting Correlation Coefficient Distribution'
- Input to primefeature_corr is the first item of the list returned by data_transform, with featurelist equal to rownames(inputdf)

## Limitations

- Single-cell data has limitations due to non-viable cells and low abundance genes that require read count filtration, so raw single-cell counts may need filtering before this analysis
- Sequencing data exhibits heteroskedasticity (high variation in features with high signal/read count), so preprocessing/transformation such as log2 is needed before correlation
- The function addresses one feature vs. many features; it does not perform pairwise multi-feature correlation or per-pair scatter analysis (other package functions cover those)

## Evidence

- [abstract] Prime feature correlation analysis: "This method or function allows the user to observe the correlation of one feature with many features by providing a prime feature to be observed."
- [abstract] Log2 transformation reduces artifacts: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal or read count."
- [readme] primefeature_corr invocation from README: "primefcorr <- primefeature_corr(df=inputdf,featurelist=rownames(inputdf) ,primefeature="RNASEH2A")"
- [readme] data_transform parameters: "transdf<- data_transform(df=GTEX[-1],transformation='log2', featurelist=GTEX$Description, medianthres=1)"
- [methods] GTEX dataset description: "It contains Gene-Tissue Expression dataset with ~8000 genes(features) expression in 53 tissues"
