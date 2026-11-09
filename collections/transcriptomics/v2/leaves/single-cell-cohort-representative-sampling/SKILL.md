---
name: single-cell-cohort-representative-sampling
description: Use when a cohort's bulk sequencing data (e.g., preprocessed COVID-19
  cohort bulk RNA-seq from the scSemiProfiler example_data folder) must yield all
  representative samples in one batch, and multi-round active learning is impractical
  or would introduce batch effects.
when_to_use_negative:
- You need iterative, inference-driven representative selection to improve single-cell
  inference — use scSemiProfiler's Active Learning module instead of global mode
- The cohort has 100 or more samples, where the 'sample size minus one PCs' rule described
  for smaller cohorts no longer applies
- You only have single-target bulk data for one sample — use the bulk deconvolution
  (single-cell level) pipeline with a single-cell reference instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSemiProfiler
  role: Provides the global mode, initial setup (library size normalization, log1p,
    top-6000 HVG selection), Silhouette-score-based heterogeneity inspection, and
    representative selection
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: Scikit-learn
  role: Implements PCA dimensionality reduction and the K-Means clustering used to
    designate the sample closest to each centroid as representative
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:1eb0e134d37170dd12fb8f85f65fa6b83a3323fae7297fedbad6cf579b9ceffe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - Scikit-learn
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-cohort-representative-sampling

## Summary

Selects representative samples from a bulk RNA-seq cohort in a single round using scSemiProfiler's global mode, so that only a small set of samples needs real single-cell sequencing for semi-profiling. For the COVID-19 cohort, global mode selected 28 representatives.

## When to use

Use when a cohort's bulk sequencing data (e.g., preprocessed COVID-19 cohort bulk RNA-seq from the scSemiProfiler example_data folder) must yield all representative samples in one batch, and multi-round active learning is impractical or would introduce batch effects.

## When NOT to use

- You need iterative, inference-driven representative selection to improve single-cell inference — use scSemiProfiler's Active Learning module instead of global mode
- The cohort has 100 or more samples, where the 'sample size minus one PCs' rule described for smaller cohorts no longer applies
- You only have single-target bulk data for one sample — use the bulk deconvolution (single-cell level) pipeline with a single-cell reference instead

## Inputs

- Preprocessed bulk RNA-seq data for the cohort (e.g., 28 COVID-19 cohort samples from the scSemiProfiler example_data folder)
- Optional manually added marker gene list to supplement the default top 6000 highly variable genes

## Outputs

- global_selection.csv: one row per representative sample, with an explicit sample-ID column listing the selected representative sample IDs

## How to apply

Load the preprocessed bulk RNA-seq data and run scSemiProfiler's initial setup with library size normalization followed by log1p transformation, selecting the default top 6000 highly variable genes plus manually added markers. Reduce the processed bulk data with PCA implemented in scikit-learn, using a number of PCs equal to sample size minus one (27 PCs for 28 samples, the rule for cohorts with fewer than 100 samples). Assess cohort heterogeneity with Silhouette scores and bulk-data visualization to decide the approximate number of representative clusters, then cluster the PCA-reduced data into B distinct clusters with K-Means and designate, for each cluster, the sample closest to the centroid as its representative. This is the global mode, which performs all-in-one selection in a single round, in contrast to the incremental Active Learning module; write the selected representative sample IDs, one row per representative with an explicit sample-ID column, to global_selection.csv.

## Related tools

- **scSemiProfiler** (Provides the global mode, initial setup (library size normalization, log1p, top-6000 HVG selection), Silhouette-score-based heterogeneity inspection, and representative selection) — https://github.com/mcgilldinglab/scSemiProfiler
- **Scikit-learn** (Implements PCA dimensionality reduction and the K-Means clustering used to designate the sample closest to each centroid as representative)

## Examples

```
from sklearn.decomposition import PCA; from sklearn.cluster import KMeans; X_pca = PCA(n_components=27).fit_transform(X_norm); labels = KMeans(n_clusters=B).fit_predict(X_pca)
```

## Evaluation signals

- global_selection.csv exists and contains exactly the expected number of data rows (28 for the COVID-19 cohort) with a present sample-ID field
- Every selected representative is the sample closest to its K-Means cluster centroid in the 27-PC (sample size minus one) space
- Silhouette scores computed during inspection support the chosen number of representative clusters B
- HVG count matches the default top 6000 plus manually added markers after library size normalization and log1p transformation
- Qualitative claim that global mode performs comparably but slightly inferior to active learning (Supplementary Fig. S8) deferred to expert review

## Limitations

- Global mode is expected to perform comparably but slightly inferior to active learning-based incremental selection (Supplementary Fig. S8)
- Tool requires a single-cell reference to convert selections into full semi-profiling output ('single-cell level deconvolution' requires a single-cell reference)
- No software versions stated for scSemiProfiler or dependencies in the article discussion section

## Evidence

- [methods] Global Mode Representative Selection: [evidence span withheld — anchor status: not_found]
- [methods] PCA and K-Means in the global mode pipeline: "we employ Principal Component Analysis (PCA)83–85, implemented in the Python package Scikit-learn86, to reduce the dimensionality"
- [methods] K-Means clustering of reduced data: "The dimensionality-reduced data is clustered into B distinct clusters using the K-Means algorithm87"
- [methods] Preprocessing of bulk data: "we perform library size normalization followed by a log1p transformation"
- [readme] Global mode vs active learning selection in README: [evidence span withheld — anchor status: external_doc]
