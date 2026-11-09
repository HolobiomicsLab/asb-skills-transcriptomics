---
name: gene-pairwise-correlation-analysis
description: Use when you have a log2-transformed expression matrix (data frame) of
  multiple genes/features (e.g. 40 genes across ~10K TCGA Pan-Cancer samples) and
  need a pairwise correlation heatmap ordered by hierarchical clustering into feature
  clusters.
when_to_use_negative:
- You want correlation of ONE prime feature (e.g. RNASEH2A) against many features
  — use primefeature_corr instead of pairwise_corr.
- You need deep analysis of a single feature pair with regression/density — use pair_scatter
  (e.g. feature1="RNASEH2A", feature2="PCNA").
- Your input data is untransformed/raw sequencing counts and exhibits heteroskedasticity
  — first apply data_transform with transformation='log2' and a medianthres filter
  (e.g. medianthres=1) before pairwise_corr.
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0091
tools:
- name: FeatureCorr
  role: R package providing pairwise_corr for multiple pairwise correlation with clustering
    and correlation heatmap visualization; installed via devtools::install_github("DKundnani/FeatureCorr")
  repo: https://github.com/DKundnani/FeatureCorr
- name: R
  role: Runtime environment in which the FeatureCorr package and its functions (pairwise_corr,
    data_transform) are executed
- name: devtools
  role: Used to install the development version of FeatureCorr from GitHub
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:ed186cf8b2a8c8f78ac69268d6cfc218971a8db7ddf275fc072668f536b9c1ce
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
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# gene-pairwise-correlation-analysis

## Summary

Compute and visualize multiple pairwise correlations among a set of genes/features using the FeatureCorr R package's pairwise_corr function, whose default output is a hierarchically clustered correlation heatmap. It is designed for NGS or microarray expression data such as the TCGA Pan-Cancer logTCGA40 dataset (40 genes, >10K samples).

## When to use

Use when you have a log2-transformed expression matrix (data frame) of multiple genes/features (e.g. 40 genes across ~10K TCGA Pan-Cancer samples) and need a pairwise correlation heatmap ordered by hierarchical clustering into feature clusters.

## When NOT to use

- You want correlation of ONE prime feature (e.g. RNASEH2A) against many features — use primefeature_corr instead of pairwise_corr.
- You need deep analysis of a single feature pair with regression/density — use pair_scatter (e.g. feature1="RNASEH2A", feature2="PCNA").
- Your input data is untransformed/raw sequencing counts and exhibits heteroskedasticity — first apply data_transform with transformation='log2' and a medianthres filter (e.g. medianthres=1) before pairwise_corr.

## Inputs

- log2-transformed gene expression data frame (e.g. logTCGA40: log2 TCGA Pan-Cancer expression, 40 genes/features)
- feature list of gene identifiers (rownames of the data frame)
- optional boolean featuregroup vector (e.g. grepl("RNASE", rownames(logTCGA40)))

## Outputs

- clustered pairwise correlation heatmap (default visorder="hclust" output of pairwise_corr)
- pairwise correlation results object (e.g. corr_pair) returned by pairwise_corr
- saved plot image (e.g. pairwise_corr_heatmap.png)

## How to apply

Install FeatureCorr via devtools::install_github("DKundnani/FeatureCorr") and attach the packaged logTCGA40 dataset (log2-transformed TCGA Pan-Cancer expression for 40 genes). Call pairwise_corr with df=logTCGA40, featurelist=rownames(logTCGA40), visorder="hclust", and clustno=2 so features are ordered by hierarchical clustering split into 2 clusters, mirroring the README example for '40 genes in ~10K samples in The Cancer Genome Atlas-Pan Cancer dataset'. Optionally compute a boolean feature group with grepl("RNASE", rownames(logTCGA40)) and pass it via featuregroup to visually separate RNASE-family features. The rationale for the log2-transformed input is that log2 transformation reduces PCR artifacts and heteroskedasticity in sequencing data. Render the default clustered correlation heatmap output and save it (e.g. as pairwise_corr_heatmap.png).

## Related tools

- **FeatureCorr** (R package providing pairwise_corr for multiple pairwise correlation with clustering and correlation heatmap visualization; installed via devtools::install_github("DKundnani/FeatureCorr")) — https://github.com/DKundnani/FeatureCorr
- **R** (Runtime environment in which the FeatureCorr package and its functions (pairwise_corr, data_transform) are executed)
- **devtools** (Used to install the development version of FeatureCorr from GitHub)

## Examples

```
corr_pair <- pairwise_corr(df=logTCGA40, featurelist=rownames(logTCGA40), visorder="hclust", clustno=2)
```

## Evaluation signals

- The saved heatmap file (e.g. pairwise_corr_heatmap.png) exists and is a readable PNG.
- The rendered plot is a correlation heatmap of the 40 logTCGA40 genes with hierarchical-clustering (hclust) ordering.
- The heatmap displays 2 feature clusters when clustno=2 is passed.
- The call matches the README signature: pairwise_corr(df=logTCGA40, featurelist=rownames(logTCGA40), visorder="hclust", clustno=2).
- If featuregroup is used, RNASE-family features are visually separated from other features in the heatmap.

## Limitations

- Single-cell experiments face limitations due to non-viable cells and low abundance genes that require read count filtration.
- Untransformed sequencing data can exhibit PCR artifacts and heteroskedasticity (high variation in features with high signal or read count), so log2 transformation is typically required before correlation.
- The packaged TCGA40/logTCGA40 dataset is a random subsample (2000 samples from >10K samples), not the full 10,071-sample TCGA Pan-Cancer set.

## Evidence

- [abstract] While observing the pairwise correlation, the default output provided users a clustering method to visualize the correlation heatmap: "While observing the pairwise correlation, the default output provided users a clustering method to visualize the correlation heatmap"
- [abstract] multiple pairwise correlations were tested on TCGA dataset containing 10,071 samples for 40 genes: "multiple pairwise correlations were tested on TCGA dataset containing 10,071 samples for 40 genes"
- [abstract] Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal or read count."
- [readme] corr_pair<-pairwise_corr(df=logTCGA40,featurelist=rownames(logTCGA40), visorder="hclust", clustno=2): "corr_pair<-pairwise_corr(df=logTCGA40,featurelist=rownames(logTCGA40), visorder="hclust", clustno=2)"
- [readme] featgroup<-grepl( "RNASE",rownames(logTCGA40)) #optional, a set of features to separated: "featgroup<-grepl( "RNASE",rownames(logTCGA40)) #optional, a set of features to separated"
