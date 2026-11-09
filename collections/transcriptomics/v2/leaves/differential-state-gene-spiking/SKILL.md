---
name: differential-state-gene-spiking
description: Use when building synthetic two-condition, multi-batch single-cell benchmark
  datasets that require a known set of DS genes with controlled log fold-change and
  proportion, e.g. to evaluate integration or DS-detection pipelines.
when_to_use_negative:
- The data already contain genuine condition effects you want to quantify — the null
  permutation step destroys real biology and is only appropriate for constructing
  controlled benchmark data.
- You need realistic multi-sample disease signatures rather than a synthetic evaluation
  set — use the real COVID-19 collection integration workflow instead.
- Ground truth for differential state is already experimentally available for your
  dataset, making simulated spiking unnecessary.
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: scMerge2
  role: Integration method whose DS-detection benchmark consumes the spiked-in synthetic
    datasets; the simulation framework is built in the scMerge2 paper
  repo: https://github.com/SydneyBioX/scMerge
- name: sc_mixology
  role: Benchmarking repository with ground-truth-labelled single-cell mixture datasets
    illustrating the ground-truth paradigm
  repo: https://github.com/LuyiTian/sc_mixology
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:4a03cdf787ca042e5c96fa56c42663f24fa7b0a9ac281720c198f985b460e2be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge2
  - sc_mixology
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Differential-state gene spiking into simulated scRNA-seq data

## Summary

A benchmarking skill that injects controlled ground-truth differential-state (DS) genes into a real scRNA-seq count matrix (with permuted condition labels and log-normal batch effects) so that DS-detection methods can be evaluated against known positives.

## When to use

Use when building synthetic two-condition, multi-batch single-cell benchmark datasets that require a known set of DS genes with controlled log fold-change and proportion, e.g. to evaluate integration or DS-detection pipelines.

## When NOT to use

- The data already contain genuine condition effects you want to quantify — the null permutation step destroys real biology and is only appropriate for constructing controlled benchmark data.
- You need realistic multi-sample disease signatures rather than a synthetic evaluation set — use the real COVID-19 collection integration workflow instead.
- Ground truth for differential state is already experimentally available for your dataset, making simulated spiking unnecessary.

## Inputs

- scRNA-seq count matrix (Stephenson training-subset, 4 cell types, 23 training samples)
- condition label assignments per sample
- DS gene injection parameters: log fold-change range 1.1–2 and DS gene proportions of 5% and 10%

## Outputs

- simulated_counts.rds containing synthetic two-condition, two-batch count matrices
- ground-truth DS gene label annotations (which genes are DS, at which logFC)

## How to apply

Start from the Stephenson training-subset scRNA-seq count matrix restricted to 4 cell types and 23 training samples, and permute condition labels across samples to construct a null dataset with no genuine condition effect (this ensures any signal recovered later is attributable to injected DS genes). Add log-normal batch effects to the permuted counts to simulate sample-level technical variation. Then inject DS genes with log fold-changes drawn between 1.1 and 2, at DS gene proportions of 5% and 10% of genes across the four cell types, covering two conditions and two batches. Finally, write the synthetic count matrices together with the ground-truth DS gene label annotations to simulated_counts.rds. The framework was used with the scMerge2 ecosystem to benchmark integration methods for DS detection, and the sc_mixology repository provides analogous ground-truth-labelled benchmarking data.

## Related tools

- **scMerge2** (Integration method whose DS-detection benchmark consumes the spiked-in synthetic datasets; the simulation framework is built in the scMerge2 paper) — https://github.com/SydneyBioX/scMerge
- **sc_mixology** (Benchmarking repository with ground-truth-labelled single-cell mixture datasets illustrating the ground-truth paradigm) — https://github.com/LuyiTian/sc_mixology

## Examples

```
write the synthetic count matrices and ground-truth DS gene labels to simulated_counts.rds after injecting DS genes with logFC in [1.1, 2] at 5% and 10% proportions
```

## Evaluation signals

- All injected DS genes have log fold-changes within the specified 1.1–2 range.
- The proportion of DS genes in the synthetic matrices matches the 5% and 10% targets.
- The permuted (null) dataset shows no genuine condition effect before spiking.
- Output is a valid simulated_counts.rds containing both count matrices and DS gene labels.
- Batch effects applied as log-normal shifts are present as sample-level technical variation across the two batches.

## Limitations

- The simulation is restricted to a null-then-spiking design, so unmodelled true condition effects or interactions between injected logFCs and real biology are not captured.
- Batch effects are simulated via log-normal additions only, which may not reflect other technical artefacts.
- Only two DS gene proportions (5%, 10%) and a single logFC range (1.1–2) were evaluated, limiting coverage of milder or more pervasive DS signals.

## Evidence

- [other] Add log-normal batch effects to the permuted counts to simulate sample-level technical variation.: [evidence span withheld — anchor status: not_found]
- [other] Inject DS genes with log fold-changes drawn between 1.1 and 2, at DS gene proportions of 5% and 10%.: [evidence span withheld — anchor status: not_found]
- [other] Permute condition labels to construct a null dataset with no genuine condition effect.: [evidence span withheld — anchor status: not_found]
- [abstract] scMerge2 benchmarking context from the abstract.: "scMerge2, a scalable algorithm that allows data integration of atlas-scale multi-sample multi-condition single-cell studies."
- [readme] sc_mixology provides ground-truth benchmarking data with known labels.: [evidence span withheld — anchor status: external_doc]
