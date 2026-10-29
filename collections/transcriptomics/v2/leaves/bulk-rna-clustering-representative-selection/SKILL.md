---
name: bulk-rna-clustering-representative-selection
description: Use when you have bulk RNA-seq data for a cohort (e.g., 124-sample COVID-19
  dataset) and must choose a small set of informative samples to receive real single-cell
  sequencing before semi-profiling the rest.
when_to_use_negative:
- You need single-cell data for a target sample from bulk data alone with a single-cell
  reference — use scSemiProfiler's bulk deconvolution (single-cell level) pipeline
  instead of representative selection.
- The cohort has already been fully single-cell sequenced, so representative selection
  offers no cost saving.
- You have no bulk RNA-seq data for the cohort; clustering-based selection cannot
  be performed without bulk profiles.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scSemiProfiler
  role: Provides initial setup (normalization, log1p, HVG selection), PCA reduction,
    K-Means clustering, global mode (inspect_data, global_stop_checking), and the
    active learning representative-selection module
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: scikit-learn (PCA, K-Means, Silhouette)
  role: Implements the PCA dimensionality reduction and K-Means clustering used to
    group bulk samples and pick centroid-nearest representatives
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:4fc93a883b6088b24939238f8b54c38e27b4dde52ef6819eaa0686ab509c91df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - scSemiProfiler
  - scikit-learn (PCA, K-Means, Silhouette)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# bulk-rna-clustering-representative-selection

## Summary

Selects representative samples for targeted single-cell sequencing by clustering PCA-reduced bulk RNA-seq data with K-Means and picking the sample closest to each cluster centroid, either once (global mode) or iteratively via active learning within scSemiProfiler.

## When to use

Use when you have bulk RNA-seq data for a cohort (e.g., 124-sample COVID-19 dataset) and must choose a small set of informative samples to receive real single-cell sequencing before semi-profiling the rest.

## When NOT to use

- You need single-cell data for a target sample from bulk data alone with a single-cell reference — use scSemiProfiler's bulk deconvolution (single-cell level) pipeline instead of representative selection.
- The cohort has already been fully single-cell sequenced, so representative selection offers no cost saving.
- You have no bulk RNA-seq data for the cohort; clustering-based selection cannot be performed without bulk profiles.

## Inputs

- Preprocessed bulk RNA-seq cohort matrix (e.g., 124-sample COVID-19 bulk data as AnnData/.h5ad from the scSemiProfiler example_data folder)
- Manually curated marker genes to add to the 6000 highly variable genes
- Desired number of clusters B (e.g., B=4) or a target representative count (e.g., 28)

## Outputs

- List of representative sample IDs (e.g., 28 samples written one-per-row to global_selection.csv with an explicit sample-ID column)
- Cluster memberships for all cohort samples
- Updated representatives_per_round records across active-learning rounds

## How to apply

Run scSemiProfiler's initial setup: library-size normalization, log1p transformation, and selection of the top 6000 highly variable genes plus manually added markers. Reduce the processed bulk data with scikit-learn PCA, using a number of PCs equal to sample size minus one when the cohort has fewer than 100 samples (e.g., 27 PCs for 28 samples) and 100 PCs for larger cohorts like the 124-sample COVID-19 data. Cluster the PCA-reduced data into B clusters with K-Means (e.g., B=4 for the initial active-learning setup of the COVID-19 cohort) and designate, per cluster, the sample closest to the centroid as the initial representative; in global mode, assess heterogeneity with Silhouette scores to fix the cluster count and select all representatives in one round. For iterative refinement, use the active learning module, ranking clusters by combined heterogeneity Db + λsc*Dsc + λpb*Dpb with λsc = λpb = 1, selecting the splitting sample that minimizes total in-cluster heterogeneity, updating memberships, and re-running inference — the COVID-19 case added 6 rounds of 4 samples to reach 28 representatives. Rationale: representative selection concentrates sequencing budget on samples that capture cohort heterogeneity, which is what the VAE-GAN model needs to accurately infer single-cell data for non-representative targets.

## Related tools

- **scSemiProfiler** (Provides initial setup (normalization, log1p, HVG selection), PCA reduction, K-Means clustering, global mode (inspect_data, global_stop_checking), and the active learning representative-selection module) — https://github.com/mcgilldinglab/scSemiProfiler
- **scikit-learn (PCA, K-Means, Silhouette)** (Implements the PCA dimensionality reduction and K-Means clustering used to group bulk samples and pick centroid-nearest representatives)

## Examples

```
import scSemiProfiler; scSemiProfiler.utils.inspect_data(cohort_bulk_data, topgens=6000)
```

## Evaluation signals

- global_selection.csv exists with exactly 28 data rows and a present sample-ID field for the COVID-19 cohort example
- Each selected representative is the sample closest to its K-Means cluster centroid in the PCA-reduced bulk space
- Active-learning iterations increase representatives by the intended batch size (e.g., 6 rounds × 4 samples to reach 28) while cluster heterogeneity scores (Db + λsc*Dsc + λpb*Dpb) decrease
- Silhouette-score assessment supports the chosen cluster count before global-mode selection
- Downstream semi-profiling of the selected representatives stays within the expected cost budget (e.g., $60,000–$66,000, a 0.30–0.37 ratio of the $186,000 full-cohort cost)

## Limitations

- Global mode performs comparably but slightly inferior to multi-round active learning (per Supplementary Fig. S8), so one-shot selection may under-represent heterogeneity
- Multi-round active learning can challenge laboratories and introduce batch effects, motivating the global-mode alternative
- The PCA PC count depends on cohort size (sample-size minus one for <100 samples, 100 PCs for larger cohorts), so very small cohorts may limit the representational space

## Evidence

- [methods] The dimensionality-reduced data is clustered into B distinct clusters using the K-Means algorithm: "The dimensionality-reduced data is clustered into B distinct clusters using the K-Means algorithm"
- [methods] scSemiProfiler offers a "global mode". This mode enables users to assess data heterogeneity using Silhouette scores: [evidence span withheld — anchor status: not_found]
- [methods] we perform library size normalization followed by a log1p transformation: "we perform library size normalization followed by a log1p transformation"
- [discussion] The pipeline ingests bulk data, leverages an active learning module to judiciously select representatives for authentic single-cell sequencing: "The pipeline ingests bulk data, leverages an active learning module to judiciously select representatives for authentic single-cell sequencing"
- [readme] New global mode functions "inspect_data" and "global_stop_checking" have been introduced: "New global mode functions "inspect_data" and "global_stop_checking" have been introduced"
