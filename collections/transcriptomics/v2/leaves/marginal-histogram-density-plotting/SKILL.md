---
name: marginal-histogram-density-plotting
description: Use when a deep dive into the relationship between exactly two features
  (e.g., RNASEH2A vs PCNA) is needed, requiring regression fit, Pearson correlation,
  and marginal density visualization on log2-transformed data.
when_to_use_negative:
- The goal is one-feature-vs-many correlation (use primefeature_corr) or multiple
  pairwise correlation with clustering heatmap (use pairwise_corr)
- The data frame is raw count data with high heteroskedasticity and no log2/z-score
  transformation has been applied — pair_scatter on untransformed data can be misleading
- Only a single feature is of interest rather than a pair — pair_scatter requires
  two features
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3518
tools:
- name: FeatureCorr
  role: Provides the pair_scatter function for single-pair correlation scatter with
    regression fit and histograms
  repo: https://github.com/DKundnani/FeatureCorr
- name: ggplot2
  role: Rendering engine for the scatterplot and geom_smooth linear regression line
- name: ggExtra
  role: Imported dependency supporting marginal histogram/density visualization on
    the pair scatter plot
- name: R
  role: Runtime environment for executing pair_scatter and rendering figures
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:3384176905edec911da4de9e55ca9672353313d437164e29ddfbf28c5f8bac2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DKundnani/FeatureCorr
  tools:
  - FeatureCorr
  - R
  - ggExtra
  - ggplot2
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# marginal-histogram-density-plotting

## Summary

In-depth pairwise scatter analysis of two features using FeatureCorr::pair_scatter, which returns a linear regression fit, correlation coefficient, histograms of both features, and a smooth density scatter for distribution visualization.

## When to use

Use when a deep dive into the relationship between exactly two features (e.g., RNASEH2A vs PCNA) is needed, requiring regression fit, Pearson correlation, and marginal density visualization on log2-transformed data.

## When NOT to use

- The goal is one-feature-vs-many correlation (use primefeature_corr) or multiple pairwise correlation with clustering heatmap (use pairwise_corr)
- The data frame is raw count data with high heteroskedasticity and no log2/z-score transformation has been applied — pair_scatter on untransformed data can be misleading
- Only a single feature is of interest rather than a pair — pair_scatter requires two features

## Inputs

- R data frame of feature expression values (e.g., logTCGA40 or TCGA40, 2000 samples x 40 genes)
- Feature list identifiers (e.g., rownames(logTCGA40))
- Two target feature identifiers (e.g., feature1="RNASEH2A", feature2="PCNA")
- Correlation method string (e.g., corrmeth='pearson')

## Outputs

- A list whose first element ([[1]]) is a ggplot2 scatterplot of feature1 vs feature2 with linear regression fit (geom_smooth)
- Marginal histograms of both features attached to the scatterplot
- Pearson correlation coefficient annotation on the plot
- Smooth scatter density visualization of feature distribution
- Console message from geom_smooth: '`geom_smooth()` using formula 'y ~ x'

## How to apply

Load the FeatureCorr R package and its bundled logTCGA40 dataset (TCGA Pan Cancer subset of 2000 samples with 40 genes, log2-transformed). Call pair_scatter(df=logTCGA40, featurelist=rownames(logTCGA40), feature1="RNASEH2A", feature2="PCNA", corrmeth='pearson') to compute the Pearson correlation coefficient and linear regression (geom_smooth) fit for the pair. Extract the first element of the returned list ([[1]]), which contains a ggplot2-based scatterplot enhanced with marginal histograms and smooth density visualization (via ggExtra). Optionally, repeat on the untransformed TCGA40 data to compare raw vs log2-transformed relationship shapes, since log2 transformation reduces PCR artifacts and heteroskedasticity. Render the ggplot object to a PNG and visually compare against the package README reference images.

## Related tools

- **FeatureCorr** (Provides the pair_scatter function for single-pair correlation scatter with regression fit and histograms) — https://github.com/DKundnani/FeatureCorr
- **ggplot2** (Rendering engine for the scatterplot and geom_smooth linear regression line)
- **ggExtra** (Imported dependency supporting marginal histogram/density visualization on the pair scatter plot)
- **R** (Runtime environment for executing pair_scatter and rendering figures)

## Examples

```
pair_scatter(df=logTCGA40,featurelist=rownames(logTCGA40),feature1="RNASEH2A", feature2="PCNA", corrmeth='pearson')[[1]]
```

## Evaluation signals

- The returned object is a list; element [[1]] is a ggplot2 object that renders without error
- The rendered figure includes a Pearson correlation coefficient annotation and a geom_smooth regression line
- Console output shows the expected geom_smooth messages ('`geom_smooth()` using formula 'y ~ x')
- The saved PNG exists, is valid, and visually matches README-checking-pairwise-scatter-3/4.png from the package README
- Running the same call on raw TCGA40 vs log2-transformed logTCGA40 yields visibly different scatter/regression patterns, confirming transformation was applied

## Limitations

- pair_scatter only analyzes one pair of features at a time; for genome-wide or 40-gene pairwise structure, pairwise_corr with hclust ordering is needed
- Results are sensitive to data transformation: the article notes log2 transformation is used to reduce PCR artifacts and heteroskedasticity, so untransformed inputs may distort correlation and regression visuals
- geom_smooth uses formula 'y ~ x' by default; the regression is a simple linear fit and may not capture non-linear relationships

## Evidence

- [abstract] pair_scatter outputs: "FeatureCorr also has a pairwise scatter function for deep analysis of each pair of features."
- [methods] function call: "pair_scatter(df=logTCGA40,featurelist=rownames(logTCGA40)"
- [readme] readme invocation: "pair_scatter(df=logTCGA40,featurelist=rownames(logTCGA40),feature1="RNASEH2A", feature2="PCNA", corrmeth='pearson')[[1]]"
- [readme] readme console: "`geom_smooth()` using formula 'y ~ x'"
- [abstract] log2 rationale: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal or read count."
