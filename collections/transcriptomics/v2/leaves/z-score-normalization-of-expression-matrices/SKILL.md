---
name: z-score-normalization-of-expression-matrices
description: Use when an expression matrix (e.g. TCGA-style microarray gene-by-sample
  data or single-cell RNA-seq counts after read-count filtration) shows non-normal
  or bimodal feature distributions and features must be put on a comparable, zero-centered
  scale.
when_to_use_negative:
- When the goal is specifically to reduce PCR artifacts and heteroskedasticity in
  sequencing data — log2 transformation is the article's recommended choice there
- When features are already normally distributed and on comparable scales — normalization
  adds no benefit and may distort the data
- When working with raw, unfiltered single-cell counts containing non-viable cells
  and low-abundance genes — read count filtration must precede transformation
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3518
tools:
- name: FeatureCorr
  role: Provides the data_transform function that performs the Z-score transformation
    and outputs before/after distribution plots
  repo: github.com/DKundnani/FeatureCorr
- name: R
  role: Runtime environment in which the FeatureCorr package functions are executed
- name: ggplot2
  role: Used to plot density/histogram panels comparing original vs. Z-score-transformed
    expression distributions
- name: devtools
  role: Installs the FeatureCorr package from GitHub (devtools::install_github)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:ab12c8851d54d2da0ea901264a34e58e3e758ac5e02379db749dfff6e02eed46
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureCorr
  - R
  - devtools
  - ggplot2
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# z-score-normalization-of-expression-matrices

## Summary

Apply Z-score transformation to expression feature tables (e.g. microarray or single-cell RNA-seq matrices) with non-normal or multimodal feature distributions, using FeatureCorr::data_transform in R to rescale each feature to mean 0 and standard deviation 1 and to visualize the original vs. transformed distributions.

## When to use

Use when an expression matrix (e.g. TCGA-style microarray gene-by-sample data or single-cell RNA-seq counts after read-count filtration) shows non-normal or bimodal feature distributions and features must be put on a comparable, zero-centered scale.

## When NOT to use

- When the goal is specifically to reduce PCR artifacts and heteroskedasticity in sequencing data — log2 transformation is the article's recommended choice there
- When features are already normally distributed and on comparable scales — normalization adds no benefit and may distort the data
- When working with raw, unfiltered single-cell counts containing non-viable cells and low-abundance genes — read count filtration must precede transformation

## Inputs

- Expression data frame (genes/features as rows, samples as columns), e.g. TCGA40 (2000 samples x 40 genes from The Cancer Genome Atlas Pan Cancer subset)
- Feature list (rownames of the input data frame)
- Optional medianthres threshold for low-abundance feature filtering

## Outputs

- Z-score-normalized expression data frame (first item of the returned list, transdf[[1]])
- Distribution visualizations of the original and transformed data (density/histogram plots, e.g. zscore_tcga40_distribution.png)

## How to apply

Install FeatureCorr with devtools::install_github("DKundnani/FeatureCorr"), load the feature (rows) by sample (columns) expression data frame, and call FeatureCorr::data_transform with transformation='z-score' and the featurelist taken from rownames(df). For datasets with very low-abundance features, the package supports a medianthres filtering argument (used with medianthres=1 in the README's log2 example) before transformation. The function returns a list whose first item is the transformed data frame (extract with transdf[[1]]) and produces distribution plots of both the original and transformed data so the user can assess whether normalization achieved symmetric, zero-centered distributions. Z-score is chosen over log2 when the goal is to reduce scale differences between features on non-normal/multimodal data rather than to address PCR artifacts and heteroskedasticity typical of sequencing data.

## Related tools

- **FeatureCorr** (Provides the data_transform function that performs the Z-score transformation and outputs before/after distribution plots) — github.com/DKundnani/FeatureCorr
- **R** (Runtime environment in which the FeatureCorr package functions are executed)
- **ggplot2** (Used to plot density/histogram panels comparing original vs. Z-score-transformed expression distributions)
- **devtools** (Installs the FeatureCorr package from GitHub (devtools::install_github))

## Examples

```
transdf <- data_transform(df=TCGA40, transformation='z-score', featurelist=rownames(TCGA40)); inputdf <- transdf[[1]]
```

## Evaluation signals

- The Z-score-transformed values have mean (center) approximately 0 for each feature — the defining property of z-score normalization
- The Z-score-transformed values have standard deviation approximately 1 per feature
- The output distribution PNG (e.g. zscore_tcga40_distribution.png) exists and is a valid image file
- The transformed data frame has the same feature-by-sample dimensions as the input (e.g. 40 genes x 2000 samples for TCGA40)
- Side-by-side density plots show the transformed distribution is more symmetric/zero-centered than the original non-normal or multimodal one

## Limitations

- Single-cell experiments retain limitations from non-viable cells and low-abundance genes that require read count filtration before transformation
- FeatureCorr offers three transformations total (log2, z-score and a third preprocessing option); choosing Z-score when log2 is appropriate (sequencing data with PCR artifacts/heteroskedasticity) is a misuse
- The data_transform function returns a list, so the user must extract transdf[[1]] to obtain the transformed data frame — a common source of errors

## Evidence

- [abstract] Thus, we provide three different transformations for the user to aid in preprocessing and normalization of data.: "Thus, we provide three different transformations for the user to aid in preprocessing and normalization of data."
- [abstract] Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity.: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal or read count."
- [methods] Data Transformation using FeatureCorr::data_transform function: "Data Transformation using FeatureCorr::data_transform function"
- [readme] Another dataset is from The Cancer Genome Atlas Pan Cancer dataset randomly selected 2000 samples from >10K samples: "Another dataset is from The Cancer Genome Atlas Pan Cancer dataset randomly selected 2000 samples from >10K samples, with information for selected 40 features(genes)"
