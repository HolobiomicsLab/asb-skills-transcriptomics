---
name: tcga-cohort-shape-change-screening
description: Use when screening a cohort of base-level RNA-seq pileup data (e.g.,
  TCGA BAM files) for samples with aberrant transcript shapes — splicing changes,
  intron retention, deletions — in a specific gene.
when_to_use_negative:
- The cohort contains many non-expressed samples for the target gene — SCISSOR's per-gene
  'not expressed' filter excluded 5802 genes (~28%) from downstream analysis
- Cohort is very small — empirically ~30 samples is the smallest cohort considered
  successfully
- The question requires junction-spanning or transcript-model-based isoform quantification
  rather than unbiased base-level coverage shape analysis
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: SCISSOR
  role: R package implementing the full two-step shape change detection pipeline (Scissor
    function)
  repo: https://github.com/hyochoi/SCISSOR
- name: build_gaf
  role: Builds gene annotation from the GTF file
  repo: https://github.com/hyochoi/SCISSOR
- name: get_Ranges
  role: Derives genomic ranges (outputType="part_intron") used as input for the Scissor
    function
  repo: https://github.com/hyochoi/SCISSOR
- name: read_BAM
  role: Reads base-level pileup coverage from cohort BAM files
  repo: https://github.com/hyochoi/SCISSOR
- name: plot_ScissorOS
  role: Visualizes global and local outlyingness scores (colmat=NULL, textSC=TRUE)
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:74d121d4cd9c7fb9acf7b62ba6f0ae2464c4d84753c49210d59479072dbc441c
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# tcga-cohort-shape-change-screening

## Summary

Two-step SCISSOR screening of base-level RNA-seq coverage across a tumor cohort (e.g., 522 TCGA HNSC) to detect global and local shape changes in a target gene, at a significance cutoff of 1e-04.

## When to use

Use when screening a cohort of base-level RNA-seq pileup data (e.g., TCGA BAM files) for samples with aberrant transcript shapes — splicing changes, intron retention, deletions — in a specific gene.

## When NOT to use

- The cohort contains many non-expressed samples for the target gene — SCISSOR's per-gene 'not expressed' filter excluded 5802 genes (~28%) from downstream analysis
- Cohort is very small — empirically ~30 samples is the smallest cohort considered successfully
- The question requires junction-spanning or transcript-model-based isoform quantification rather than unbiased base-level coverage shape analysis

## Inputs

- TCGA cohort base-level RNA-seq BAM files (e.g., 522 HNSC tumors)
- Gene annotation GTF file (Homo_sapiens.GRCh37.87.gtf)
- Target gene name (e.g., TP53)
- Significance cutoff for shape change calls (e.g., 1e-04)

## Outputs

- ScissorOutput object with $SC (identified shape changes), $GSC$OS and $LSC$OS (outlyingness scores and cutoffs)
- Counts of global shape changes (GSCs) and local shape changes (LSCs) per gene
- Outlier sample IDs per procedure (e.g., tp53_scissor_outliers.csv)
- Outlyingness score plots via plot_ScissorOS

## How to apply

Build the gene annotation with SCISSOR's build_gaf from a GTF (e.g., Homo_sapiens.GRCh37.87.gtf) and derive genomic ranges via get_Ranges(outputType="part_intron"). Read base-level pileup coverage from BAM files with read_BAM, keeping samples as columns and genomic coordinates as rows. Run the all-in-one Scissor(pileupData, Ranges) function, which automatically selects the log-shift parameter for log-transformation, performs base-level normalization, detects global shape changes (GSCs) by exploring all possible low-dimensional directions, then detects local shape changes (LSCs) in the residual space. Extract hits from ScissorOutput$SC with outlyingness scores (ScissorOutput$GSC$OS, ScissorOutput$LSC$OS) and call outliers exceeding the significance cutoff (e.g., 1e-04). Exclude degraded samples first (the HNSC analysis removed 70 severely degraded cases from 522, leaving 452).

## Related tools

- **SCISSOR** (R package implementing the full two-step shape change detection pipeline (Scissor function)) — https://github.com/hyochoi/SCISSOR
- **build_gaf** (Builds gene annotation from the GTF file) — https://github.com/hyochoi/SCISSOR
- **get_Ranges** (Derives genomic ranges (outputType="part_intron") used as input for the Scissor function) — https://github.com/hyochoi/SCISSOR
- **read_BAM** (Reads base-level pileup coverage from cohort BAM files) — https://github.com/hyochoi/SCISSOR
- **plot_ScissorOS** (Visualizes global and local outlyingness scores (colmat=NULL, textSC=TRUE)) — https://github.com/hyochoi/SCISSOR

## Examples

```
library(SCISSOR); regions <- build_gaf(Gene="TP53",GTF.file="./Homo_sapiens.GRCh37.87.gtf"); geneRanges <- get_Ranges(gaf=regions,outputType="part_intron"); ScissorOutput <- Scissor(pileupData=pileupData,Ranges=geneRanges)
```

## Evaluation signals

- On 452 non-degraded TCGA HNSC samples, the TP53 analysis should recover 26 global shape changes (GSCs) and 24 local shape changes (LSCs) at cutoff 1e-04
- Approximately 82% of identified TP53 shape changes should coincide with known mutations, most commonly at splice sites
- ScissorOutput$GSC$OS and $LSC$OS should contain outlyingness scores with corresponding cutoff values for every analyzed sample
- pileupData matrix should have samples as columns and genomic coordinates as rows over the part_intron ranges of the target gene
- Only ~59% of splice site mutations annotated in the public HNSC repository should be expected to show a significant shape change — a lower overlap does not indicate failure

## Limitations

- 70 severely degraded samples out of 522 HNSC must be excluded; residual undetected degradation may still exist
- Tuning parameters (log normalization shift, normality cutoff) may require optimization for RNA-seq protocols different from TCGA
- The GSC detection algorithm loses information about the order of base pairs
- LSC detection results may change with different window sizes
- Omitting large intronic portions (part_intron ranges) may miss alternate splice sites and cryptic exons

## Evidence

- [results] Two-step GSC/LSC procedure: "SCISSOR implements a two-step procedure, global and local, taking advantage of low-dimensional transformations and sparsity"
- [results] TP53 counts validation: "In 452 samples, SCISSOR identified 26 GSCs and 24 LSCs capturing a rich set of known (82%) and previously unrecognized (18%) genetic aberrations in TP53"
- [results] Degraded sample filtering: "identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis"
- [methods] Method functions: "We first obtain pileup data matrix using `build_pileup` and genomic ranges using `get_Ranges` that will be used as inputs for the function `Scissor`"
- [abstract] Purpose: "Shape changes in selecting sample outliers in RNA-seq, SCISSOR, is a series of procedures for transforming and normalizing base-level RNA sequencing coverage data"
- [readme] Installation: "install_github("hyochoi/SCISSOR")
library(SCISSOR)"
