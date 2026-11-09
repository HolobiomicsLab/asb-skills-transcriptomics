---
name: median-threshold-feature-filtering
description: Use when a sequencing (NGS/RNA-seq) or microarray expression data frame
  contains low-abundance or background-noise features that would distort correlation
  analysis, and you need to filter them by median expression (e.g. medianthres=1)
  prior to transformation.
when_to_use_negative:
- Single-cell data where the article notes non-viable cells and low-abundance genes
  require read count filtration as a separate step rather than a simple median threshold
- Data already transformed and normalized — re-filtering a transformed table would
  discard legitimately transformed low-scale values
- Features are methylation or transcription data mixed in one data frame; the article
  suggests grouping with featuregroup (e.g. grepl) instead of threshold filtering
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: FeatureCorr
  role: Provides the data_transform function whose medianthres argument performs the
    median-expression feature filtering before log2 or z-score transformation
  repo: github.com/DKundnani/FeatureCorr
- name: R
  role: Runtime environment in which FeatureCorr::data_transform is called on the
    expression data frame
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.simpa.2021.100144
    title: 'FeatureCorr: An R package to study feature correlations aided with data
      transformation for sequencing and microarray data'
  source_content_hash: sha256:4fc7849bb205d53e2d5e7c187d1cc04d68b738d9e9520411b0193137a4c3a95d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FeatureCorr
  - R
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# median-threshold-feature-filtering

## Summary

A preprocessing skill from FeatureCorr that removes low-abundance features by applying a median expression threshold (medianthres) before data transformation, so that downstream correlation analysis (log2/z-score, pairwise or prime-feature) is computed only on features with sufficient signal.

## When to use

Use when a sequencing (NGS/RNA-seq) or microarray expression data frame contains low-abundance or background-noise features that would distort correlation analysis, and you need to filter them by median expression (e.g. medianthres=1) prior to transformation.

## When NOT to use

- Single-cell data where the article notes non-viable cells and low-abundance genes require read count filtration as a separate step rather than a simple median threshold
- Data already transformed and normalized — re-filtering a transformed table would discard legitimately transformed low-scale values
- Features are methylation or transcription data mixed in one data frame; the article suggests grouping with featuregroup (e.g. grepl) instead of threshold filtering

## Inputs

- Expression data frame from NGS sequencing or microarray experiments (e.g. TCGA40: 2000 TCGA Pan Cancer samples x 40 genes; GTEX_8000: ~8000 genes x 53 tissues)
- Feature identifier list (e.g. rownames(df) or GTEX$Description)
- Median expression threshold value (medianthres, e.g. 1)

## Outputs

- A list whose first element (transdf[[1]]) is the median-filtered, log2-transformed expression data frame retaining the same feature identifiers
- Optionally, density/distribution plots of expression values before vs. after transformation showing reduced spread in high-count features

## How to apply

Load the FeatureCorr package in R and pass the expression data frame to FeatureCorr::data_transform, setting transformation='log2' (recommended for sequencing data to reduce PCR artifacts and heteroskedasticity in high-signal features), featurelist to the feature identifiers (e.g. rownames(df) or GTEX$Description), and medianthres to the median-expression cutoff (the paper's example uses medianthres=1) so features below the threshold are filtered before transformation. The filtering happens inside data_transform, and the function returns a list whose first element is the transformed, filtered data frame (retrieve with transdf[[1]]). Choose the medianthres value relative to the data's expression scale; for the packaged TCGA40 (TCGA Pan Cancer, 2000 samples, 40 genes) and GTEX_8000 (~8000 genes, 53 tissues) examples, medianthres=1 is the demonstrated setting. The filtered, transformed data frame then feeds primefeature_corr, pair_scatter, or pairwise_corr.

## Related tools

- **FeatureCorr** (Provides the data_transform function whose medianthres argument performs the median-expression feature filtering before log2 or z-score transformation) — github.com/DKundnani/FeatureCorr
- **R** (Runtime environment in which FeatureCorr::data_transform is called on the expression data frame)

## Examples

```
transdf <- data_transform(df=TCGA40, transformation='log2', featurelist=rownames(TCGA40), medianthres=1); inputdf <- transdf[[1]]
```

## Evaluation signals

- Features with median expression below the medianthres cutoff (1 in the paper's examples) are absent from the returned data frame
- The filtered/transformed object retains the expected feature identifiers — e.g. the TCGA40 workflow must keep the same 40 gene features
- transdf[[1]] must be the transformed data frame, confirming the list structure returned by data_transform
- A before/after density plot shows the log2-transformed distribution with reduced spread (heteroskedasticity) in high-count features, consistent with the stated rationale

## Limitations

- Single-cell experiments have non-viable cells and low-abundance genes that require read count filtration; a median threshold alone is insufficient
- The threshold choice (medianthres) is demonstrated as 1 in the paper's examples but is dataset-scale dependent; an unsuitable cutoff can drop valid features
- Log2 transformation is recommended specifically for sequencing data; for microarray data other transformations (z-score) may be more appropriate

## Evidence

- [methods] medianthres filtering in data_transform call: "transdf<- data_transform(df=GTEX[-1],transformation='log2', featurelist=GTEX$Description, medianthres=1)"
- [abstract] rationale for filtering low-abundance features: "there are still limitations to single-cell experiments due to non-viable cells and low abundance genes that require read count filtration"
- [abstract] rationale for log2 transformation in sequencing data: "Log2 transformation is most frequently used with sequencing data to reduce PCR artifacts and heteroskedasticity, i.e., high variation in features with high signal or read count."
- [readme] TCGA40 input dataset description: "Another dataset is from The Cancer Genome Atlas Pan Cancer dataset randomly selected 2000 samples from >10K samples, with information for selected 40 features(genes)"
