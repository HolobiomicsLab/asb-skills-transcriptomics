---
name: linear-regression-of-log-coverage-vs-base-position
description: Use when a base-level RNA-seq coverage dataset shows systematic 3'-end
  coverage decline per sample, and you need per-sample decay rates to identify degraded
  cases before shape change detection.
when_to_use_negative:
- Input is only gene-level expression counts (e.g. RSEM gene summaries) rather than
  base-level pileup coverage — slopes across base position cannot be computed
- Goal is detection of splicing/deletion shape changes rather than sample-level degradation
  screening — use SCISSOR's GSC/LSC detection on non-degraded samples instead
- Genes with many non-expressed samples (e.g. the 5802 genes ~28% excluded in HNSC)
  are retained — coverage-fitting on non-expressed genes yields unreliable decay rates
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCISSOR
  role: R package providing the base-level coverage pipeline and the exported decay.rate.hy
    function used to fit log-coverage vs base-position regressions
  repo: https://github.com/hyochoi/SCISSOR
- name: decay.rate.hy
  role: Exported SCISSOR function that computes per-sample per-gene RNA decay rates
    from coverage and distance-to-end inputs
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:df9814951cc574cdbdda97d0542cca56b6e0fa8dceb9d4045510064ac78370d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  - decay.rate.hy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# linear-regression-of-log-coverage-vs-base-position

## Summary

Quantifies sample-level RNA degradation by regressing mean-centered, log-transformed base-level pileup coverage on scaled base position; the per-sample slope (decay rate) captures 3'-to-5' coverage decline and is used to flag severely degraded samples.

## When to use

Use when a base-level RNA-seq coverage dataset shows systematic 3'-end coverage decline per sample, and you need per-sample decay rates to identify degraded cases before shape change detection.

## When NOT to use

- Input is only gene-level expression counts (e.g. RSEM gene summaries) rather than base-level pileup coverage — slopes across base position cannot be computed
- Goal is detection of splicing/deletion shape changes rather than sample-level degradation screening — use SCISSOR's GSC/LSC detection on non-degraded samples instead
- Genes with many non-expressed samples (e.g. the 5802 genes ~28% excluded in HNSC) are retained — coverage-fitting on non-expressed genes yields unreliable decay rates

## Inputs

- Base-level RNA-seq pileup coverage matrix (samples × gene × base position), e.g. built via SCISSOR build_pileup
- Genomic ranges per gene from SCISSOR get_Ranges (providing distance-to-end coordinates, dist2end)

## Outputs

- Samples-by-genes decay-rate matrix (per-sample per-gene regression slopes), exportable as decay_rate_matrix.csv
- List of severely degraded sample IDs for exclusion (e.g. excluded_sample_ids.txt; 70 of 522 TCGA HNSC samples in the reference application)

## How to apply

Load base-level pileup coverage into SCISSOR (e.g. `data(TOY_data)` after `library(SCISSOR)`), then log-transform the coverage and mean-center it per gene so sample-specific 3'-to-5' slopes are comparable. For every sample–gene pair, fit a linear regression of the mean-corrected log-coverage on scaled base position using the exported function `decay.rate.hy(Data, dist2end, robust)`, obtaining a per-sample decay rate. Assemble rates into a samples-by-genes decay-rate matrix, then cluster samples (e.g. hierarchical clustering) on this matrix and extract the cluster of severely degraded cases. Rationale: degraded RNA produces a monotonic 3'-to-5' coverage slope across many genes, so regression slope is a robust degradation statistic; in the 522-sample TCGA HNSC cohort this procedure identified 70 severely degraded cases that were excluded before downstream shape change detection.

## Related tools

- **SCISSOR** (R package providing the base-level coverage pipeline and the exported decay.rate.hy function used to fit log-coverage vs base-position regressions) — https://github.com/hyochoi/SCISSOR
- **decay.rate.hy** (Exported SCISSOR function that computes per-sample per-gene RNA decay rates from coverage and distance-to-end inputs) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); data(TOY_data); rates <- decay.rate.hy(Data, dist2end, robust)
```

## Evaluation signals

- The per-sample decay-rate matrix has one row per sample and one column per gene, with every sample receiving a fitted regression slope for each expressed gene
- Hierarchical clustering on the decay-rate matrix isolates a degraded cluster; in the 522-sample TCGA HNSC cohort the excluded list must contain exactly 70 sample IDs
- Mean-centering per gene precedes slope fitting, so slopes are comparable across genes regardless of absolute coverage level
- Excluded samples should show elevated decay rates across a large fraction of genes, matching 'severely degraded in a large fraction of genes' rather than single-gene anomalies

## Limitations

- SCISSOR tuning parameters including log normalization and a normality cutoff value likely require optimization when a significantly different wet lab RNA-seq protocol is used
- Inability to detect residual RNA degradation does not exclude the possibility of its existence
- Variable fragment length in otherwise high-quality TCGA-like RNA may introduce artifactual signals detectable by SCISSOR

## Evidence

- [results] For 522 HNSC tumor patients, we measured the amount of degradation across genes: "For 522 HNSC tumor patients, we measured the amount of degradation across genes"
- [results] identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis: "identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis"
- [abstract] a series of procedures for transforming and normalizing base-level RNA sequencing coverage data: [evidence span withheld — anchor status: not_found]
- [methods] install_github("hyochoi/SCISSOR"): "install_github("hyochoi/SCISSOR")"
- [discussion] We addressed this by empirically removing samples with evidence of degradation.: "We addressed this by empirically removing samples with evidence of degradation."
