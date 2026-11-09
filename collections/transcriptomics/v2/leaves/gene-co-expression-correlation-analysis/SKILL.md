---
name: gene-co-expression-correlation-analysis
description: Use when a gene-expression or feature table (e.g., GTEx Gene-Tissue Expression
  data) needs to be queried for which features correlate with one prime feature, with
  coefficients and p-values for each.
when_to_use_negative:
- The goal is pairwise correlation of multiple features against multiple features
  or deep two-feature scatter analysis — use pairwise_corr or pair_scatter instead
  of the prime-feature mode
- The input data are raw counts that should not be log2-transformed (e.g., microarray
  data where z-score may be more appropriate), since the medianthres=1 log2 filter
  targets sequencing read-count data
- Single-cell read-count data with non-viable cells and low-abundance genes requiring
  read count filtration prior to correlation analysis
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: FeatureCorr
  role: R package providing the data_transform (log2, medianthres=1 filtering) and
    primefeature_corr functions used for the prime feature correlation workflow
  repo: https://github.com/DKundnani/FeatureCorr
- name: R
  role: Runtime environment in which the FeatureCorr workflow is executed
- name: devtools
  role: Used to install the development version of FeatureCorr from GitHub
- name: ggplot2
  role: Graphics backend underlying the correlation coefficient distribution histogram
    output
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:cc56054b6063c424035923e063a4a31f0305189eef8b32aa4228cf1192c423df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DKundnani/FeatureCorr
  tools:
  - FeatureCorr
  - R
  - devtools
  - ggplot2
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# gene-co-expression-correlation-analysis

## Summary

Computes the correlation of one prime feature (e.g., the RNASEH2A gene) against all other features in a gene expression dataset using the FeatureCorr R package, returning correlation coefficients, p-values, distribution statistics, and a histogram of the coefficient distribution.

## When to use

Use when a gene-expression or feature table (e.g., GTEx Gene-Tissue Expression data) needs to be queried for which features correlate with one prime feature, with coefficients and p-values for each.

## When NOT to use

- The goal is pairwise correlation of multiple features against multiple features or deep two-feature scatter analysis — use pairwise_corr or pair_scatter instead of the prime-feature mode
- The input data are raw counts that should not be log2-transformed (e.g., microarray data where z-score may be more appropriate), since the medianthres=1 log2 filter targets sequencing read-count data
- Single-cell read-count data with non-viable cells and low-abundance genes requiring read count filtration prior to correlation analysis

## Inputs

- Feature expression data frame (e.g., packaged GTEX_8000 Gene-Tissue Expression dataset with ~8000 genes across 53 tissues)
- Feature list / feature identifiers (e.g., GTEX$Description or rownames of the transformed data frame)
- Prime feature name (e.g., 'RNASEH2A')
- Transformation parameters (transformation='log2', medianthres=1) for preprocessing

## Outputs

- Data frame of correlation coefficients and p-values for the prime feature vs. all other features (e.g., rnaseh2a_prime_correlation.csv)
- Distribution statistics of the correlation coefficients
- Histogram plot of the correlation coefficient distribution (e.g., rnaseh2a_corr_histogram.png)

## How to apply

Load the FeatureCorr package (installed via devtools::install_github("DKundnani/FeatureCorr")) together with the packaged GTEX_8000 Gene-Tissue Expression dataset (~8000 genes across 53 tissues). First preprocess with data_transform using transformation='log2' and medianthres=1, passing GTEX$Description as the featurelist, and extract the transformed data frame as the first item of the returned list — log2 transformation is used with sequencing data to reduce PCR artifacts and heteroskedasticity. Then run primefeature_corr on the transformed data frame with primefeature='RNASEH2A' and featurelist=rownames(inputdf) to calculate correlation coefficients, distribution statistics, and plot the correlation coefficient distribution. The function computes correlation of one feature with many features by providing a prime feature to be observed.

## Related tools

- **FeatureCorr** (R package providing the data_transform (log2, medianthres=1 filtering) and primefeature_corr functions used for the prime feature correlation workflow) — https://github.com/DKundnani/FeatureCorr
- **R** (Runtime environment in which the FeatureCorr workflow is executed)
- **devtools** (Used to install the development version of FeatureCorr from GitHub)
- **ggplot2** (Graphics backend underlying the correlation coefficient distribution histogram output)

## Examples

```
transdf<- data_transform(df=GTEX[-1],transformation='log2',featurelist=GTEX$Description,medianthres=1); primefcorr <- primefeature_corr(df=transdf[[1]],featurelist=rownames(transdf[[1]]),primefeature="RNASEH2A")
```

## Evaluation signals

- Both output files exist: the correlation coefficient/p-value table (e.g., rnaseh2a_prime_correlation.csv) and the histogram (e.g., rnaseh2a_corr_histogram.png)
- The table contains one row per non-prime feature with correlation coefficient and p-value columns
- The histogram renders the RNASEH2A correlation coefficient distribution as shown in the package README prime-feature-correlation workflow figure
- The transformed data frame is the first item of the list returned by data_transform, and featurelist matches rownames(inputdf)
- Console output follows the expected sequence: 'Calculating Correlation Coefficients', 'Calculating Distribution Statistics', 'Plotting Correlation Coefficient Distribution'

## Limitations

- Single-cell experiments are limited by non-viable cells and low abundance genes that require read count filtration before correlation analysis
- The packaged GTEX_8000 dataset is a sample (~8000 genes, 53 tissues) of a larger dataset containing >42K genes, so results may not reflect the full GTEx data
- Data transformation choice matters: log2 is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity; other data types may need z-score or other transformations

## Evidence

- [abstract] prime feature correlation analysis: "This method or function allows the user to observe the correlation of one feature with many features by providing a prime feature to be observed."
- [abstract] log2 transformation rationale: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal"
- [methods] data_transform call with medianthres: "transdf<- data_transform(df=GTEX[-1],transformation='log2', featurelist=GTEX$Description, medianthres=1)"
- [readme] primefeature_corr invocation in README: "primefcorr <- primefeature_corr(df=inputdf,featurelist=rownames(inputdf) ,primefeature="RNASEH2A")"
- [readme] GTEX_8000 dataset description: "It contains Gene-Tissue Expression dataset with
~8000 genes(features) expression in 53 tissues, sampled from a larger
dataset containing \>42K genes."
