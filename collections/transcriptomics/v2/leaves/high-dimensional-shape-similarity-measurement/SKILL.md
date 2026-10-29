---
name: high-dimensional-shape-similarity-measurement
description: Use when base-level RNA-seq pileup matrices must be compared per sample
  against a gene mean shape reference, e.g. to derive per-gene on/off statistics or
  exclude genes with many off samples before shape-change analysis.
when_to_use_negative:
- Input data are gene-level summarized expression (e.g., normalized RSEM values) rather
  than base-level pileup coverage — shape angles require the uncompressed base-resolution
  representation SCISSOR builds from pileup files
- The goal is junction-spanning or known-transcript isoform analysis better served
  by methods such as DEXSeq, SpliceTrap, MISO, or rMATS rather than shape-change screening
- The cohort is too small for unsupervised shape analysis — the authors note ~30 samples
  is the smallest cohort considered successfully
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: SCISSOR
  role: Provides build_pileup/get_Ranges to construct base-level pileup inputs, angle.hy
    to compute high-dimensional angles between sample vectors and the gene mean, and
    get_exon_offstat/get_offstat to aggregate on/off statistics per gene
  repo: https://github.com/hyochoi/SCISSOR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20593-3
    title: 'SCISSOR: a framework for identifying structural changes in RNA transcripts'
  source_content_hash: sha256:6ffdcbc735cdcc9b0520711c25741b5191fbe8e009f52ebcbd5b5a2296c64130
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hyochoi/SCISSOR
  tools:
  - SCISSOR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# high-dimensional-shape-similarity-measurement

## Summary

Quantifies per-sample RNA coverage shape similarity to a gene-level mean vector by computing high-dimensional angles (SCISSOR's angle.hy), which underpins the on/off classification used to flag genes with many non-expressed samples in the 522-sample TCGA HNSC cohort.

## When to use

Use when base-level RNA-seq pileup matrices must be compared per sample against a gene mean shape reference, e.g. to derive per-gene on/off statistics or exclude genes with many off samples before shape-change analysis.

## When NOT to use

- Input data are gene-level summarized expression (e.g., normalized RSEM values) rather than base-level pileup coverage — shape angles require the uncompressed base-resolution representation SCISSOR builds from pileup files
- The goal is junction-spanning or known-transcript isoform analysis better served by methods such as DEXSeq, SpliceTrap, MISO, or rMATS rather than shape-change screening
- The cohort is too small for unsupervised shape analysis — the authors note ~30 samples is the smallest cohort considered successfully

## Inputs

- Base-level RNA-seq coverage pileup matrix for a cohort (e.g., built via SCISSOR build_pileup over gene regions from get_Ranges and a GTF such as Homo_sapiens.GRCh37.87.gtf)
- Genomic ranges / gene annotation (GAF) defining per-gene base intervals
- Cohort sample set after removal of degraded cases (e.g., 452 of 522 TCGA HNSC samples)

## Outputs

- Per-gene table of off-sample fractions with excluded-gene flags (written to onoff_gene_fractions.csv)
- Per sample-gene angle statistics used as shape-similarity measures and on/off classifications
- List of excluded genes with >20% off samples (5802 genes, ~28%, in the HNSC application)

## How to apply

Load the genome-wide base-level pileup matrix (built with SCISSOR's build_pileup and get_Ranges for genomic ranges) after excluding severely degraded samples — the HNSC analysis removed 70 degraded cases out of 522. For each gene, compute the mean coverage vector across all samples to serve as the gene-shape reference, then use SCISSOR's angle computation (angle.hy) to measure the high-dimensional angle between each individual sample vector and the gene mean as the shape-similarity statistic. Apply the angle-based on/off classification to each sample-gene pair and aggregate with get_exon_offstat / get_offstat to compute the fraction of off samples per gene. Flag genes whose off-sample fraction exceeds 20% for exclusion from downstream analysis; in the TCGA HNSC application this reproduced the reported 5802 excluded genes (~28%).

## Related tools

- **SCISSOR** (Provides build_pileup/get_Ranges to construct base-level pileup inputs, angle.hy to compute high-dimensional angles between sample vectors and the gene mean, and get_exon_offstat/get_offstat to aggregate on/off statistics per gene) — https://github.com/hyochoi/SCISSOR

## Examples

```
install_github("hyochoi/SCISSOR"); library(SCISSOR); data(TOY_data)
```

## Evaluation signals

- The number of genes flagged as excluded (off-sample fraction >20%) reproduces the reported 5802 genes (~28%) on the 522-sample TCGA HNSC cohort with 70 degraded cases removed
- The per-gene off-fraction table, including the excluded-gene list, is written to onoff_gene_fractions.csv
- Each sample-gene pair receives a defined angle statistic and a binary on/off classification derived from the angle to the gene mean vector
- Genes excluded here are not carried into subsequent global/local shape change (GSC/LSC) detection steps

## Limitations

- SCISSOR tuning parameters (log normalization, normality cutoff) require optimization when a significantly different wet-lab RNA-seq protocol is used
- GC content effects and variable fragment length may introduce artifactual signals that distort shape statistics
- The >20% off-sample exclusion threshold and the minimum viable cohort size (~30 samples) were determined empirically rather than from effect-size experiments

## Evidence

- [results] Per gene definition of 'not expressed' versus 'expressed': "A per gene definition of “not expressed” versus “expressed” was developed for the purposes of excluding genes with many non-expressed samples"
- [results] 5802 genes excluded: "we identified 5802 genes (~28%) and they were excluded from the downstream analysis"
- [results] Residual after gene-shape model subtraction: "determining the residual per-base expression for each sample after subtraction of a model representing the shape of that gene across all samples"
- [readme] SCISSOR purpose: "`SCISSOR` aims for unsupervised screening of a range of structural alterations in RNA-seq data."
- [results] 70 degraded cases excluded: "identified 70 cases that were severely degraded in a large fraction of genes and excluded from the downstream analysis"
