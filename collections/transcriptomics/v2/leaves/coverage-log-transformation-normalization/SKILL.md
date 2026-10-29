---
name: coverage-log-transformation-normalization
description: Use when raw base-level coverage pileup matrices (e.g. from build_pileup)
  must be variance-stabilized and normalized across a cohort prior to SCISSOR global/local
  shape change detection.
when_to_use_negative:
- The analysis goal is differential gene expression on already-aggregated count/feature
  tables (e.g. RSEM gene-level values) — this skill requires uncompressed base-level
  pileup coverage, and SCISSOR itself notes existing methods that collapse reads to
  exons lose the needed resolution
- The cohort is smaller than ~30 samples, the smallest cohort the authors report considering
  successfully
- The gene has many non-expressed samples that were not first excluded under SCISSOR's
  'not expressed' vs 'expressed' per-gene filter
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: SCISSOR
  role: R package whose Scissor() function performs the automatic log shift selection
    and base-level normalization as pre-processing before shape change detection
  repo: https://github.com/hyochoi/SCISSOR
- name: build_pileup
  role: produces the base-level pileup data matrix that the log transformation and
    normalization operate on
  repo: https://github.com/hyochoi/SCISSOR
- name: get_Ranges
  role: supplies the genomic ranges used together with the normalized pileup as inputs
    to Scissor
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:36ab73db4eb1490c3246a86e48642dc1d424e1d71eddb37d161ebf7300e68435
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  - build_pileup
  - get_Ranges
  - plot_ScissorOS
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# coverage-log-transformation-normalization

## Summary

Preprocessing step of the SCISSOR framework that transforms base-level RNA-seq coverage by a logarithmic transformation with an automatically chosen log shift parameter, followed by base-level normalization, to stabilize variance before shape-change detection.

## When to use

Use when raw base-level coverage pileup matrices (e.g. from build_pileup) must be variance-stabilized and normalized across a cohort prior to SCISSOR global/local shape change detection.

## When NOT to use

- The analysis goal is differential gene expression on already-aggregated count/feature tables (e.g. RSEM gene-level values) — this skill requires uncompressed base-level pileup coverage, and SCISSOR itself notes existing methods that collapse reads to exons lose the needed resolution
- The cohort is smaller than ~30 samples, the smallest cohort the authors report considering successfully
- The gene has many non-expressed samples that were not first excluded under SCISSOR's 'not expressed' vs 'expressed' per-gene filter

## Inputs

- base-level pileup matrix of RNA-seq coverage (e.g. produced by SCISSOR build_pileup from a whole_intron regions file)
- genomic ranges object from get_Ranges (e.g. built from Homo_sapiens.GRCh37.87.gtf via build_gaf or packaged regions)

## Outputs

- log-transformed, variance-stabilized, base-level normalized coverage data inside the Scissor output object
- per-sample outlyingness scores in ScissorOutput$GSC$OS (global) and ScissorOutput$LSC$OS (local) that reflect post-normalization shape deviations

## How to apply

First build the base-level pileup matrix with SCISSOR's build_pileup (e.g. inputType="whole_intron", outputType="part_intron") and derive genomic ranges with get_Ranges; these are the inputs the Scissor function expects. Rather than hand-picking a transformation, call Scissor(pileupData=..., Ranges=...), which automatically chooses the log shift parameter for the logarithmic transformation and then performs base-level normalization across samples. This step is transcript independent — it operates on the uncompressed base-level coverage without assuming a mechanism driving coverage variation — and is deliberately performed before the main shape change detection (GSC followed by LSC on residual space). The rationale is that log transformation with an auto-selected shift stabilizes coverage variance so that downstream outlyingness scores reflect shape, not depth. Note that the log normalization is a tuning parameter: for a significantly different wet-lab RNA-seq protocol it may require optimization rather than relying on the automatic choice.

## Related tools

- **SCISSOR** (R package whose Scissor() function performs the automatic log shift selection and base-level normalization as pre-processing before shape change detection) — https://github.com/hyochoi/SCISSOR
- **build_pileup** (produces the base-level pileup data matrix that the log transformation and normalization operate on) — https://github.com/hyochoi/SCISSOR
- **get_Ranges** (supplies the genomic ranges used together with the normalized pileup as inputs to Scissor) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); data(TOY_data); pileupData <- build_pileup(Pileup=toygene_coverage, regions=regions, inputType="whole_intron", outputType="part_intron"); ScissorOutput <- Scissor(pileupData=pileupData, Ranges=geneRanges)
```

## Evaluation signals

- Scissor() reports having automatically chosen a log shift parameter for the logarithmic transformation; no shift is hand-set by the user
- Normalization and variance stabilization are recorded as completed before the global shape change detection step runs (GSC then LSC on residual space)
- On the packaged TOY_data demo (n=260), the normalized pipeline flags exactly 8 shape-change outliers in ScissorOutput$SC
- Post-normalization outlyingness scores are populated in ScissorOutput$GSC$OS and ScissorOutput$LSC$OS with larger values indicating more severe deviation
- Normalization is transcript independent: results should not require a gene model beyond the ranges needed to define the analyzed region

## Limitations

- SCISSOR's tuning parameters, including log normalization and the normality cutoff value, likely require optimization when a significantly different wet-lab RNA-seq protocol is used
- The log shift parameter is chosen automatically, so unusual coverage distributions may need manual tuning; variable fragment length and GC effects can introduce artifactual shape signals downstream of normalization
- Minimum effective cohort size (~30 samples) was determined only empirically, without precise effect-size experiments

## Evidence

- [methods] logarithmic transformation by automatically choosing the log shift parameter: "* logarithmic transformation by automatically choosing the log shift parameter"
- [results] Before the main shape change detection procedure, normalization and variance stabilization are performed: "Before the main shape change detection procedure, normalization and variance stabilization are performed"
- [abstract] a series of procedures for transforming and normalizing base-level RNA sequencing coverage data in a transcript independent manner: "a series of procedures for transforming and normalizing base-level RNA sequencing coverage data in a transcript independent manner"
- [discussion] SCISSOR has tuning parameters including log normalization and a normality cutoff value which will likely require some optimization when a significantly different wet lab RNA-seq protocol is used.: "SCISSOR has tuning parameters including log normalization and a normality cutoff value which will likely require some optimization when a significantly different wet lab RNA-seq protocol is used."
