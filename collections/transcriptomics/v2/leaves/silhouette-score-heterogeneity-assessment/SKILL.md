---
name: silhouette-score-heterogeneity-assessment
description: Use when you have preprocessed bulk RNA-seq data for a cohort (e.g.,
  < 100 samples) and need to decide how many representative clusters exist so that
  all representatives can be selected in one batch instead of multi-round active learning.
when_to_use_negative:
- You plan to iteratively improve inference over multiple sequencing rounds — use
  scSemiProfiler's active learning module instead of the one-shot global mode
- You want per-target-sample single-cell deconvolution from bulk data plus a single-cell
  reference — that is the bulk deconvolution (single-cell level) pipeline, not heterogeneity
  assessment for representative selection
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: scSemiProfiler
  role: Provides the global mode ('inspect_data') that assesses data heterogeneity
    using Silhouette scores; also supplies initial setup (normalization, HVG selection)
    and global representative selection
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: scikit-learn
  role: Used to reduce bulk-data dimensionality with PCA before Silhouette scoring
    and K-Means clustering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:26a888de75659cf0ac64269af0c6d4d2283cf2ef8de4dd34e2cc5a8a88e6d925
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - scSemiProfiler
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# silhouette-score-heterogeneity-assessment

## Summary

A step within scSemiProfiler's global mode that assesses cohort heterogeneity using Silhouette scores and bulk-data visualization to determine the appropriate number of representative clusters before K-Means selection. It enables all representatives to be chosen in a single round from the initial bulk-data analysis, avoiding multi-round active learning and associated batch effects.

## When to use

Use when you have preprocessed bulk RNA-seq data for a cohort (e.g., < 100 samples) and need to decide how many representative clusters exist so that all representatives can be selected in one batch instead of multi-round active learning.

## When NOT to use

- You plan to iteratively improve inference over multiple sequencing rounds — use scSemiProfiler's active learning module instead of the one-shot global mode
- You want per-target-sample single-cell deconvolution from bulk data plus a single-cell reference — that is the bulk deconvolution (single-cell level) pipeline, not heterogeneity assessment for representative selection

## Inputs

- Preprocessed bulk RNA-seq data for the cohort (e.g., 28 COVID-19 cohort samples from the scSemiProfiler example_data folder)
- Candidate range of cluster counts (k) for Silhouette scoring
- Optionally, manually added marker genes to supplement the top 6000 highly variable genes

## Outputs

- Estimated number of representative clusters supported by Silhouette scores and bulk-data visualization
- Global-mode representative sample IDs (e.g., written one row per representative to global_selection.csv with an explicit sample-ID column)

## How to apply

Load the cohort's bulk RNA-seq data and run scSemiProfiler's initial setup: library size normalization, log1p transformation, and selection of the default top 6000 highly variable genes plus manually added markers. Reduce the processed bulk data with PCA implemented in scikit-learn, using a number of PCs equal to sample size minus one (e.g., 27 PCs for 28 samples, per the rule for cohorts with fewer than 100 samples). Compute Silhouette scores across candidate cluster counts and inspect bulk-data visualizations to gauge cohort heterogeneity and pick the approximate number of representative clusters. Then cluster the PCA-reduced data with K-Means into that many clusters and designate, for each cluster, the sample closest to the centroid as its representative — scSemiProfiler's global mode selects all representatives in this single batch.

## Related tools

- **scSemiProfiler** (Provides the global mode ('inspect_data') that assesses data heterogeneity using Silhouette scores; also supplies initial setup (normalization, HVG selection) and global representative selection) — https://github.com/mcgilldinglab/scSemiProfiler
- **scikit-learn** (Used to reduce bulk-data dimensionality with PCA before Silhouette scoring and K-Means clustering)

## Examples

```
import scSemiProfiler as scp; scp.utils.inspect_data(...)  # global mode: Silhouette-score heterogeneity assessment; then K-Means on PCA-reduced bulk data to write global_selection.csv
```

## Evaluation signals

- global_selection.csv exists and contains exactly 28 data rows (for the COVID-19 cohort) with a present, explicit sample-ID field
- Number of PCs used equals sample size minus one (e.g., 27 PCs for 28 samples)
- Number of K-Means clusters matches the cluster count supported by Silhouette score analysis
- Each cluster contributes exactly one representative — the sample closest to its centroid
- Qualitative expectation that global mode performs comparably but slightly inferior to active learning (Supplementary Fig. S8), deferred to expert review

## Limitations

- Global mode performs comparably but slightly inferior to active learning for representative selection
- The number of clusters is only an approximation based on Silhouette scores and visualization; it relies on user judgment
- Multi-round active learning can challenge laboratories and introduce batch effects — global mode trades some optimality for one-shot selection

## Evidence

- [methods] Global mode enables one-shot selection of representatives: [evidence span withheld — anchor status: not_found]
- [methods] PCA via scikit-learn for dimensionality reduction: "we employ Principal Component Analysis (PCA)83–85, implemented in the Python package Scikit-learn86, to reduce the dimensionality"
- [methods] K-Means clustering of the reduced data: "The dimensionality-reduced data is clustered into B distinct clusters using the K-Means algorithm87"
- [readme] Representative selection can be based on bulk clustering (global mode): [evidence span withheld — anchor status: external_doc]
- [methods] Normalization and HVG filter steps precede analysis: "we perform library size normalization followed by a log1p transformation"
