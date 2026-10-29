---
name: rna-seq-coverage-decay-quantification
description: Use when base-level pileup coverage from an RNA-seq cohort must be screened
  for degraded RNA — e.g. samples with steep 3'-to-5' coverage slopes that would masquerade
  as spurious shape-change outliers in downstream analysis.
when_to_use_negative:
- The task is gene-level expression quantification or differential expression — this
  skill operates on base-level pileup coverage shapes, not normalized RSEM counts
- You are detecting global/local shape changes (GSC/LSC) in individual genes; degradation
  filtering is a pre-processing step that should be run BEFORE shape change detection
- Genes with many non-expressed samples have not been filtered — the article excluded
  5802 genes (~28%) defined as 'not expressed' before degradation analysis, so decay
  rates on such genes would be unreliable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCISSOR
  role: R package providing base-level pileup construction (`build_pileup`, `get_Ranges`)
    and the exported `decay.rate.hy` function used to fit per-sample, per-gene RNA
    decay rates from log-transformed, mean-centered coverage
  repo: https://github.com/hyochoi/SCISSOR
- name: decay.rate.hy
  role: Exported SCISSOR function that regresses mean-corrected log-coverage on scaled
    base position to produce the per-sample decay rate used for degradation calling
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:46a9ad300cd5cf52aa558fa17d408b43ef5ecd9d10fb7968c1bd6bf28e8c49d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  - decay.rate.hy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rna-seq-coverage-decay-quantification

## Summary

Quantifies per-sample RNA degradation from base-level RNA-seq pileup coverage by fitting sample-wise 3'-to-5' log-coverage decay rates across genes with SCISSOR's `decay.rate.hy`, then clustering samples to flag severely degraded cases (70 of 522 in the TCGA HNSC cohort) for exclusion.

## When to use

Use when base-level pileup coverage from an RNA-seq cohort must be screened for degraded RNA — e.g. samples with steep 3'-to-5' coverage slopes that would masquerade as spurious shape-change outliers in downstream analysis.

## When NOT to use

- The task is gene-level expression quantification or differential expression — this skill operates on base-level pileup coverage shapes, not normalized RSEM counts
- You are detecting global/local shape changes (GSC/LSC) in individual genes; degradation filtering is a pre-processing step that should be run BEFORE shape change detection
- Genes with many non-expressed samples have not been filtered — the article excluded 5802 genes (~28%) defined as 'not expressed' before degradation analysis, so decay rates on such genes would be unreliable

## Inputs

- Base-level RNA-seq pileup coverage matrix (built via SCISSOR `build_pileup` with genomic ranges from `get_Ranges`)
- Scaled base positions / distance-to-end coordinates per gene (dist2end argument)
- Genomic annotation for gene regions (e.g. Homo_sapiens.GRCh37.87.gtf via `build_gaf`)

## Outputs

- Samples-by-genes decay-rate matrix (e.g. decay_rate_matrix.csv)
- List of severely degraded sample IDs excluded from downstream analysis (e.g. excluded_sample_ids.txt; expected to contain exactly 70 IDs for the 522-sample TCGA HNSC cohort)

## How to apply

Load base-level coverage data with SCISSOR (e.g. `data(TOY_data)` after `library(SCISSOR)`), then log-transform the pileup coverage and mean-center it per gene so that sample-specific 3'-to-5' slopes are comparable across samples. For every sample and gene, fit a robust linear regression of the mean-corrected log-coverage on scaled base position using the exported function `decay.rate.hy(Data, dist2end, robust)` to obtain per-sample decay rates. Assemble the per-sample rates into a samples-by-genes decay-rate matrix (e.g. written to decay_rate_matrix.csv). Run hierarchical clustering on the samples over this matrix, identify the cluster of severely degraded cases, and extract their sample IDs (e.g. to excluded_sample_ids.txt). In the reference application, applying this measurement across genes to 522 TCGA HNSC tumor patients identified 70 severely degraded cases that were excluded before shape change detection.

## Related tools

- **SCISSOR** (R package providing base-level pileup construction (`build_pileup`, `get_Ranges`) and the exported `decay.rate.hy` function used to fit per-sample, per-gene RNA decay rates from log-transformed, mean-centered coverage) — https://github.com/hyochoi/SCISSOR
- **decay.rate.hy** (Exported SCISSOR function that regresses mean-corrected log-coverage on scaled base position to produce the per-sample decay rate used for degradation calling) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); data(TOY_data); rates <- t(apply(Data, 1, function(x) decay.rate.hy(x, dist2end, robust=TRUE)))
```

## Evaluation signals

- The excluded sample ID list must contain exactly 70 sample IDs, reproducing the reported count of severely degraded cases from the 522-sample TCGA HNSC cohort
- The decay-rate output must be a samples-by-genes matrix (522 samples in the reference cohort) with one rate per sample-gene pair
- Robust regression fitting must succeed per sample-gene pair with `decay.rate.hy(Data, dist2end, robust)`; check for non-finite or missing rates on excluded low-expression genes
- Hierarchical clustering on the decay-rate matrix must yield a distinct cluster of severely degraded cases whose members were empirically removed before the downstream shape change analysis (452 retained samples for TP53 analysis)

## Limitations

- SCISSOR tuning parameters including log normalization and a normality cutoff value will likely require optimization when a significantly different wet lab RNA-seq protocol is used
- Inability to detect residual RNA degradation does not exclude the possibility of its existence
- Variable fragment length in high-quality TCGA RNA may introduce artifactual signals, so degradation-filtered samples can still show shape artifacts

## Evidence

- [results] identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis: "identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis"
- [results] degradation measured across genes for 522 HNSC tumor patients: "For 522 HNSC tumor patients, we measured the amount of degradation across genes"
- [methods] log shift parameter choice in SCISSOR preprocessing: "logarithmic transformation by automatically choosing the log shift parameter"
- [results] normalization and variance stabilization precede shape change detection: "Before the main shape change detection procedure, normalization and variance stabilization are performed"
- [readme] SCISSOR installation from GitHub: "install_github("hyochoi/SCISSOR")
library(SCISSOR)"
