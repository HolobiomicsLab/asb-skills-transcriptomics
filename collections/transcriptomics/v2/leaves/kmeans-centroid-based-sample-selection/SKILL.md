---
name: kmeans-centroid-based-sample-selection
description: Use when a cohort of bulk RNA-seq samples must be distilled into representatives
  for single-cell sequencing in one batch, and multi-round active learning is impractical
  or risks introducing batch effects across sequencing rounds.
when_to_use_negative:
- When iterative active learning is feasible and preferred, since active learning
  can incrementally refine representative selection and global mode was reported as
  comparable but slightly inferior (Supplementary Fig. S8)
- When deconvolving a single target sample with a single-cell reference rather than
  selecting representatives from a cohort
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scSemiProfiler
  role: Provides the global mode (inspect_data / global_stop_checking utilities) for
    Silhouette-score heterogeneity assessment, PCA reduction, K-Means clustering,
    and single-round representative selection from initial bulk data analysis
  repo: https://github.com/mcgilldinglab/scSemiProfiler
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:aa1b84d9b33013f07e9841ccafa862c88a70576354510f2d4262140b2d5b78c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# kmeans-centroid-based-sample-selection

## Summary

Selects all representative cohort samples in a single round by K-Means clustering of PCA-reduced bulk RNA-seq data, designating the sample closest to each cluster centroid as the representative — the 'global mode' of scSemiProfiler that avoids multi-round active learning.

## When to use

Use when a cohort of bulk RNA-seq samples must be distilled into representatives for single-cell sequencing in one batch, and multi-round active learning is impractical or risks introducing batch effects across sequencing rounds.

## When NOT to use

- When iterative active learning is feasible and preferred, since active learning can incrementally refine representative selection and global mode was reported as comparable but slightly inferior (Supplementary Fig. S8)
- When deconvolving a single target sample with a single-cell reference rather than selecting representatives from a cohort

## Inputs

- Preprocessed bulk RNA-seq data for all cohort samples (e.g., 28 COVID-19 cohort samples from the scSemiProfiler example_data folder)
- Approximate number of representative clusters B, determined via Silhouette scores and bulk-data visualization

## Outputs

- global_selection.csv listing selected representative sample IDs, one row per representative with an explicit sample-ID column

## How to apply

Load preprocessed bulk RNA-seq data (e.g., the 28-sample COVID-19 cohort from the scSemiProfiler example_data folder), then apply library size normalization followed by log1p transformation and select the default top 6000 highly variable genes plus manually added markers via scSemiProfiler's initial setup. Reduce the data with PCA (scikit-learn), using PCs equal to sample size minus one (27 PCs for 28 samples, per the rule for cohorts under 100 samples). Assess cohort heterogeneity with Silhouette scores and bulk-data visualization to pick the number of clusters B, then cluster the PCA-reduced data into B distinct clusters with K-Means. For each cluster, designate the sample closest to the centroid as its representative, and write the selected sample IDs (one row per representative, with an explicit sample-ID column) to global_selection.csv.

## Related tools

- **scSemiProfiler** (Provides the global mode (inspect_data / global_stop_checking utilities) for Silhouette-score heterogeneity assessment, PCA reduction, K-Means clustering, and single-round representative selection from initial bulk data analysis) — https://github.com/mcgilldinglab/scSemiProfiler

## Evaluation signals

- global_selection.csv exists with exactly 28 data rows for the COVID-19 cohort example and a present, non-empty sample-ID field
- Number of selected representatives equals the number of K-Means clusters B chosen from Silhouette-score analysis
- Each selected representative lies closest to its cluster centroid in the 27-PC space (sample size minus one for a 28-sample cohort)
- Every cluster contributes exactly one representative, and no sample ID appears more than once

## Limitations

- Global mode performs comparably but slightly inferior to active learning-based selection (Supplementary Fig. S8), so the qualitative claim should be deferred to expert review
- The rule of sample-size-minus-one PCs is specified for cohorts with fewer than 100 samples; behavior for very large cohorts is not covered here
- Extension to modalities beyond scRNA-seq (e.g., proteomics) is future work only, not implemented

## Evidence

- [methods] global mode enables single-round selection: [evidence span withheld — anchor status: not_found]
- [methods] K-Means clustering of reduced data: "The dimensionality-reduced data is clustered into B distinct clusters using the K-Means algorithm87"
- [methods] PCA via scikit-learn: "we employ Principal Component Analysis (PCA)83–85, implemented in the Python package Scikit-learn86, to reduce the dimensionality"
- [methods] normalization and transformation preprocessing: "we perform library size normalization followed by a log1p transformation"
- [readme] README: representative choice via clustering: [evidence span withheld — anchor status: external_doc]
