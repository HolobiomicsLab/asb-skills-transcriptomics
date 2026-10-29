---
name: projection-pursuit-outlier-scoring
description: Use when a cohort of base-level RNA-seq pileup matrices at one gene must
  be screened unsupervised for samples with anomalous coverage shapes (splicing, intron
  retention, deletions, ATS/ATT) rather than differential expression.
when_to_use_negative:
- The question is differential expression of known transcripts or junction-level isoform
  quantification — methods like DEXSeq, SpliceTrap, MISO, rMATS, or DiffSplice target
  that; SCISSOR targets base-level shape anomalies.
- The cohort is small (fewer than ~30 samples); empirically 30 samples is the smallest
  cohort considered successfully to date.
- Input is already a compressed expression/feature count table — SCISSOR requires
  intact, uncompressed base-level pileup data to retain shape information.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3170
tools:
- name: SCISSOR
  role: R package implementing transformation, normalization, and the two-step global/local
    shape change detection with outlyingness scoring (Scissor function)
  repo: https://github.com/hyochoi/SCISSOR
- name: build_pileup
  role: Constructs the base-level pileup data matrix used as input to Scissor
  repo: https://github.com/hyochoi/SCISSOR
- name: get_Ranges
  role: Derives the analysis genomic ranges passed as Ranges to Scissor
  repo: https://github.com/hyochoi/SCISSOR
- name: plot_ScissorOS
  role: Visualizes global and local outlyingness score distributions to confirm flagged
    outlier indices
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:421c44739aaa82cc40841a53ee593bb02ddfda4870bba2b7a83674621cb2a653
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  - build_gaf
  - build_pileup
  - get_Ranges
  - plot_ScissorOS
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# projection-pursuit-outlier-scoring

## Summary

SCISSOR's two-step statistical procedure that scores sample outliers in base-level RNA-seq coverage by exploring all possible low-dimensional projections (global shape change detection) and then searching the residual space after subtracting a gene-shape model (local shape change detection).

## When to use

Use when a cohort of base-level RNA-seq pileup matrices at one gene must be screened unsupervised for samples with anomalous coverage shapes (splicing, intron retention, deletions, ATS/ATT) rather than differential expression.

## When NOT to use

- The question is differential expression of known transcripts or junction-level isoform quantification — methods like DEXSeq, SpliceTrap, MISO, rMATS, or DiffSplice target that; SCISSOR targets base-level shape anomalies.
- The cohort is small (fewer than ~30 samples); empirically 30 samples is the smallest cohort considered successfully to date.
- Input is already a compressed expression/feature count table — SCISSOR requires intact, uncompressed base-level pileup data to retain shape information.

## Inputs

- base-level pileup coverage matrix (samples as columns, genomic coordinates as rows), e.g. from build_pileup or read_BAM with outputType='part_intron'
- genomic ranges for the analyzed gene from get_Ranges (outputType='part_intron')
- gene annotation built with build_gaf from a GTF file such as Homo_sapiens.GRCh37.87.gtf
- significance cutoff for shape change calling (e.g. 1e-04)

## Outputs

- ScissorOutput$SC: flagged shape-change sample IDs
- ScissorOutput$GSC$OS and ScissorOutput$LSC$OS: global and local outlyingness scores per sample
- Cutoff values for global and local procedures
- Optional visualization of score distributions via plot_ScissorOS and CSV of outlier IDs with scores

## How to apply

First transform and normalize base-level coverage: automatically choose the log shift parameter for logarithmic transformation, then perform base-level normalization. Run the two-step detection: global shape change (GSC) detection explores all possible low-dimensional directions in the normalized coverage space, assigning each sample an outlyingness score at each gene; local shape change (LSC) detection determines the residual per-base expression after subtracting a gene-shape model and explores the residual space, relying on sparsity. Flag samples as shape changes at a chosen significance cutoff (e.g. 1e-04), and rank them by the outlyingness scores in the GSC$OS and LSC$OS slots. Validate by recovering known counts — e.g. eight outliers in the TOY_data demo (n=260) and 26 GSCs plus 24 LSCs for TP53 across the 452 non-degraded TCGA HNSC samples.

## Related tools

- **SCISSOR** (R package implementing transformation, normalization, and the two-step global/local shape change detection with outlyingness scoring (Scissor function)) — https://github.com/hyochoi/SCISSOR
- **build_pileup** (Constructs the base-level pileup data matrix used as input to Scissor) — https://github.com/hyochoi/SCISSOR
- **get_Ranges** (Derives the analysis genomic ranges passed as Ranges to Scissor) — https://github.com/hyochoi/SCISSOR
- **plot_ScissorOS** (Visualizes global and local outlyingness score distributions to confirm flagged outlier indices) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); data(TOY_data); pileupData <- build_pileup(Pileup=toygene_coverage, regions=regions, inputType="whole_intron", outputType="part_intron"); geneRanges <- get_Ranges(Gene="TOY", regions=regions, outputType="part_intron"); ScissorOutput <- Scissor(pileupData=pileupData, Ranges=geneRanges)
```

## Evaluation signals

- TOY_data demo: exactly eight shape-change samples are flagged among n=260 cases (seven exon-skipping/intron-retention cases plus one small first-exon deletion)
- TP53 on TCGA HNSC: 26 global shape changes (GSCs) and 24 local shape changes (LSCs) among 452 non-degraded samples at cutoff 1e-04
- 82% of TP53 shape changes should coincide with known mutations, most commonly at splice sites — a concordance check against public mutation repositories
- plot_ScissorOS(object=ScissorOutput, textSC=TRUE) should visually separate flagged outliers from the score distribution
- Outlyingness scores (GSC$OS, LSC$OS) should exist for every sample and larger values must correspond to more severe deviation from other samples

## Limitations

- The GSC detection algorithm loses information about orders of base pairs.
- LSC detection results might differ with different window sizes.
- Tuning parameters (log normalization shift, normality cutoff value) will likely require optimization when a significantly different wet lab RNA-seq protocol is used.
- Omitting large intronic portions (part_intron) may miss important alternate splice sites and cryptic exons.
- Variable fragment length and GC-content-related experimental effects may introduce artifactual shape-change signals.

## Evidence

- [results] SCISSOR implements a two-step procedure, global and local, taking advantage of low-dimensional transformations and sparsity: "SCISSOR implements a two-step procedure, global and local, taking advantage of low-dimensional transformations and sparsity"
- [other] SCISSOR assigns each sample an outlyingness score at each gene, with larger values indicating more severe deviation from other samples: [evidence span withheld — anchor status: not_found]
- [results] determining the residual per-base expression for each sample after subtraction of a model representing the shape of that gene across all samples: "determining the residual per-base expression for each sample after subtraction of a model representing the shape of that gene across all samples"
- [results] In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
- [readme] SCISSOR considers a novel shape property of aligned short read data through a base-level pileup file.: "SCISSOR considers a novel shape property of aligned short read data through a base-level pileup file."
- [discussion] the GSC detection algorithm loses information about orders of base pairs: "the GSC detection algorithm loses information about orders of base pairs"
