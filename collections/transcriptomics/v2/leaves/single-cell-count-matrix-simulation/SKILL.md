---
name: single-cell-count-matrix-simulation
description: Use when benchmarking integration or differential-state detection methods
  needs synthetic two-condition, two-batch single-cell count data with known injected
  DS genes at controlled logFC (1.1–2) and prevalence (5% and 10%).
when_to_use_negative:
- The goal is integration of real, unpermuted data (e.g. the COVID-19 collection of
  5 million cells) — this skill is only for building synthetic benchmark data, not
  for running scMerge2 integration itself
- Ground-truth DS effects must be biologically real rather than injected — simulation
  replaces genuine condition effects with controlled logFC injections
- The input data has already been batch-corrected or integrated; the simulation pipeline
  needs raw permuted counts so log-normal batch effects can be added
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMerge2
  role: Underlying framework extended (via scDesign3) to generate the synthetic multi-sample
    multi-condition datasets used to benchmark integration methods for DS detection
  repo: https://github.com/SydneyBioX/scMerge
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:5c49aad2f073dc5054e713cd30369b13df0b851ef52c450bcf8414fc5775f354
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge2
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-count-matrix-simulation

## Summary

Generates synthetic multi-sample, multi-condition scRNA-seq count matrices with controlled ground-truth differential-state (DS) genes by extending the scMerge2 simulation framework (scDesign3) on real training-subset data. It exists so integration and DS-detection methods can be benchmarked against known injected effects rather than unverified biological signals.

## When to use

Use when benchmarking integration or differential-state detection methods needs synthetic two-condition, two-batch single-cell count data with known injected DS genes at controlled logFC (1.1–2) and prevalence (5% and 10%).

## When NOT to use

- The goal is integration of real, unpermuted data (e.g. the COVID-19 collection of 5 million cells) — this skill is only for building synthetic benchmark data, not for running scMerge2 integration itself
- Ground-truth DS effects must be biologically real rather than injected — simulation replaces genuine condition effects with controlled logFC injections
- The input data has already been batch-corrected or integrated; the simulation pipeline needs raw permuted counts so log-normal batch effects can be added

## Inputs

- Stephenson training-subset scRNA-seq count matrix (4 cell types, 23 training samples)
- Per-sample condition labels (for permutation into the null dataset)
- Cell-type annotations for each cell
- Simulation parameters: DS gene proportions (5% and 10%) and logFC range (1.1–2)

## Outputs

- simulated_counts.rds containing synthetic two-condition, two-batch count matrices
- Ground-truth DS gene label annotations per cell type and condition setting

## How to apply

Load the Stephenson training-subset scRNA-seq count matrix restricted to the 4 cell types and 23 training samples, then permute condition labels across samples to construct a null dataset with no genuine condition effect. Add log-normal batch effects to the permuted counts to simulate sample-level technical variation, yielding a two-condition, two-batch synthetic dataset. Inject a controlled set of DS genes whose log fold-changes are drawn between 1.1 and 2, at DS gene proportions of 5% and 10%, across the four cell types. Write the synthetic count matrices together with the ground-truth DS gene label annotations to simulated_counts.rds. Rationale: label permutation guarantees a valid null so any method signal on the synthetic data is attributable to the injected DS effects plus simulated batch variation.

## Related tools

- **scMerge2** (Underlying framework extended (via scDesign3) to generate the synthetic multi-sample multi-condition datasets used to benchmark integration methods for DS detection) — https://github.com/SydneyBioX/scMerge

## Examples

```
saveRDS(list(counts = sim_counts, ds_gene_labels = ds_truth), "simulated_counts.rds")
```

## Evaluation signals

- Check that the injected DS genes fall within the specified logFC range (1.1–2)
- Check that the DS gene proportions in the synthetic matrices match the 5% and 10% targets
- Confirm the null dataset (label-permuted counts) shows no genuine condition effect before injection
- Verify the output R object contains both the synthetic count matrices and the DS gene label annotations (simulated_counts.rds)
- Confirm the synthetic data retains the two-condition, two-batch structure across the four cell types

## Limitations

- The null is valid only as far as condition-label permutation removes genuine condition effects; residual structure may survive permutation
- Batch effects are simulated as log-normal sample-level technical variation, which may not capture real batch-effect complexity (the article notes integration order and modality handling remain data-guided and within-modality only)
- Synthetic datasets are restricted to the two-condition, two-batch design with two DS proportions (5% and 10%) and a logFC range of 1.1–2; other regimes require re-parameterisation

## Evidence

- [supplementary] Simulation design and DS injection parameters: [evidence span withheld — anchor status: not_found]
- [supplementary] Null construction and batch-effect simulation rationale: [evidence span withheld — anchor status: not_found]
- [abstract] scMerge2 as the multi-sample multi-condition framework the simulation extends: [evidence span withheld — anchor status: not_found]
- [readme] Tool source and installation context: "`scMerge` is a R package for merging and normalising single-cell RNA-Seq datasets."
