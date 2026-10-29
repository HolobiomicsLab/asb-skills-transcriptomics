---
name: kallisto-pseudoalignment-quantification
description: Use when raw RNA-Seq FASTQ reads (e.g. from GEO/SRA/ENA accessions) must
  be converted to transcript-level TPM abundances for downstream isoform-based analyses
  such as CanIsoNet.
when_to_use_negative:
- Input is already a gene-level or transcript-level expression matrix — quantification
  from raw reads is unnecessary
- Single-end reads or non-Ensembl transcript definitions are required, since this
  workflow specifies paired-end mode and an Ensembl GRCh37.p13 cDNA index for isoform
  compatibility with CanIsoNet
- Analysis targets PCAWG cancer samples, which in CanIsoNet derive from a different
  data source than the four RNA-Seq disease datasets
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Kallisto
  role: Pseudoaligns paired-end RNA-Seq reads to an Ensembl transcriptome index and
    quantifies transcript abundances (TPM) for each sample
- name: CanIsoNet
  role: Downstream database consuming the TPM abundance matrix to explore per-sample
    isoform switching events with expression values
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:c03a8e8eedfd94c134fe322ab71072271f6814e26804cc2bb7511f6a7de13ccb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - Kallisto
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# kallisto-pseudoalignment-quantification

## Summary

Quantify transcript-level abundances (TPM) from paired-end RNA-Seq FASTQ data using Kallisto against an Ensembl transcriptome index (human GRCh37.p13), so that downstream isoform switching analyses such as CanIsoNet operate on transcript isoform definitions consistent with Ensembl cDNA.

## When to use

Use when raw RNA-Seq FASTQ reads (e.g. from GEO/SRA/ENA accessions) must be converted to transcript-level TPM abundances for downstream isoform-based analyses such as CanIsoNet.

## When NOT to use

- Input is already a gene-level or transcript-level expression matrix — quantification from raw reads is unnecessary
- Single-end reads or non-Ensembl transcript definitions are required, since this workflow specifies paired-end mode and an Ensembl GRCh37.p13 cDNA index for isoform compatibility with CanIsoNet
- Analysis targets PCAWG cancer samples, which in CanIsoNet derive from a different data source than the four RNA-Seq disease datasets

## Inputs

- Paired-end RNA-Seq FASTQ files (e.g. GSE203206, GSE128177, GSE205867 via GEO/SRA; PRJNA433105 via direct ENA run downloads)
- Ensembl transcript cDNA sequences (human GRCh37.p13; mouse GRCm38.p6) for building the Kallisto index

## Outputs

- Per-sample Kallisto quant output directories with transcript-level abundance estimates (TPM)
- Aggregated TPM abundance matrix (tpm_abundance_matrix.tsv) with transcripts as rows and samples as columns

## How to apply

First download the FASTQ files of the target RNA-Seq datasets — here GSE203206, GSE128177 and GSE205867 via GEO/SRA and PRJNA433105 via direct ENA run downloads — then build a Kallisto index from Ensembl human GRCh37.p13 transcript cDNA sequences (mouse data use GRCm38.p6) so quantification is compatible with the transcript isoform definitions used downstream. Run 'kallisto quant' in paired-end mode on each FASTQ sample to obtain transcript-level abundance estimates, since Kallisto's pseudoalignment quantifies against the full transcriptome without per-read alignment to a genome. Collect the TPM values from the per-sample quantification output files and aggregate them into a single matrix with transcripts as rows and samples as columns (tpm_abundance_matrix.tsv). Validate that the matrix contains one column per sample from all four disease datasets (OIH, Alzheimer's, Parkinson's, Behçet's) and that all TPM entries are non-negative numeric values before feeding it into CanIsoNet.

## Related tools

- **Kallisto** (Pseudoaligns paired-end RNA-Seq reads to an Ensembl transcriptome index and quantifies transcript abundances (TPM) for each sample)
- **CanIsoNet** (Downstream database consuming the TPM abundance matrix to explore per-sample isoform switching events with expression values) — https://github.com/kahramanlab/CanIsoNet_Web

## Examples

```
kallisto index -i GRCh37.p13_cdna.idx Ensembl.GRCh37.p13.cdna.fa; kallisto quant -i GRCh37.p13_cdna.idx -o sample1_quant read1.fastq.gz read2.fastq.gz
```

## Evaluation signals

- tpm_abundance_matrix.tsv contains one column per sample from all four disease datasets (GSE203206, GSE128177, GSE205867, PRJNA433105)
- All TPM entries in the matrix are non-negative numeric values
- The Kallisto index was built from Ensembl GRCh37.p13 transcript cDNA (not a genome reference) so isoform IDs match downstream CanIsoNet definitions
- Quantification was run in paired-end mode on each FASTQ sample, producing per-sample quant output files
- Rows of the matrix correspond to Ensembl transcripts, matching the isoform-level annotation scope of CanIsoNet

## Limitations

- The workflow requires paired-end FASTQ input; single-end data would need a different Kallisto invocation not covered here
- Index must be rebuilt against the correct Ensembl release (GRCh37.p13 for human, GRCm38.p6 for mouse); using a different release breaks compatibility with downstream isoform switching annotations
- TPM values are relative abundances per sample and are not suitable for cross-sample absolute comparison without additional normalization

## Evidence

- [methods] Transcript abundances for the disease RNAseq data were quantified with Kallisto against Ensembl transcript sequences (human GRCh37.p13, mouse GRCm38.p6), providing the TPM values used in CanIsoNet.: [evidence span withheld — anchor status: not_found]
- [other] Build a Kallisto index from the Ensembl GRCh37.p13 human transcript cDNA sequences so that quantification is compatible with the transcript isoform definitions used downstream.: [evidence span withheld — anchor status: not_found]
- [intro] four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease: "four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease"
- [readme] Exploring the isoform switching event in each sample with their expression values.: "4) Exploring the isoform switching event in each sample with their expression values."
