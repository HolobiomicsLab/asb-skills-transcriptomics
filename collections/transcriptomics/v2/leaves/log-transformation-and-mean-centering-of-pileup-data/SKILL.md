---
name: log-transformation-and-mean-centering-of-pileup-data
description: Use when you have base-level RNA-seq coverage (pileup) matrices from
  SCISSOR's build_pileup and need comparable, variance-stabilized per-gene coverage
  before shape change detection or decay-rate estimation.
when_to_use_negative:
- When the input is already normalized gene-level expression (e.g. normalized RSEM
  values used only for percentile-cutoff gene filtering), not base-level pileup coverage
- When the analysis goal is not shape/structural-change detection but e.g. read alignment
  or transcript quantification, where SCISSOR's coverage transform is irrelevant
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCISSOR
  role: R package providing build_pileup/get_Ranges for the pileup input and the log-transformation/normalization
    procedures preceding shape change detection
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:be4ba2593419b8370f627f86aa6667b26aa2fa0c8c34b3e0a1251c88c3af14f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# log-transformation-and-mean-centering-of-pileup-data

## Summary

Pre-processing step from the SCISSOR pipeline that variance-stabilizes base-level RNA-seq pileup coverage by logarithmic transformation (with automatic log-shift selection) and mean-centering per gene, so that sample-specific coverage shapes and 3'-to-5' decay slopes become comparable across samples.

## When to use

Use when you have base-level RNA-seq coverage (pileup) matrices from SCISSOR's build_pileup and need comparable, variance-stabilized per-gene coverage before shape change detection or decay-rate estimation.

## When NOT to use

- When the input is already normalized gene-level expression (e.g. normalized RSEM values used only for percentile-cutoff gene filtering), not base-level pileup coverage
- When the analysis goal is not shape/structural-change detection but e.g. read alignment or transcript quantification, where SCISSOR's coverage transform is irrelevant

## Inputs

- Base-level pileup coverage matrix produced by SCISSOR's build_pileup
- Genomic ranges from SCISSOR's get_Ranges (e.g. built from Homo_sapiens.GRCh37.87.gtf via build_gaf)
- SCISSOR toy coverage data loaded via data(TOY_data)

## Outputs

- Log-transformed, per-gene mean-centered coverage matrix (samples x bases) ready for Scissor shape-change analysis
- Mean-corrected log-coverage per sample/gene used to fit 3'-to-5' decay rates (decay_rate_matrix.csv inputs)

## How to apply

After building the pileup matrix with SCISSOR's build_pileup and get_Ranges, apply logarithmic transformation choosing the log shift parameter automatically so variance is stabilized across bases. Then mean-center the log-transformed coverage per gene across samples so that each sample's profile is expressed as a deviation from the gene-wide average shape. This normalization and variance stabilization must be performed before the main shape change detection procedure (GSC/LSC) or before fitting per-sample linear regressions of mean-corrected log-coverage on scaled base position (e.g. via decay.rate.hy) for RNA-decay-rate computation. The rationale is transcript-independent normalization: raw pileup depth varies with library size and positional biases, and centering removes the gene-level mean so sample-specific slopes and residual shapes are comparable.

## Related tools

- **SCISSOR** (R package providing build_pileup/get_Ranges for the pileup input and the log-transformation/normalization procedures preceding shape change detection) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(devtools); install_github("hyochoi/SCISSOR"); library(SCISSOR); data(TOY_data)
```

## Evaluation signals

- The log shift parameter must be selected automatically during the logarithmic transformation step (SCISSOR's own criterion), not fixed arbitrarily
- After mean-centering, each gene's mean log-coverage across samples is zero, so sample profiles are deviations from the gene-wide shape
- Downstream per-sample 3'-to-5' slopes fitted on the mean-corrected log-coverage must be comparable across samples; on TCGA HNSC this enabled identifying exactly 70 severely degraded cases out of 522
- The transform must precede GSC/LSC shape change detection, mirroring the article's pre-processing order (normalization and variance stabilization before the main detection procedure)

## Limitations

- SCISSOR's tuning parameters, including log normalization and a normality cutoff value, may require optimization when a significantly different wet-lab RNA-seq protocol is used
- Residual RNA degradation or variable fragment length may still introduce artifactual signals even after normalization; samples with degradation evidence must be empirically removed
- The transform is designed for base-level coverage; it does not itself filter lowly expressed genes (percentile cutoffs on normalized RSEM values are a separate filtering step)

## Evidence

- [methods] logarithmic transformation by automatically choosing the log shift parameter: "logarithmic transformation by automatically choosing the log shift parameter"
- [results] Normalisation before shape change detection: "Before the main shape change detection procedure, normalization and variance stabilization are performed"
- [methods] Pileup input construction: "We first obtain pileup data matrix using `build_pileup` and genomic ranges using `get_Ranges`"
- [readme] Package installation from README: "install_github("hyochoi/SCISSOR")"
