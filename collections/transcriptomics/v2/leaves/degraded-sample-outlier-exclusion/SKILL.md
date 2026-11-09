---
name: degraded-sample-outlier-exclusion
description: Use when a cohort of base-level RNA-seq coverage data must be screened
  for degraded samples whose 3'-to-5' decay slopes would create spurious outliers
  in downstream shape/outlier analyses.
when_to_use_negative:
- Input is a gene-level expression count matrix rather than base-level pileup coverage
  — decay slopes along transcript position cannot be computed.
- The goal is to keep and analyze degraded samples or study degradation itself rather
  than exclude degraded cases before shape change detection.
- The cohort is smaller than the ~30-sample minimum SCISSOR's outlier framework was
  empirically designed for.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: SCISSOR
  role: R package providing log transformation, base-level normalization, and the
    framework in which degradation-based sample exclusion is performed before shape
    change detection
  repo: https://github.com/hyochoi/SCISSOR
- name: decay.rate.hy
  role: Exported SCISSOR function fitting per-sample linear regression of mean-corrected
    log-coverage on scaled base position to quantify RNA decay rate per gene
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:a1b6d16e9cce4d268f3e6c7efa46ce9c2a743113e8b5a292f5c5d22c6e9c6668
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  - decay.rate.hy
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# degraded-sample-outlier-exclusion

## Summary

Identify and exclude RNA-seq samples with severe RNA degradation before shape change detection by computing per-sample 3'-to-5' coverage decay rates (e.g., via SCISSOR's decay.rate.hy on base-level pileup data), clustering samples on the decay-rate matrix, and removing the degraded cluster.

## When to use

Use when a cohort of base-level RNA-seq coverage data must be screened for degraded samples whose 3'-to-5' decay slopes would create spurious outliers in downstream shape/outlier analyses.

## When NOT to use

- Input is a gene-level expression count matrix rather than base-level pileup coverage — decay slopes along transcript position cannot be computed.
- The goal is to keep and analyze degraded samples or study degradation itself rather than exclude degraded cases before shape change detection.
- The cohort is smaller than the ~30-sample minimum SCISSOR's outlier framework was empirically designed for.

## Inputs

- Base-level RNA-seq pileup coverage matrix (e.g., built with SCISSOR build_pileup / data(TOY_data))
- Base position / distance-to-end annotations per gene (dist2end)
- Cohort sample IDs (e.g., 522 TCGA HNSC tumor samples)

## Outputs

- Samples-by-genes decay-rate matrix (e.g., decay_rate_matrix.csv)
- List of excluded severely degraded sample IDs (e.g., excluded_sample_ids.txt)
- Filtered cohort (e.g., 452 remaining TCGA HNSC samples) for downstream shape change detection

## How to apply

Load base-level pileup coverage (e.g., data(TOY_data) from the SCISSOR package), log-transform the coverage and mean-center it per gene so sample-specific 3'-to-5' slopes are comparable. For each sample and gene, fit the linear regression of mean-corrected log-coverage on scaled base position using decay.rate.hy(Data, dist2end, robust) to obtain per-sample decay rates. Assemble the rates into a samples-by-genes decay-rate matrix, then run hierarchical clustering on samples to isolate the cluster of severely degraded cases. Exclude that cluster from downstream shape change detection; in the 522-sample TCGA HNSC cohort this filter removed exactly 70 severely degraded cases before SCISSOR's global/local shape change analysis.

## Related tools

- **SCISSOR** (R package providing log transformation, base-level normalization, and the framework in which degradation-based sample exclusion is performed before shape change detection) — https://github.com/hyochoi/SCISSOR
- **decay.rate.hy** (Exported SCISSOR function fitting per-sample linear regression of mean-corrected log-coverage on scaled base position to quantify RNA decay rate per gene) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); data(TOY_data); rates <- apply(gene_pileups, function(g) decay.rate.hy(log2(g$coverage - shift), g$dist2end, robust=TRUE)); hc <- hclust(dist(rates)); excluded <- ids[hc$cluster == degraded_cluster]
```

## Evaluation signals

- The decay-rate matrix has one row per sample and one column per gene, with finite decay-rate values for each sample-gene regression.
- Hierarchical clustering on the decay-rate matrix yields a distinct cluster of severely degraded samples separable from the main cohort.
- For the 522-sample TCGA HNSC cohort, the excluded sample ID list must contain exactly 70 sample IDs, reproducing the reported count.
- Remaining cohort size matches the analysis set (e.g., 452 samples for the TP53 shape change analysis).
- Degradation is measured across genes, not per single gene, so exclusion should reflect degradation in a large fraction of genes per sample.

## Limitations

- SCISSOR tuning parameters, including log normalization and a normality cutoff value, likely require optimization when a significantly different wet lab RNA-seq protocol is used.
- Inability to detect residual RNA degradation does not exclude its existence; mild degradation may escape the filter.
- Variable fragment length in even high-quality TCGA RNA may introduce artifactual signals that the degradation filter does not address.

## Evidence

- [results] 70 severely degraded cases out of 522 HNSC tumor patients excluded from downstream analysis: "identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis"
- [results] Degradation measured across genes for the HNSC cohort: "For 522 HNSC tumor patients, we measured the amount of degradation across genes"
- [discussion] Rationale: degraded RNA causes spurious outliers: "We addressed this by empirically removing samples with evidence of degradation."
- [readme] SCISSOR package installation and toy data loading: "install_github("hyochoi/SCISSOR")
library(SCISSOR)"
- [results] SCISSOR preprocessing precedes shape change detection: "Before the main shape change detection procedure, normalization and variance stabilization are performed"
