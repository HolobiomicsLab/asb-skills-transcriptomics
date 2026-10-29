---
name: somatic-snv-variant-calling
description: Use when raw bulk DNA WES and paired scRNA-seq or bulk RNA-seq FASTQ
  data must be turned into mutation-by-sample read count matrices (Rs, Xs, Rb, Xb)
  of QC-filtered somatic SNVs for downstream clonal tree inference.
when_to_use_negative:
- Input is already a finalized read count matrix (Rs/Xs/Rb/Xb) or an R object like
  GBM10_postproc — go straight to Canopy2 inference instead
- Only germline heterozygous loci are needed for estimating gene activation/deactivation
  rates — use the separately provided germline variant calling pipeline scripts
- No matched bulk DNA WES data are available, since Rb/Xb matrices require bulk whole
  exome sequencing
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_2640
tools:
- name: BWA
  role: Aligns bulk DNA WES reads to produce sorted, indexed BAM files for somatic
    SNV calling
- name: STAR
  role: Aligns single-cell and bulk RNA-seq reads to produce BAMs for read count extraction
- name: Mutect2
  role: Calls candidate somatic SNVs from aligned bulk DNA and single-cell data; PASS
    calls from FilterMutectCalls are the first QC criterion
- name: SAMtools
  role: Runs mpileup in force-calling mode at finalized SNV loci to extract alternative
    and total read counts per cell/sample
- name: R
  role: Used to assemble and save the Rs, Xs, Rb, Xb matrices as read_count_matrices.rda
    for Canopy2
  repo: https://github.com/annweideman/canopy2
- name: Canopy2
  role: Downstream consumer of the Rs/Xs/Rb/Xb matrices produced by this pre-processing
    pipeline
  repo: https://github.com/annweideman/canopy2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:993ebfd19762287f06415a193f946cc616383ffcbc006b8b6d8ebff6ad28dfef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - BWA
  - Canopy2
  - Mutect2
  - R
  - SAMtools
  - STAR
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# somatic-snv-variant-calling

## Summary

A pre-processing pipeline that converts raw bulk DNA WES and scRNA-seq/bulk RNA-seq FASTQ data into the alternative and total read count matrices (Rs, Xs, Rb, Xb) of stringently QC-filtered somatic SNVs required as input to Canopy2 tumor phylogeny inference.

## When to use

Use when raw bulk DNA WES and paired scRNA-seq or bulk RNA-seq FASTQ data must be turned into mutation-by-sample read count matrices (Rs, Xs, Rb, Xb) of QC-filtered somatic SNVs for downstream clonal tree inference.

## When NOT to use

- Input is already a finalized read count matrix (Rs/Xs/Rb/Xb) or an R object like GBM10_postproc — go straight to Canopy2 inference instead
- Only germline heterozygous loci are needed for estimating gene activation/deactivation rates — use the separately provided germline variant calling pipeline scripts
- No matched bulk DNA WES data are available, since Rb/Xb matrices require bulk whole exome sequencing

## Inputs

- Raw FASTQ files: bulk DNA WES reads (e.g., SRP067248, EGAS00001001880)
- Raw FASTQ files: single-cell and bulk RNA-seq reads (e.g., GSE75688, EGAS00001001880)
- Sorted, indexed BAM files from BWA (bulk DNA) and STAR (RNA-seq) alignment
- Reference genome for alignment and variant calling

## Outputs

- Rs: M x N (mutation by single-cell) matrix of alternative read counts from scRNA-seq
- Xs: M x N (mutation by single-cell) matrix of total read counts (benign + mutated) from scRNA-seq
- Rb: M x T (mutation by bulk sample) matrix of alternative read counts from bulk DNA WES
- Xb: M x T matrix of total read counts from bulk DNA WES
- read_count_matrices.rda (R data file with identical mutation rows across single-cell and bulk matrices)
- Finalized somatic SNV callset passing pipeline QC

## How to apply

Download raw FASTQ data (e.g., breast cancer scRNA-seq and bulk RNA-seq from GEO accession GSE75688, breast cancer bulk DNA WES from SRA accession SRP067248, glioblastoma RNA-seq and bulk DNA WES from EGA accession EGAS00001001880) and run the deposited master shell scripts from the Zenodo repository (zenodo.org/record/7931384). Align bulk DNA WES reads with BWA and single-cell/bulk RNA-seq reads with STAR, producing sorted, indexed BAM files. Call candidate somatic SNVs with Mutect2 on the aligned bulk DNA and single-cell data, then apply the pipeline's QC filtering: retain only SNVs that are PASS from FilterMutectCalls, homozygous 0/0 in normal genotypes, have a single alternative allele, and have at least 20 total reads in normal samples. Run SAMtools mpileup in force-calling mode at the finalized SNV loci to extract alternative and total read counts for every single cell and bulk sample, then assemble the M x N matrices Rs and Xs (mutations by single-cells) and M x S matrices Rb and Xb (mutations by bulk samples) with identical mutation rows across single-cell and bulk data, saved as read_count_matrices.rda.

## Related tools

- **BWA** (Aligns bulk DNA WES reads to produce sorted, indexed BAM files for somatic SNV calling)
- **STAR** (Aligns single-cell and bulk RNA-seq reads to produce BAMs for read count extraction)
- **Mutect2** (Calls candidate somatic SNVs from aligned bulk DNA and single-cell data; PASS calls from FilterMutectCalls are the first QC criterion)
- **SAMtools** (Runs mpileup in force-calling mode at finalized SNV loci to extract alternative and total read counts per cell/sample)
- **R** (Used to assemble and save the Rs, Xs, Rb, Xb matrices as read_count_matrices.rda for Canopy2) — https://github.com/annweideman/canopy2
- **Canopy2** (Downstream consumer of the Rs/Xs/Rb/Xb matrices produced by this pre-processing pipeline) — https://github.com/annweideman/canopy2

## Evaluation signals

- All retained SNVs are PASS from FilterMutectCalls with homozygous 0/0 normal genotypes and a single alternative allele
- Every retained mutation has at least 20 total reads in normal samples
- Rs and Xs are M x N (mutations by single-cells) while Rb and Xb are M x S (mutations by bulk samples), with identical mutation rows across the single-cell and bulk matrices
- Output is a single read_count_matrices.rda file loadable in R as Canopy2 input
- Mutations and cells/samples excluded by QC (zero total counts, non-estimable burstiness) are removed before matrix assembly

## Limitations

- Stringent QC (PASS FilterMutectCalls, 0/0 normals, single alt allele, >=20 total reads in normals) can discard a large fraction of candidate SNVs, shrinking the finalized callset
- No software version was stated for Canopy2 or the pipeline tools in the article; users must track tool versions themselves
- Germline heterozygous loci used for estimating gene activation/deactivation rates require a separate germline variant calling pipeline

## Evidence

- [readme] matrix of alternative read counts from single-cell RNA sequencing data: "`Rs`:   $M \times N$ (mutation by single-cell) matrix of alternative read counts from single-cell RNA sequencing data."
- [discussion] stringent QC via pre-processing pipeline to obtain finalized SNV callset: "We suggest stringent QC of the bulk and single-cell read count data using our pre-processing pipeline (Fig. 6 in appendix) to obtain the finalized SNV callset."
- [methods] Zenodo repository hosts the sample pre-processing pipeline: "A sample pipeline that can be used to preprocess these data is located in our Zenodo repository at https://zenodo.org/record/7931384"
- [other] QC criteria applied during pre-processing: [evidence span withheld — anchor status: not_found]
