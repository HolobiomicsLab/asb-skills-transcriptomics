---
name: ground-truth-label-annotation
description: Use when generating synthetic benchmark data (e.g. two-condition, two-batch
  scRNA-seq simulations) that require known injected signal — DS genes at specified
  proportions (5%/10%) and logFC ranges — to be written alongside the counts as label
  annotations.
when_to_use_negative:
- Input is real experimental data without known ground truth — labels must come from
  a controlled injection, not inferred calls
- The goal is unsupervised integration only (e.g. standard scMerge2 pseudo-replication)
  with no downstream DS/differential benchmark to validate
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMerge2
  role: Integration method whose multi-sample multi-condition DS detection is benchmarked
    against the injected ground-truth DS gene labels
  repo: https://github.com/SydneyBioX/scMerge
- name: sc_mixology
  role: Benchmarking dataset repository whose pattern of storing ground-truth labels
    in colData columns (e.g. cell_line_demuxlet, cell-line proportion columns) informs
    how DS gene annotations are attached and accessed
  repo: https://github.com/LuyiTian/sc_mixology
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:3080148659d37d883f1b4d34a008594551a7d91cb7d0f1353bbbb871aae729e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge2
  - sc_mixology
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# ground-truth-label-annotation

## Summary

Constructs and stores verifiable ground-truth labels for synthetic single-cell simulations — e.g. differential-state (DS) gene annotations with controlled logFC ranges — so downstream benchmark results can be validated against known truth.

## When to use

Use when generating synthetic benchmark data (e.g. two-condition, two-batch scRNA-seq simulations) that require known injected signal — DS genes at specified proportions (5%/10%) and logFC ranges — to be written alongside the counts as label annotations.

## When NOT to use

- Input is real experimental data without known ground truth — labels must come from a controlled injection, not inferred calls
- The goal is unsupervised integration only (e.g. standard scMerge2 pseudo-replication) with no downstream DS/differential benchmark to validate

## Inputs

- scRNA-seq count matrix from a training subset (e.g. Stephenson data, 4 cell types, 23 training samples)
- Condition and batch labels per sample
- DS gene injection parameters: log fold-change range (1.1–2) and DS gene proportions (5% and 10%)

## Outputs

- Synthetic count matrix (e.g. simulated_counts.rds) with two conditions and two batches
- Ground-truth DS gene label annotations specifying which genes are DS and their injected log fold-changes

## How to apply

Start from a real count matrix (e.g. the Stephenson training-subset scRNA-seq data restricted to 4 cell types and 23 training samples) and permute condition labels across samples to create a null dataset with no genuine condition effect. Add log-normal batch effects to the permuted counts to simulate sample-level technical variation. Then inject a controlled set of differential-state (DS) genes with log fold-changes drawn between 1.1 and 2, at DS gene proportions of 5% and 10%, across the four cell types. Persist the synthetic count matrices together with the ground-truth DS gene label annotations (e.g. as simulated_counts.rds) so that integration methods like scMerge2 can be evaluated for DS detection against known truth. Record the injected logFC range and per-gene labels explicitly; these annotations, not the counts alone, are what make validation possible.

## Related tools

- **scMerge2** (Integration method whose multi-sample multi-condition DS detection is benchmarked against the injected ground-truth DS gene labels) — https://github.com/SydneyBioX/scMerge
- **sc_mixology** (Benchmarking dataset repository whose pattern of storing ground-truth labels in colData columns (e.g. cell_line_demuxlet, cell-line proportion columns) informs how DS gene annotations are attached and accessed) — https://github.com/LuyiTian/sc_mixology

## Examples

```
paste(sce_SC1_qc$H1975,sce_SC1_qc$H2228,sce_SC1_qc$HCC827,sep="_")
```

## Evaluation signals

- All injected DS genes have log fold-changes within the specified range (1.1–2)
- DS gene proportions in the synthetic matrices match the 5% and 10% targets
- Permuting condition labels yields a null dataset with no genuine condition effect (non-DS genes show no condition association)
- Ground-truth labels are co-persisted with the counts (e.g. in simulated_counts.rds) and recoverable at analysis time
- Non-injected genes are labeled as non-DS so false-positive rates can be computed for the benchmarked method

## Limitations

- Ground truth exists only for the injected DS genes; any condition effect the permutation fails to fully remove could contaminate the null labels
- Simulated batch effects are log-normal and may not capture real sample-level technical variation
- Labels are defined relative to the chosen 1.1–2 logFC window; weaker injected effects outside this range would be mislabeled as non-DS

## Evidence

- [abstract] sc_merge2 benchmarking purpose: [evidence span withheld — anchor status: not_found]
- [readme] ground-truth label storage in colData: [evidence span withheld — anchor status: not_found]
- [readme] label combination from colData columns: [evidence span withheld — anchor status: external_doc]
- [readme] proportion-based RNA mixture ground truth: [evidence span withheld — anchor status: external_doc]
