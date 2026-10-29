---
name: cancer-gene-coexpression-analysis
description: Use when a panel of cancer genes (e.g., 40 genes from >10K TCGA pan-cancer
  samples) must be screened for co-expression and visualized as a clustered pairwise
  correlation heatmap.
when_to_use_negative:
- The question is how one prime gene (e.g., RNASEH2A) correlates with many others
  — use FeatureCorr::primefeature_corr (prime feature correlation) instead of pairwise_corr.
- Input is single-cell data with non-viable cells and low abundance genes that require
  read count filtration — the skill targets bulk sequencing/microarray feature tables.
- Input is raw untransformed expression with high heteroskedasticity and no log2 preprocessing
  (data_transform) is planned — correlations on untransformed sequencing counts can
  mislead.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2648
tools:
- name: FeatureCorr
  role: R package supplying pairwise_corr for multiple pair-wise correlation with
    clustered heatmap visualization; its packaged logTCGA40/TCGA40 datasets provide
    the TCGA pan-cancer input
  repo: https://github.com/DKundnani/FeatureCorr
- name: devtools
  role: Used to install FeatureCorr from GitHub via devtools::install_github("DKundnani/FeatureCorr")
- name: R
  role: Runtime environment in which FeatureCorr functions and the packaged TCGA40/logTCGA40
    expression data are loaded and executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:e2ad0a3cf30658e8823f38c75c9e99f6258de0d13b5d5e4692f94b88a2465fa4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DKundnani/FeatureCorr
  tools:
  - FeatureCorr
  - Hmisc
  - R
  - corrplot
  - devtools
  - ggplot2
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2648
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cancer-gene-coexpression-analysis

## Summary

FeatureCorr-driven pairwise co-expression analysis of cancer gene panels: pairwise_corr computes gene-gene correlations on log2-transformed TCGA Pan-Cancer expression (logTCGA40, 40 genes from >10K samples) and renders a hierarchically clustered correlation heatmap as its default output.

## When to use

Use when a panel of cancer genes (e.g., 40 genes from >10K TCGA pan-cancer samples) must be screened for co-expression and visualized as a clustered pairwise correlation heatmap.

## When NOT to use

- The question is how one prime gene (e.g., RNASEH2A) correlates with many others — use FeatureCorr::primefeature_corr (prime feature correlation) instead of pairwise_corr.
- Input is single-cell data with non-viable cells and low abundance genes that require read count filtration — the skill targets bulk sequencing/microarray feature tables.
- Input is raw untransformed expression with high heteroskedasticity and no log2 preprocessing (data_transform) is planned — correlations on untransformed sequencing counts can mislead.

## Inputs

- Gene expression data frame (genes x samples) from a cancer cohort, e.g., FeatureCorr::logTCGA40 — log2-transformed TCGA Pan-Cancer expression for 40 genes
- Feature list of gene identifiers (e.g., rownames(logTCGA40))
- Optional logical feature-group vector (e.g., featgroup<-grepl("RNASE",rownames(logTCGA40)))

## Outputs

- Pairwise correlation coefficients among the gene panel (multiple pair-wise correlation of the 40 features)
- Clustered correlation heatmap plot (hclust ordering, clustno=2), saveable as e.g. pairwise_corr_heatmap.png
- Optionally a second heatmap variant separating the featuregroup of interest

## How to apply

Install the package with devtools::install_github("DKundnani/FeatureCorr"), then attach the packaged logTCGA40 data — log2-transformed TCGA Pan-Cancer expression for 40 genes; if starting from raw counts, run data_transform(df, transformation='log2', medianthres=1) first, since log2 transformation reduces PCR artifacts and heteroskedasticity in sequencing data. Call pairwise_corr(df=logTCGA40, featurelist=rownames(logTCGA40), visorder="hclust", clustno=2): the function's default output provides a clustering method to visualize the correlation heatmap, here with features ordered by hierarchical clustering split into 2 clusters. Optionally build a logical feature group with featgroup<-grepl("RNASE",rownames(logTCGA40)) and pass it via the featuregroup argument to separate RNASE-family features in the visualization. Save the rendered heatmap (e.g., pairwise_corr_heatmap.png) and check it is a readable PNG showing a 40-gene correlation matrix with hclust ordering and 2 feature clusters. Note the study cohort spanned 10,071 TCGA samples for 40 genes, while the packaged subset is 2000 randomly selected samples from >10K.

