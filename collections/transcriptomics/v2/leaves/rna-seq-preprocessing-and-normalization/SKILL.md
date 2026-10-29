---
name: rna-seq-preprocessing-and-normalization
description: Use when expression data (microarray, bulk or single-cell RNA-seq, e.g.
  TCGA-like sample-by-gene matrices) has a non-normal, bimodal, or multimodal feature
  distribution and must be Z-score normalized before correlation analysis.
when_to_use_negative:
- Sequencing count data dominated by PCR artifacts or heteroskedasticity (high variation
  in features with high signal or read count) — the article states log2 transformation
  is most frequently used there instead of z-score.
- Single-cell data with non-viable cells and low abundance genes that first require
  read count filtration — the article notes this limitation precedes transformation.
- Data already normalized/centered, where a further z-score pass would be redundant
  before correlation analysis.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3518
tools:
- name: FeatureCorr
  role: Provides the data_transform function used to apply the z-score transformation
    and to output distribution plots of the original and transformed data.
  repo: https://github.com/DKundnani/FeatureCorr
- name: R
  role: Runtime language in which FeatureCorr is installed via devtools and the transformation
    workflow is executed.
  repo: https://github.com/DKundnani/FeatureCorr
- name: ggplot2
  role: Used to plot expression-value distributions of the original vs. z-score-transformed
    data as density/histogram panels for comparison.
- name: devtools
  role: Installs the development version of FeatureCorr from GitHub before running
    data_transform.
  repo: https://github.com/DKundnani/FeatureCorr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:3205fda14a1d4ff52b9e60c529ae151990f3885f3290182ff83e6f2cc35b8233
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DKundnani/FeatureCorr
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
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# rna-seq-preprocessing-and-normalization

## Summary

Z-score transformation of non-normal or multimodal expression data (e.g., microarray and single-cell RNA-seq) using the FeatureCorr::data_transform function in R, with distribution visualizations of both original and transformed data to assess the transformation.

## When to use

Use when expression data (microarray, bulk or single-cell RNA-seq, e.g. TCGA-like sample-by-gene matrices) has a non-normal, bimodal, or multimodal feature distribution and must be Z-score normalized before correlation analysis.

## When NOT to use

- Sequencing count data dominated by PCR artifacts or heteroskedasticity (high variation in features with high signal or read count) — the article states log2 transformation is most frequently used there instead of z-score.
- Single-cell data with non-viable cells and low abundance genes that first require read count filtration — the article notes this limitation precedes transformation.
- Data already normalized/centered, where a further z-score pass would be redundant before correlation analysis.

## Inputs

- Expression data frame (features x samples or samples x features), e.g. the bundled TCGA40 dataset: 2000 TCGA Pan Cancer samples, 40 genes
- Feature list, typically rownames(df) of the expression data frame
- Choice of transformation, 'z-score' (among three transformations offered: log2, z-score, and a third preprocessing/normalization option)

## Outputs

- Z-score-normalized expression data frame (returned as the first element of the data_transform result list)
- Distribution visualization of the original and transformed data (density/histogram panels, e.g. zscore_tcga40_distribution.png)

## How to apply

Install the development version with devtools::install_github("DKundnani/FeatureCorr") and load the package in R. Supply an expression data frame such as the bundled TCGA40 dataset (2000 samples selected from >10K Pan Cancer samples, 40 genes) with the transformation argument set to 'z-score' and the featurelist taken from rownames(df); this contrasts with the log2 option that is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity. The data_transform function returns a list whose first element is the transformed data frame (as shown for the log2 example where inputdf<-transdf[[1]]), and it outputs distribution visualizations of both the original and transformed data so users can assess whether the transformation removed the skew or multimodality. Optionally, render the original vs. transformed expression-value distributions as density/histogram panels with ggplot2 and save them (e.g. zscore_tcga40_distribution.png) for side-by-side comparison. Validate that the Z-score-transformed values are centered at zero with unit standard deviation, the defining property of z-score normalization.

## Related tools

- **FeatureCorr** (Provides the data_transform function used to apply the z-score transformation and to output distribution plots of the original and transformed data.) — https://github.com/DKundnani/FeatureCorr
- **R** (Runtime language in which FeatureCorr is installed via devtools and the transformation workflow is executed.) — https://github.com/DKundnani/FeatureCorr
- **ggplot2** (Used to plot expression-value distributions of the original vs. z-score-transformed data as density/histogram panels for comparison.)
- **devtools** (Installs the development version of FeatureCorr from GitHub before running data_transform.) — https://github.com/DKundnani/FeatureCorr

## Examples

```
transdf <- data_transform(df=TCGA40, transformation='z-score', featurelist=rownames(TCGA40)); zscore_df <- transdf[[1]]
```

## Evaluation signals

- The Z-score-transformed values are centered at zero with unit standard deviation (the defining property of z-score normalization).
- The saved visualization (e.g. zscore_tcga40_distribution.png) exists and is a valid PNG image.
- Density/histogram panels of original vs. transformed data show the multimodality/skew of the original TCGA40 distribution removed after transformation.
- data_transform returns a list whose first element (transdf[[1]]) is the transformed data frame, consistent with the README usage pattern inputdf<-transdf[[1]].

## Limitations

- Single-cell experiments are limited by non-viable cells and low abundance genes that require read count filtration before transformation.
- Log2 transformation is the more frequent choice for sequencing data because it reduces PCR artifacts and heteroskedasticity; z-score is indicated for non-normal or multimodal distributions rather than raw count skew.
- The demo dataset TCGA40 is a randomly selected subset (2000 of >10K samples, 40 genes), so transformed-distribution conclusions are specific to this scale of input.

## Evidence

- [abstract] three transformations offered: "Thus, we provide three different transformations for the user to aid in preprocessing and normalization of data."
- [abstract] log2 for sequencing artifacts: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal or read count."
- [methods] data_transform function: "Data Transformation using FeatureCorr::data_transform function"
- [abstract] single-cell filtration limitation: "there are still limitations to single-cell experiments due to non-viable cells and low abundance genes that require read count filtration"
- [readme] package scope: "An R package to study feature correlations aided with data transformation for Next Generation sequencing and microarray data"
- [readme] TCGA40 dataset description: "Another dataset is from The Cancer Genome Atlas Pan Cancer dataset randomly selected 2000 samples from >10K samples, with information for selected 40 features(genes)"
