---
name: hierarchical-clustering-of-sample-decay-rates
description: Use when a base-level RNA-seq cohort shows 3'-to-5' coverage decline
  in some samples and you must identify and exclude degraded samples so they do not
  appear as spurious shape-change outliers.
when_to_use_negative:
- Input is a gene-level count or RSEM expression table rather than base-level pileup
  coverage — decay slopes cannot be computed without per-base coverage
- The cohort is smaller than the ~30-sample minimum the SCISSOR authors report as
  the smallest successfully analyzed cohort
- The goal is differential expression rather than detecting coverage shape aberrations
  or sample degradation outliers
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: SCISSOR
  role: R package providing the base-level coverage framework, TOY_data example, and
    sample degradation screening workflow
  repo: https://github.com/hyochoi/SCISSOR
- name: decay.rate.hy
  role: exported SCISSOR function that fits the linear regression of mean-corrected
    log-coverage on scaled base position to compute per-sample decay rates
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:bf050fc8210eeadd567b6e3d3a6a3db4223d12e18109beefdc8f60b18ba03e6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  - decay.rate.hy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# hierarchical-clustering-of-sample-decay-rates

## Summary

Clusters RNA-seq samples by their per-gene RNA decay rates (3'-to-5' coverage slopes) using hierarchical clustering, in order to isolate severely degraded tumor samples — e.g. 70 of 522 TCGA HNSC cases — before SCISSOR shape change detection.

## When to use

Use when a base-level RNA-seq cohort shows 3'-to-5' coverage decline in some samples and you must identify and exclude degraded samples so they do not appear as spurious shape-change outliers.

## When NOT to use

- Input is a gene-level count or RSEM expression table rather than base-level pileup coverage — decay slopes cannot be computed without per-base coverage
- The cohort is smaller than the ~30-sample minimum the SCISSOR authors report as the smallest successfully analyzed cohort
- The goal is differential expression rather than detecting coverage shape aberrations or sample degradation outliers

## Inputs

- base-level RNA-seq pileup coverage matrix (samples x bases per gene), e.g. built via SCISSOR's build_pileup
- scaled base-position (distance-to-end) vector per gene
- optional robust-regression flag for decay.rate.hy

## Outputs

- samples-by-genes decay-rate matrix (e.g. decay_rate_matrix.csv)
- list of severely degraded sample IDs to exclude (e.g. excluded_sample_ids.txt, expected to contain 70 IDs for the 522-sample TCGA HNSC cohort)

## How to apply

Load the base-level coverage data with SCISSOR (e.g. `data(TOY_data)` after `library(SCISSOR)`), then log-transform the pileup coverage and mean-center it per gene so sample-specific 3'-to-5' slopes are comparable. For each sample and gene, fit the linear regression of mean-corrected log-coverage on scaled base position using the exported function `decay.rate.hy(Data, dist2end, robust)` to obtain per-sample decay rates, and assemble them into a samples-by-genes decay-rate matrix. Run hierarchical clustering on the samples over this matrix, identify the cluster of severely degraded cases, and extract their sample IDs for exclusion. The rationale is that degraded RNA produces systematic coverage slopes that mimic structural aberrations; SCISSOR's degradation measurement across genes caught 70 severely degraded cases out of 522 TCGA HNSC tumors, which were removed before downstream shape change analysis.

## Related tools

- **SCISSOR** (R package providing the base-level coverage framework, TOY_data example, and sample degradation screening workflow) — https://github.com/hyochoi/SCISSOR
- **decay.rate.hy** (exported SCISSOR function that fits the linear regression of mean-corrected log-coverage on scaled base position to compute per-sample decay rates) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); data(TOY_data); rates <- apply(gene_coverages, 1, function(x) decay.rate.hy(Data=x, dist2end=positions, robust=TRUE))
```

## Evaluation signals

- The excluded sample list contains exactly 70 IDs, matching the reported count of severely degraded cases excluded from the 522-sample TCGA HNSC cohort
- The decay-rate matrix has one row per sample and one column per gene, with finite values for samples with expressed genes
- Per-sample decay rates are computed from regressions of mean-corrected, log-transformed coverage on scaled base position (negative slopes indicating 3'-to-5' decay)
- Samples flagged as degraded form a distinct cluster in the dendrogram and, once removed, no longer surface as spurious outliers in downstream shape change detection
- Genes with many non-expressed samples (e.g. the 5802 genes, ~28%, excluded in the paper) are dropped before clustering to avoid noise

## Limitations

- SCISSOR tuning parameters including log normalization and the normality cutoff value likely require optimization when a significantly different wet-lab RNA-seq protocol is used
- Inability to detect residual RNA degradation does not exclude the possibility of its existence, so some degraded samples may remain after filtering
- Variable fragment length may introduce artifactual signals detectable by SCISSOR even among non-degraded samples

## Evidence

- [results] For 522 HNSC tumor patients, we measured the amount of degradation across genes: "For 522 HNSC tumor patients, we measured the amount of degradation across genes"
- [results] identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis: "identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis"
- [methods] A toy dataset is available in the `SCISSOR` package:

r
library(SCISSOR)
data(TOY_data): "A toy dataset is available in the `SCISSOR` package:

r
library(SCISSOR)
data(TOY_data)"
- [results] Because degraded RNA can cause systematic spurious outliers: "identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis"
- [discussion] We addressed this by empirically removing samples with evidence of degradation.: "We addressed this by empirically removing samples with evidence of degradation."
- [methods] `SCISSOR` is free software and available on [Github](https://github.com/hyochoi/SCISSOR).: "`SCISSOR` is free software and available on [Github](https://github.com/hyochoi/SCISSOR)."
