---
name: coverage-normalization-and-variance-stabilization
description: Use when base-level pileup coverage matrices from RNA-seq BAM files must
  be transformed and normalized before cohort-level outlier or structural-variation
  analysis, e.g. as the input step to Scissor() shape change detection.
when_to_use_negative:
- Samples show severe RNA degradation across a large fraction of genes (e.g. the 70
  excluded of 522 HNSC cases) — remove them first, as degradation contaminates normalization
- The gene has many non-expressed samples (genes like the 5802 ~28% filtered out)
  — per-gene expressed/not-expressed filtering should precede normalization
- The goal is gene- or transcript-level differential expression from quantified counts
  (e.g. RSEM values) rather than base-resolution coverage shape analysis
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SCISSOR
  role: Package whose Scissor() function performs the automatic log-shift transformation
    and base-level normalization of pileup data before shape change detection
  repo: https://github.com/hyochoi/SCISSOR
- name: read_BAM
  role: Reads base-level pileup coverage from TCGA HNSC BAM files with samples as
    columns and coordinates as rows (outputType='part_intron'), producing the input
    matrix to normalize
  repo: https://github.com/hyochoi/SCISSOR
- name: build_gaf
  role: Builds the gene annotation (e.g. Gene='TP53' from Homo_sapiens.GRCh37.87.gtf)
    from which get_Ranges derives analysis ranges
  repo: https://github.com/hyochoi/SCISSOR
- name: get_Ranges
  role: Derives the part_intron genomic ranges used to restrict and normalize the
    pileup data
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:d14a63ccf4caacf7a35e05bd78ca0f3600943c1c60a712e8abfd444985c58802
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  - build_gaf
  - get_Ranges
  - plot_ScissorOS
  - read_BAM
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# coverage-normalization-and-variance-stabilization

## Summary

SCISSOR's pre-processing step that log-transforms (with an automatically chosen log-shift parameter) and normalizes base-level RNA-seq coverage pileup data prior to shape change detection, stabilizing variance across samples so that outlying coverage shapes can be detected fairly.

## When to use

Use when base-level pileup coverage matrices from RNA-seq BAM files must be transformed and normalized before cohort-level outlier or structural-variation analysis, e.g. as the input step to Scissor() shape change detection.

## When NOT to use

- Samples show severe RNA degradation across a large fraction of genes (e.g. the 70 excluded of 522 HNSC cases) — remove them first, as degradation contaminates normalization
- The gene has many non-expressed samples (genes like the 5802 ~28% filtered out) — per-gene expressed/not-expressed filtering should precede normalization
- The goal is gene- or transcript-level differential expression from quantified counts (e.g. RSEM values) rather than base-resolution coverage shape analysis

## Inputs

- Base-level pileup coverage matrix from RNA-seq BAM files (samples as columns, genomic coordinates as rows), produced by read_BAM/build_pileup
- Genomic ranges for the target gene from get_Ranges with outputType='part_intron'
- Gene annotation built by build_gaf from a GTF file (e.g. Homo_sapiens.GRCh37.87.gtf)

## Outputs

- Log-transformed (auto-selected shift) and base-level normalized coverage matrix ready for Scissor() shape change detection
- ScissorOutput object containing detected shape changes (ScissorOutput$SC), global outlyingness scores (ScissorOutput$GSC$OS), and local outlyingness scores (ScissorOutput$LSC$OS) with cutoff values

## How to apply

First build genomic ranges from the GTF annotation (e.g. Homo_sapiens.GRCh37.87.gtf) with build_gaf and get_Ranges (outputType='part_intron' to omit large intronic portions), then read base-level coverage with read_BAM keeping samples as columns and genomic coordinates as rows. The Scissor() all-in-one function then performs the normalization: it automatically chooses the log-shift parameter for logarithmic transformation of the pileup matrix and applies base-level normalization before any shape change detection. Normalization and variance stabilization are deliberately performed as pre-processing before the two-step global (GSC) and local (LSC) detection, so that outlyingness scores are computed in a comparable scale across samples. Degraded samples (e.g. the 70 severely degraded cases out of 522 TCGA HNSC) and lowly/non-expressed genes (5802 genes, ~28%) should be excluded beforehand. Correct application is judged by stable, comparable coverage distributions across samples feeding the GSC/LSC outlyingness scores and by recovering expected outlier counts (e.g. 26 GSCs and 24 LSCs at TP53 in 452 HNSC samples at cutoff 1e-04).

## Related tools

- **SCISSOR** (Package whose Scissor() function performs the automatic log-shift transformation and base-level normalization of pileup data before shape change detection) — https://github.com/hyochoi/SCISSOR
- **read_BAM** (Reads base-level pileup coverage from TCGA HNSC BAM files with samples as columns and coordinates as rows (outputType='part_intron'), producing the input matrix to normalize) — https://github.com/hyochoi/SCISSOR
- **build_gaf** (Builds the gene annotation (e.g. Gene='TP53' from Homo_sapiens.GRCh37.87.gtf) from which get_Ranges derives analysis ranges) — https://github.com/hyochoi/SCISSOR
- **get_Ranges** (Derives the part_intron genomic ranges used to restrict and normalize the pileup data) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); regions = build_gaf(Gene="TP53", GTF.file="./Homo_sapiens.GRCh37.87.gtf"); Ranges = get_Ranges(regions, outputType="part_intron"); ScissorOutput = Scissor(pileupData=pileupData, Ranges=Ranges)
```

## Evaluation signals

- Recovers the reported result: in 452 non-degraded TCGA HNSC samples, 26 GSCs and 24 LSCs detected in TP53 at significance cutoff 1e-04
- Scissor() reports automatic selection of the log-shift parameter rather than a user-set hardcoded shift
- Normalized coverage distributions are comparable across samples before GSC/LSC outlyingness scores (ScissorOutput$GSC$OS, ScissorOutput$LSC$OS) are computed
- Upstream filters were applied: ~70 severely degraded samples excluded from 522 HNSC and lowly expressed genes removed by median/MAD percentile cutoffs
- pileupData matrix has samples as columns and genomic coordinates as rows with outputType='part_intron'

## Limitations

- SCISSOR tuning parameters including log normalization and a normality cutoff value will likely require optimization when a significantly different wet lab RNA-seq protocol is used
- Omission of large intronic portions (part_intron ranges) may miss important alternate splice sites and cryptic exons
- Variable fragment length and GC content effects may introduce artifactual signals detectable after normalization

## Evidence

- [abstract] SCISSOR is a series of procedures for transforming and normalizing base-level RNA sequencing coverage data: "SCISSOR, is a series of procedures for transforming and normalizing base-level RNA sequencing coverage data"
- [methods] log transformation with automatic shift: "logarithmic transformation by automatically choosing the log shift parameter"
- [results] normalization before detection: "Before the main shape change detection procedure, normalization and variance stabilization are performed"
- [results] degraded samples excluded: "identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis"
- [methods] Scissor inputs: "We first obtain pileup data matrix using `build_pileup` and genomic ranges using `get_Ranges`"
