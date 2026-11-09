---
name: gene-expression-data-preprocessing
description: Use when a raw expression matrix from NGS/microarray experiments (e.g.,
  TCGA, GTEx-style gene-by-sample data frames) shows PCR artifacts, heteroskedasticity
  in high-signal features, or non-normal/multimodal feature distributions before correlation
  analysis.
when_to_use_negative:
- The data are already log2-transformed or normalized (e.g., the packaged logTCGA40
  used directly in pair_scatter/pairwise_corr) — re-transforming would distort downstream
  correlations
- Single-cell data with non-viable cells and low-abundance genes that require read
  count filtration rather than a simple medianthres filter before transformation
- The goal is correlation or network analysis itself (primefeature_corr, pair_scatter,
  pairwise_corr) rather than the preprocessing/normalization step that precedes it
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: FeatureCorr
  role: Provides the data_transform function that performs log2/z-score transformations,
    median-expression filtering (medianthres), and outputs distribution plots of original
    vs transformed data
  repo: github.com/DKundnani/FeatureCorr
- name: R
  role: Runtime environment in which FeatureCorr and its transformation/visualization
    functions are executed
  repo: github.com/DKundnani/FeatureCorr
- name: devtools
  role: Used to install the development version of FeatureCorr from GitHub before
    running data_transform
  repo: github.com/DKundnani/FeatureCorr
- name: ggplot2
  role: Used to plot density/distribution panels of original vs log2- or z-score-transformed
    expression values for visual validation
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:c7b28a2512d9fb21bd263afaefd64a61caf1177a8fd827a35ec17d86bcf90c18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureCorr
  - R
  - devtools
  - dplyr
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gene-expression-data-preprocessing

## Summary

Preprocessing and normalization of sequencing- or microarray-derived gene expression matrices using FeatureCorr's data_transform function, offering log2 and z-score transformations with median-expression filtering and original-vs-transformed distribution visualizations.

## When to use

Use when a raw expression matrix from NGS/microarray experiments (e.g., TCGA, GTEx-style gene-by-sample data frames) shows PCR artifacts, heteroskedasticity in high-signal features, or non-normal/multimodal feature distributions before correlation analysis.

## When NOT to use

- The data are already log2-transformed or normalized (e.g., the packaged logTCGA40 used directly in pair_scatter/pairwise_corr) — re-transforming would distort downstream correlations
- Single-cell data with non-viable cells and low-abundance genes that require read count filtration rather than a simple medianthres filter before transformation
- The goal is correlation or network analysis itself (primefeature_corr, pair_scatter, pairwise_corr) rather than the preprocessing/normalization step that precedes it

## Inputs

- Gene-by-sample expression data frame from NGS or microarray preliminary analysis (e.g., FeatureCorr's TCGA40: 2000 samples x 40 genes, or GTEX: ~8000 genes in 53 tissues)
- Feature list (e.g., rownames(TCGA40) or GTEX$Description gene identifiers)
- Transformation choice ('log2' or 'z-score') and median expression threshold (medianthres, e.g., 1)

## Outputs

- Transformed expression data frame (first element of the list returned by data_transform, e.g., transdf[[1]] or logTCGA40)
- Distribution visualization comparing original vs transformed expression values (density/histogram plot, e.g., saved as PNG)
- Filtered feature set meeting the median expression threshold

## How to apply

Install FeatureCorr with devtools::install_github('DKundnani/FeatureCorr') and load an expression data frame (e.g., the packaged TCGA40, 2000 samples x 40 genes, or GTEX with ~8000 genes in 53 tissues). Choose the transformation per data type: 'log2' is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity (high variation in features with high signal or read count), while 'z-score' targets non-normal or multimodal distributions such as microarray and single-cell RNA-seq data. Call FeatureCorr::data_transform with df, transformation, featurelist (e.g., rownames(df) or GTEX$Description), and medianthres (e.g., 1) to filter low-abundance features by median expression before transformation. Extract the transformed data frame as the first element of the returned list (transdf[[1]]), then plot original and transformed value distributions (density/histogram, e.g., ggplot2) to confirm reduced spread in high-count features or zero-centered unit-variance values.

## Related tools

- **FeatureCorr** (Provides the data_transform function that performs log2/z-score transformations, median-expression filtering (medianthres), and outputs distribution plots of original vs transformed data) — github.com/DKundnani/FeatureCorr
- **R** (Runtime environment in which FeatureCorr and its transformation/visualization functions are executed) — github.com/DKundnani/FeatureCorr
- **devtools** (Used to install the development version of FeatureCorr from GitHub before running data_transform) — github.com/DKundnani/FeatureCorr
- **ggplot2** (Used to plot density/distribution panels of original vs log2- or z-score-transformed expression values for visual validation)

## Examples

```
transdf <- data_transform(df=TCGA40, transformation='log2', featurelist=rownames(TCGA40), medianthres=1)
```

## Evaluation signals

- The transformed object returned by data_transform retains the same feature count (e.g., all 40 genes of TCGA40) as the input feature list
- For log2: density plots of transformed values show reduced spread (heteroskedasticity) in features with high signal or read count relative to the original distribution
- For z-score: transformed values are centered at zero with unit standard deviation
- Features below the medianthres cutoff (e.g., medianthres=1) are excluded prior to transformation, as in the GTEX example
- The distribution visualization (e.g., zscore_tcga40_distribution.png) exists, is a valid image, and shows side-by-side original vs transformed distributions

## Limitations

- Log2 transformation is recommended specifically for sequencing data; non-normal or bimodal microarray/single-cell distributions are instead served by the z-score option
- Single-cell experiments carry additional limitations due to non-viable cells and low abundance genes that require read count filtration beyond a simple median threshold
- data_transform depends on a user-supplied featurelist and medianthres, so incorrect feature identifiers (e.g., using rownames vs a Description column) or thresholds change which features survive filtering

## Evidence

- [abstract] Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal or read count.: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal or read count."
- [abstract] Thus, we provide three different transformations for the user to aid in preprocessing and normalization of data.: "Thus, we provide three different transformations for the user to aid in preprocessing and normalization of data."
- [methods] transdf<- data_transform(df=GTEX[-1],transformation='log2', featurelist=GTEX$Description, medianthres=1): "transdf<- data_transform(df=GTEX[-1],transformation='log2', featurelist=GTEX$Description, medianthres=1)"
- [readme] Another dataset is from The Cancer Genome Atlas Pan Cancer dataset randomly selected 2000 samples from >10K samples, with information for selected 40 features(genes): "Another dataset is from The Cancer Genome Atlas Pan Cancer dataset
randomly selected 2000 samples from \>10K samples, with information for
selected 40 features(genes)"
- [readme] inputdf<-transdf[[1]] #First item in the list returned is the transformed Data: "inputdf<-transdf[[1]] #First item in the list returned is the transformed Data"
- [abstract] there are still limitations to single-cell experiments due to non-viable cells and low abundance genes that require read count filtration: "there are still limitations to single-cell experiments due to non-viable cells and low abundance genes that require read count filtration"