## Related tools

- **FeatureCorr** (R package supplying pairwise_corr for multiple pair-wise correlation with clustered heatmap visualization; its packaged logTCGA40/TCGA40 datasets provide the TCGA pan-cancer input) — https://github.com/DKundnani/FeatureCorr
- **devtools** (Used to install FeatureCorr from GitHub via devtools::install_github("DKundnani/FeatureCorr"))
- **R** (Runtime environment in which FeatureCorr functions and the packaged TCGA40/logTCGA40 expression data are loaded and executed)

## Examples

```
corr_pair <- pairwise_corr(df=logTCGA40, featurelist=rownames(logTCGA40), visorder="hclust", clustno=2)
```

## Evaluation signals

- pairwise_corr on logTCGA40 with visorder="hclust" and clustno=2 renders a heatmap whose row/column order follows hierarchical clustering split into 2 feature clusters (the function's documented default output).
- The saved figure (pairwise_corr_heatmap.png) exists, is a readable PNG, and displays a correlation heatmap of the 40 genes.
- Feature identifiers match logTCGA40/TCGA40 scope: 40 selected genes; packaged data are 2000 randomly selected samples from >10K TCGA pan-cancer samples (10,071 in the full study cohort).
- Passing featuregroup=grepl("RNASE", rownames(logTCGA40)) visually separates RNASE-family features without altering the underlying correlation calculation.
- If computed from raw expression, data_transform(..., transformation='log2', medianthres=1) runs first and the transformed data frame (first list item returned) is passed to pairwise_corr.

## Limitations

- The packaged TCGA40/logTCGA40 subsets are 2000 randomly selected samples from >10K — reproduction is against the sampled subset, not the full 10,071-sample cohort.
- Single-cell experiments have limitations due to non-viable cells and low abundance genes that require read count filtration; this skill is aimed at bulk sequencing/microarray feature tables.
- Sequencing data should be log2-transformed first (per data_transform) to reduce PCR artifacts and heteroskedasticity; the packaged logTCGA40 is already log2-transformed.

## Evidence

- [readme] corr_pair<-pairwise_corr(df=logTCGA40,featurelist=rownames(logTCGA40), visorder="hclust", clustno=2): "corr_pair<-pairwise_corr(df=logTCGA40,featurelist=rownames(logTCGA40), visorder="hclust", clustno=2)"
- [readme] Multiple pairwise correlation for 40 genes in ~10K samples in The Cancer Genome Atlas-Pan Cancer dataset: "Multiple pairwise correlation for 40 genes in ~10K samples in The Cancer Genome Atlas-Pan Cancer dataset"
- [readme] Another dataset is from The Cancer Genome Atlas Pan Cancer dataset randomly selected 2000 samples from >10K samples: "Another dataset is from The Cancer Genome Atlas Pan Cancer dataset randomly selected 2000 samples from >10K samples"
- [abstract] multiple pairwise correlations were tested on TCGA dataset containing 10,071 samples for 40 genes: "multiple pairwise correlations were tested on TCGA dataset containing 10,071 samples for 40 genes"
- [abstract] While observing the pairwise correlation, the default output provided users a clustering method to visualize the correlation heatmap: "While observing the pairwise correlation, the default output provided users a clustering method to visualize the correlation heatmap"
- [abstract] Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity"
- [methods] featgroup<-grepl( "RNASE",rownames(logTCGA40)) #optional, a set of features to separated: "featgroup<-grepl( "RNASE",rownames(logTCGA40)) #optional, a set of features to separated"
