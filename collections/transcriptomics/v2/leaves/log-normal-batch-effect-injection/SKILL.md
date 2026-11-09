---
name: log-normal-batch-effect-injection
description: 'Use when building synthetic multi-sample multi-condition scRNA-seq benchmark
  data: condition-permuted counts must be given sample-level technical variation (batch
  structure) before ground-truth DS genes are injected.'
when_to_use_negative:
- The task is to remove or correct batch variability from real data — that is scMerge2
  integration, not simulation of batch effects
- The analysis must preserve genuine condition effects (e.g., real disease-vs-control
  contrasts); this pipeline first permutes condition labels to construct a null dataset
- The input already contains real batch artifacts (platform/chemistry effects) — adding
  synthetic log-normal batch effects would confound them
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMerge2
  role: Integration algorithm benchmarked on the synthetic multi-sample multi-condition
    datasets produced by this batch-effect injection simulation workflow
  repo: https://github.com/SydneyBioX/scMerge
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39923-2
    title: Atlas-scale single-cell multi-sample multi-condition data integration using
      scMerge2
  source_content_hash: sha256:431d6048ecb2d16b3205e44eb1628411a7a6e371db9e3e4b9c2ec18b69e588f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SydneyBioX/scMerge
  tools:
  - scMerge2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# log-normal-batch-effect-injection

## Summary

A simulation step that adds log-normal, sample-level batch effects to condition-permuted single-cell counts, producing a controlled two-condition, two-batch null onto which ground-truth DS genes (logFC 1.1–2 at 5%/10%) are injected to benchmark DS detection and integration methods such as scMerge2.

## When to use

Use when building synthetic multi-sample multi-condition scRNA-seq benchmark data: condition-permuted counts must be given sample-level technical variation (batch structure) before ground-truth DS genes are injected.

## When NOT to use

- The task is to remove or correct batch variability from real data — that is scMerge2 integration, not simulation of batch effects
- The analysis must preserve genuine condition effects (e.g., real disease-vs-control contrasts); this pipeline first permutes condition labels to construct a null dataset
- The input already contains real batch artifacts (platform/chemistry effects) — adding synthetic log-normal batch effects would confound them

## Inputs

- Condition-permuted scRNA-seq count matrix derived from the Stephenson training subset (4 cell types, 23 training samples)
- Sample-level metadata mapping samples to the two batches and two conditions

## Outputs

- Batch-effect-carrying synthetic count matrices (two conditions, two batches) written to simulated_counts.rds
- Ground-truth DS gene label annotations paired with the synthetic matrices

## How to apply

Start from the condition-permuted Stephenson training-subset count matrix (4 cell types, 23 training samples), which already contains no genuine condition effect. Draw batch effects from a log-normal distribution and apply them at the sample level, so all cells from the same sample/batch share one multiplicative shift while the shift is independent of the permuted condition labels. This creates the controlled null dataset onto which the controlled DS gene set (log fold-changes drawn between 1.1 and 2, at DS gene proportions of 5% and 10%) is subsequently injected across the four cell types. Keep the injection within the specified logFC range so downstream benchmarking of integration/DS-detection methods (e.g., scMerge2) sees a well-defined ground truth. Write the batched synthetic count matrices together with the ground-truth DS gene label annotations to simulated_counts.rds.

## Related tools

- **scMerge2** (Integration algorithm benchmarked on the synthetic multi-sample multi-condition datasets produced by this batch-effect injection simulation workflow) — https://github.com/SydneyBioX/scMerge

## Examples

```
batch_fac <- rlnorm(nlevels(batch_id)); sim_counts <- sweep(counts_perm, 2, batch_fac[batch_id], "*"); saveRDS(sim_counts, "simulated_counts.rds")
```

## Evaluation signals

- Every injected DS gene's log fold-change falls within the specified 1.1–2 range
- DS gene proportions in the synthetic matrices match the 5% and 10% targets
- The permuted null (before DS injection) shows no genuine condition effect, since condition labels were permuted across samples
- simulated_counts.rds contains both the synthetic count matrices and the ground-truth DS gene label annotations
- Batch effects are sample-level: all cells within a sample share the same log-normal shift, consistent across the two batches

## Limitations

- Batch effects are modeled as sample-level log-normal multiplicative shifts, which will not reproduce all real technical artifacts
- The simulated design covers only two conditions and two batches over 4 cell types and 23 training samples, so extrapolation to many-batch atlas-scale settings is untested
- Only DS gene proportions of 5% and 10% with logFC 1.1–2 were injected; other regimes are not covered by the validated targets

## Evidence

- [methods] Add log-normal batch effects to the permuted counts to simulate sample-level technical variation.: [evidence span withheld — anchor status: not_found]
- [methods] DS genes with log fold-changes drawn between 1.1 and 2, at DS gene proportions of 5% and 10%: [evidence span withheld — anchor status: not_found]
- [abstract] scMerge2 presented as scalable multi-sample multi-condition integration method: "Here, we present scMerge2, a scalable algorithm that allows data integration of atlas-scale multi-sample multi-condition single-cell studies."
- [readme] Input counts are post-QC without gene filtering: [evidence span withheld — anchor status: external_doc]
