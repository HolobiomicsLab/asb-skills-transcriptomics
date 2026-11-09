---
name: condition-label-permutation-null-construction
description: Use when you need a controlled two-condition, multi-batch scRNA-seq benchmark
  with known ground-truth DS genes (e.g. to test DS detection or integration methods)
  from real count data lacking genuine condition effects.
when_to_use_negative:
- The input data contain genuine condition effects you want to study — permuting labels
  across samples will destroy them by design
- You need to benchmark integration (e.g. scMerge2 embedding quality on real cohorts
  like the COVID-19 collection) rather than DS-detection performance against known
  ground truth
- There are too few samples per condition to support meaningful label permutation
  across samples
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: scMerge2
  role: Integration method whose DS-detection benchmarking consumes the synthetic
    null/injected datasets produced by this skill
  repo: https://github.com/SydneyBioX/scMerge
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:ec104126b47bd37d783d24d04dba7607c150ea134d72a3ab17c468f016717ce4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 3
---

# Condition-label permutation null construction

## Summary

Constructs a synthetic scRNA-seq benchmark with ground-truth differential-state (DS) genes by permuting condition labels across samples to create a null, adding log-normal batch effects, and injecting DS genes at controlled log fold-changes (1.1–2) and proportions (5% and 10%).

## When to use

Use when you need a controlled two-condition, multi-batch scRNA-seq benchmark with known ground-truth DS genes (e.g. to test DS detection or integration methods) from real count data lacking genuine condition effects.

## When NOT to use

- The input data contain genuine condition effects you want to study — permuting labels across samples will destroy them by design
- You need to benchmark integration (e.g. scMerge2 embedding quality on real cohorts like the COVID-19 collection) rather than DS-detection performance against known ground truth
- There are too few samples per condition to support meaningful label permutation across samples

## Inputs

- scRNA-seq count matrix restricted to the 4 cell types and 23 Stephenson training samples (training-subset count data)
- Sample-level metadata with condition labels (to be permuted across samples)
- DS gene injection parameters: target DS gene proportions (5% and 10%) and log fold-change range (1.1–2)

## Outputs

- simulated_counts.rds containing synthetic two-condition, two-batch count matrices
- Ground-truth DS gene label annotations paired with the synthetic matrices
- Null (permuted-label) dataset with log-normal batch effects and no genuine condition effect

## How to apply

Start from a real scRNA-seq count matrix restricted to defined cell types and samples (here: the Stephenson training-subset matrix with 4 cell types and 23 training samples). Permute condition labels across samples so the resulting null dataset carries no genuine condition effect, then add log-normal batch effects to the permuted counts to mimic sample-level technical variation. Inject a controlled set of DS genes across the four cell types, drawing their log fold-changes uniformly between 1.1 and 2 and setting the DS gene proportion to either 5% or 10%. Write the synthetic count matrices together with the ground-truth DS gene label annotations to simulated_counts.rds. The rationale is that permuting labels destroys real condition signal while preserving the covariance structure of the data, so any DS signal detected downstream can only come from the injected genes.

## Related tools

- **scMerge2** (Integration method whose DS-detection benchmarking consumes the synthetic null/injected datasets produced by this skill) — https://github.com/SydneyBioX/scMerge

## Examples

```
saveRDS(list(counts = sim_counts, ds_genes = ds_labels), "simulated_counts.rds")
```

## Evaluation signals

- All injected DS genes have log fold-changes within the specified range of 1.1–2
- The proportion of DS genes in each synthetic matrix matches the 5% and 10% targets
- The permuted-label null dataset shows no genuine condition effect (downstream DS testing flags only the injected genes)
- Log-normal batch effects are present at the sample level across the two simulated batches
- simulated_counts.rds round-trips with both the count matrices and the DS gene label annotations intact

## Limitations

- The null is label-permutation-based, so it only approximates (rather than fully simulates) condition-free data; real DS structure may differ from log fold-changes injected between 1.1 and 2
- Only two conditions, two batches, and two DS proportions (5% and 10%) were generated, so benchmark conclusions may not generalize to more complex designs
- Only 4 cell types and 23 training samples from the Stephenson subset are used, limiting cell-type and donor diversity of the synthetic benchmark

## Evidence

- [methods] Permute condition labels across samples to construct a null dataset with no genuine condition effect.: [evidence span withheld — anchor status: not_found]
- [methods] Inject DS genes with controlled log fold-changes and proportions across cell types.: [evidence span withheld — anchor status: not_found]
- [supplementary] Validation that injected DS genes match the logFC range and proportion targets.: [evidence span withheld — anchor status: not_found]
- [abstract] scMerge2 as the integration method these synthetic benchmarks evaluate for DS detection.: "Here, we present scMerge2, a scalable algorithm that allows data integration of atlas-scale multi-sample multi-condition single-cell studies."
