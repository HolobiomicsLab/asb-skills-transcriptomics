---
name: base-level-pileup-processing
description: Use when RNA-seq alignment files (BAM) for a sample cohort must be converted
  to a transcript-independent, base-level coverage matrix for downstream shape-change
  or structural-variation analysis with SCISSOR.
when_to_use_negative:
- The goal is isoform-level differential expression from exon-collapsed counts (DEXSeq/SpliceTrap
  style) rather than base-resolution coverage shape analysis
- Only junction-spanning read information is needed (transcript-agnostic methods like
  MISO/rMATS suffice)
- A cohort is too small for outlier-shape statistics — the authors note ~30 samples
  is the smallest cohort considered successfully
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCISSOR
  role: R package providing build_pileup, get_Ranges, and the Scissor statistical
    framework for base-level shape-change detection
  repo: https://github.com/hyochoi/SCISSOR
- name: DEXSeq
  role: Contrasted exon-collapsing approach that SCISSOR's base-level pileup avoids
- name: MISO
  role: Contrasted junction-based method that ignores non-junction-spanning reads,
    motivating the intact pileup representation
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:3cbf6ee662cc99d0f2b47c1dc4b13b8b3eb78f4622e4065190f3e03070bcdccc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - DEXSeq
  - MISO
  - SCISSOR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# base-level pileup processing

## Summary

Obtain an uncompressed, base-resolution pileup matrix of RNA-seq coverage per gene (via SCISSOR's build_pileup and get_Ranges) so that structural alterations — splicing, intron retention, deletions, ATS/ATT — can be detected as coverage shape changes across a cohort.

## When to use

Use when RNA-seq alignment files (BAM) for a sample cohort must be converted to a transcript-independent, base-level coverage matrix for downstream shape-change or structural-variation analysis with SCISSOR.

## When NOT to use

- The goal is isoform-level differential expression from exon-collapsed counts (DEXSeq/SpliceTrap style) rather than base-resolution coverage shape analysis
- Only junction-spanning read information is needed (transcript-agnostic methods like MISO/rMATS suffice)
- A cohort is too small for outlier-shape statistics — the authors note ~30 samples is the smallest cohort considered successfully

## Inputs

- Aligned RNA-seq reads (BAM) for a sample cohort
- Gene annotation GTF file (e.g. Homo_sapiens.GRCh37.87.gtf)
- Genomic ranges per gene from get_Ranges/build_gaf

## Outputs

- Base-level pileup data matrix (bases x samples) per gene
- Genomic ranges objects consumed by the Scissor function

## How to apply

Install SCISSOR from GitHub (install_github("hyochoi/SCISSOR")) alongside its dependencies (Rsamtools, GenomicRanges, refGenome). Build genomic ranges for each gene with get_Ranges/build_gaf, supplying a GTF such as Homo_sapiens.GRCh37.87.gtf. Run build_pileup over the aligned reads to obtain a pileup data matrix in which every base of each gene is a column and every sample a row — this 'intact and uncompressed view' is what distinguishes SCISSOR from exon-collapsed methods like DEXSeq or junction-only methods like MISO. The pileup matrix plus the ranges then serve as inputs to the Scissor function, after normalization and variance stabilization (log transformation with an automatically chosen log shift). A toy dataset, TOY_data, ships with the package for validating the pipeline end to end.

## Related tools

- **SCISSOR** (R package providing build_pileup, get_Ranges, and the Scissor statistical framework for base-level shape-change detection) — https://github.com/hyochoi/SCISSOR
- **DEXSeq** (Contrasted exon-collapsing approach that SCISSOR's base-level pileup avoids)
- **MISO** (Contrasted junction-based method that ignores non-junction-spanning reads, motivating the intact pileup representation)

## Examples

```
devtools::install_github("hyochoi/SCISSOR"); library(SCISSOR); data(TOY_data); regions <- build_gaf(Gene=Gene, GTF.file="./Homo_sapiens.GRCh37.87.gtf")
```

## Evaluation signals

- The pileup matrix covers every base of each target gene (uncompressed), not exon-aggregated counts
- Genomic ranges from get_Ranges/build_gaf are non-empty and consistent with the supplied GTF (e.g. Homo_sapiens.GRCh37.87.gtf)
- The pileup matrix and ranges load successfully as inputs to the Scissor function without dimension mismatches
- The packaged TOY_data example runs through the pipeline, confirming the installation and dependencies (Rsamtools, GenomicRanges, refGenome, etc.)

## Limitations

- Large intronic portions are omitted, so important alternate splice sites and cryptic exons may be missed
- GSC detection loses information about the order of base pairs; LSC results depend on window-size choice
- Tuning parameters (log normalization, normality cutoff) may require optimization for substantially different RNA-seq protocols; variable fragment length and GC content can introduce artifactual shape signals

## Evidence

- [methods] build_pileup and get_Ranges generate the pileup matrix and ranges for Scissor: "We first obtain pileup data matrix using `build_pileup` and genomic ranges using `get_Ranges`"
- [methods] A toy dataset is available in the SCISSOR package: "A toy dataset is available in the `SCISSOR` package:

r
library(SCISSOR)
data(TOY_data)"
- [readme] SCISSOR uses an intact, uncompressed base-level pileup view enabling unbiased discovery: "`SCISSOR` considers a novel shape property of aligned short read data through a base-level pileup file. This intact and uncompressed view of RNA-seq profile enables the unbiased discovery of"
- [abstract] SCISSOR transforms and normalizes base-level coverage data transcript-independently: "a series of procedures for transforming and normalizing base-level RNA sequencing coverage data in a transcript independent manner"
