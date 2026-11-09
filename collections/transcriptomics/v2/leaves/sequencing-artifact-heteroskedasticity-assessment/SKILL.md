---
name: sequencing-artifact-heteroskedasticity-assessment
description: Use when input is raw (untransformed) NGS/RNA-seq expression data (e.g.,
  TCGA, GTEx count-style feature tables) showing high variance in high-read-count
  features, and a preprocessing/normalization choice must be made before correlation
  analysis.
when_to_use_negative:
- Data are already log2-transformed (e.g., the packaged logTCGA40 object) — re-transforming
  would distort the values
- Data derive from microarray experiments where log2 is not the indicated transformation
  among the package's three options (log2, z-score)
- Input is single-cell count data requiring prior read count filtration for non-viable
  cells and low-abundance genes before transformation
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3520
tools:
- name: FeatureCorr
  role: Provides data_transform() for log2 (and z-score) transformation with medianthres
    feature filtering, plus distribution plotting of transformed vs original data
  repo: https://github.com/DKundnani/FeatureCorr
- name: R
  role: Runtime environment in which FeatureCorr is installed via devtools and the
    transformation workflow is executed
- name: devtools
  role: Used to install the FeatureCorr package from GitHub (devtools::install_github)
- name: ggplot2
  role: Underlying plotting engine for the density/distribution visualizations comparing
    original vs log2-transformed expression
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:da09a9e020fe6f76274d6b258ccc9a76029f31abb0b5605476abfdaa617cef0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DKundnani/FeatureCorr
  tools:
  - FeatureCorr
  - R
  - devtools
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# sequencing-artifact-heteroskedasticity-assessment

## Summary

Applies log2 transformation (via FeatureCorr::data_transform) to sequencing-derived expression data to reduce PCR artifacts and heteroskedasticity — high variation in features with high signal or read count — and verifies the effect with before/after distribution plots.

## When to use

Use when input is raw (untransformed) NGS/RNA-seq expression data (e.g., TCGA, GTEx count-style feature tables) showing high variance in high-read-count features, and a preprocessing/normalization choice must be made before correlation analysis.

## When NOT to use

- Data are already log2-transformed (e.g., the packaged logTCGA40 object) — re-transforming would distort the values
- Data derive from microarray experiments where log2 is not the indicated transformation among the package's three options (log2, z-score)
- Input is single-cell count data requiring prior read count filtration for non-viable cells and low-abundance genes before transformation

## Inputs

- R data frame of raw sequencing-derived expression values (e.g., FeatureCorr's packaged TCGA40: TCGA Pan Cancer, 2000 samples, 40 genes; or GTEX_8000: ~8000 genes across 53 tissues)
- Feature list (e.g., rownames(TCGA40) or GTEX$Description gene identifiers)
- Median expression threshold (medianthres, e.g., 1) for pre-transformation feature filtering

## Outputs

- List whose first element [[1]] is the log2-transformed data frame with the same feature identifiers
- Density/distribution plot of expression values before and after log2 transformation (exportable as PNG)

## How to apply

Load the FeatureCorr R package and call data_transform with transformation='log2', passing the expression data frame (e.g., TCGA40 or GTEX[-1]), a featurelist (e.g., rownames(TCGA40) or GTEX$Description), and a medianthres filter (medianthres=1) so low-abundance features are removed before transformation. Extract the log2-transformed data frame as the first element ([[1]]) of the list returned by data_transform. Generate density/distribution plots of expression values before and after log2 transformation to inspect the reduction of spread in high-signal/high-count features. The rationale, per the article, is that log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal or read count; retain the transformed object (same feature set) for downstream primefeature_corr, pair_scatter, or pairwise_corr analyses.

## Related tools

- **FeatureCorr** (Provides data_transform() for log2 (and z-score) transformation with medianthres feature filtering, plus distribution plotting of transformed vs original data) — https://github.com/DKundnani/FeatureCorr
- **R** (Runtime environment in which FeatureCorr is installed via devtools and the transformation workflow is executed)
- **devtools** (Used to install the FeatureCorr package from GitHub (devtools::install_github))
- **ggplot2** (Underlying plotting engine for the density/distribution visualizations comparing original vs log2-transformed expression)

## Examples

```
library(FeatureCorr); transdf <- data_transform(df=TCGA40, transformation='log2', featurelist=rownames(TCGA40), medianthres=1); inputdf <- transdf[[1]]
```

## Evaluation signals

- The returned object is a list whose first element is a data frame retaining the same 40 gene features (for TCGA40 input)
- Before/after density plots show reduced spread (heteroskedasticity) in high-count/high-signal features after log2 transformation
- The medianthres=1 filter has removed features failing the median expression cutoff prior to transformation
- Downstream functions (primefeature_corr, pair_scatter, pairwise_corr) accept the transformed data frame without identifier mismatch

## Limitations

- Single-cell experiments face limitations due to non-viable cells and low abundance genes that require read count filtration before this transformation
- FeatureCorr provides three transformations (log2, z-score); log2 must be chosen deliberately — it is the recommended option specifically for sequencing data, not universally
- Features below the median expression threshold (medianthres) are discarded, so the transformed matrix may have fewer features than the input

## Evidence

- [abstract] Log2 transformation reduces PCR artifacts and heteroskedasticity: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal or read count."
- [methods] data_transform log2 invocation: "transdf<- data_transform(df=GTEX[-1],transformation='log2', featurelist=GTEX$Description, medianthres=1)"
- [readme] Transformed data frame is first list element: "inputdf<-transdf[[1]] #First item in the list returned is the transformed Data"
- [abstract] Three transformations offered for preprocessing: "Thus, we provide three different transformations for the user to aid in preprocessing and normalization of data."
