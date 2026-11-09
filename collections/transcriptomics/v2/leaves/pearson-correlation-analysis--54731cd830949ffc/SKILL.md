---
name: pearson-correlation-analysis
description: Use when two features from a sequencing or microarray expression data
  frame (e.g., RNASEH2A vs PCNA in logTCGA40) need in-depth pairwise correlation with
  regression fit, correlation coefficient, and density/histogram visualization.
when_to_use_negative:
- The goal is correlating one prime feature against many features rather than a single
  pair — use primefeature_corr instead
- The goal is a multiple pairwise correlation matrix with clustering/heatmap — use
  pairwise_corr (e.g., visorder='hclust', clustno=2) instead
- Input data are raw read counts with PCR artifacts and heteroskedasticity and have
  not yet been transformed — apply data_transform (log2, z-score) first
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3520
tools:
- name: FeatureCorr
  role: R package providing the pair_scatter function for in-depth pairwise correlation
    of two features; installed via devtools::install_github('DKundnani/FeatureCorr')
  repo: https://github.com/DKundnani/FeatureCorr
- name: R
  role: Runtime environment in which the FeatureCorr package and its bundled datasets
    are executed
- name: ggplot2
  role: Rendering backend for the scatterplot and geom_smooth linear regression line
    produced by pair_scatter
- name: ggExtra
  role: Imported by FeatureCorr to add marginal histograms around the pairwise scatterplot
- name: devtools
  role: Used to install the FeatureCorr development version from GitHub
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:7fcf321d2b85cc8ac24080cfe39164a40e68ac43b7c25f5091476c5bdfdacf28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DKundnani/FeatureCorr
  tools:
  - FeatureCorr
  - R
  - devtools
  - ggExtra
  - ggplot2
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# pearson-correlation-analysis

## Summary

In-depth pairwise correlation analysis of two features (e.g., genes) using the FeatureCorr::pair_scatter function, which fits a linear regression and computes a correlation coefficient (e.g., Pearson) while visualizing the scatter with marginal histograms and smooth density.

## When to use

Use when two features from a sequencing or microarray expression data frame (e.g., RNASEH2A vs PCNA in logTCGA40) need in-depth pairwise correlation with regression fit, correlation coefficient, and density/histogram visualization.

## When NOT to use

- The goal is correlating one prime feature against many features rather than a single pair — use primefeature_corr instead
- The goal is a multiple pairwise correlation matrix with clustering/heatmap — use pairwise_corr (e.g., visorder='hclust', clustno=2) instead
- Input data are raw read counts with PCR artifacts and heteroskedasticity and have not yet been transformed — apply data_transform (log2, z-score) first

## Inputs

- R data frame of feature expression values (e.g., FeatureCorr's bundled logTCGA40 or TCGA40 datasets)
- Feature list identifiers (e.g., rownames(logTCGA40))
- Two feature identifiers for the pair (e.g., feature1='RNASEH2A', feature2='PCNA')
- Correlation method string (e.g., corrmeth='pearson')

## Outputs

- A list whose first element ([[1]]) is a ggplot2-based pairwise scatterplot with marginal histograms and smooth density visualization
- A linear regression fit (geom_smooth, formula y ~ x) overlaid on the scatter
- A correlation coefficient value (Pearson) annotated on the plot

## How to apply

Load the FeatureCorr R package and supply a data frame of feature expression (e.g., the bundled logTCGA40 TCGA Pan Cancer subset of 2000 samples with 40 genes, log2-transformed) plus a feature list and the two feature identifiers of interest. Call pair_scatter with corrmeth='pearson' to compute the Pearson correlation coefficient and a geom_smooth linear regression (y ~ x) fit for the pair. The function returns a list whose first element [[1]] is a ggplot2-based scatterplot enhanced with marginal histograms (via ggExtra) and smooth density visualization to observe the distribution of both features. Extract the first list element and save the rendered plot as a PNG for validation against the README reference figures.

## Related tools

- **FeatureCorr** (R package providing the pair_scatter function for in-depth pairwise correlation of two features; installed via devtools::install_github('DKundnani/FeatureCorr')) — https://github.com/DKundnani/FeatureCorr
- **R** (Runtime environment in which the FeatureCorr package and its bundled datasets are executed)
- **ggplot2** (Rendering backend for the scatterplot and geom_smooth linear regression line produced by pair_scatter)
- **ggExtra** (Imported by FeatureCorr to add marginal histograms around the pairwise scatterplot)
- **devtools** (Used to install the FeatureCorr development version from GitHub)

## Examples

```
pair_scatter(df=logTCGA40, featurelist=rownames(logTCGA40), feature1="RNASEH2A", feature2="PCNA", corrmeth='pearson')[[1]]
```

## Evaluation signals

- The plot file exists and is a valid PNG after saving the [[1]] element of the pair_scatter output
- The rendered figure visually matches the README reference images (README-checking-pairwise-scatter-3/4.png), including the Pearson correlation annotation and regression line
- Console output shows the geom_smooth message '`geom_smooth()` using formula 'y ~ x'' indicating the linear regression fit was applied
- The plotted axes correspond to the two requested features (RNASEH2A and PCNA) from the supplied featurelist (rownames(logTCGA40))

## Limitations

- geom_smooth regression uses formula 'y ~ x'; the fit is a simple linear regression, not a nonlinear model
- pair_scatter performs deep analysis of a single pair only — for multi-feature pairwise correlation use pairwise_corr
- The README examples use package-bundled datasets (TCGA40, logTCGA40); results depend on whether raw (TCGA40) or log2-transformed (logTCGA40) data are supplied

## Evidence

- [methods] pair_scatter function takes a data frame and identifiers of two features and returns a linear regression fit, correlation coefficient value, and histograms of both features: "FeatureCorr also has a pairwise scatter function for deep analysis of each pair of features."
- [readme] pair_scatter call on logTCGA40 with feature1 RNASEH2A and feature2 PCNA: "pair_scatter(df=logTCGA40,featurelist=rownames(logTCGA40),feature1="RNASEH2A", feature2="PCNA", corrmeth='pearson')[[1]]"
- [methods] pair_scatter dataset description from methods: "Another dataset is from The Cancer Genome Atlas Pan Cancer dataset randomly selected 2000 samples from >10K samples"
- [readme] geom_smooth message from README example output: "#> `geom_smooth()` using formula 'y ~ x'"
- [abstract] log2 transformation rationale for sequencing data: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal or read count."
