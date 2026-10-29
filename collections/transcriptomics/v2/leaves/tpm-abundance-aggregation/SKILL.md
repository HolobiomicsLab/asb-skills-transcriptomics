---
name: tpm-abundance-aggregation
description: Use when raw paired-end FASTQ files from the disease RNA-seq datasets
  (GSE203206, GSE128177, GSE205867, PRJNA433105) must be converted into transcript-level
  TPM values compatible with CanIsoNet's transcript isoform definitions.
when_to_use_negative:
- Input is already a transcript-level TPM or expression matrix — aggregation has already
  been performed and re-running Kallisto would be redundant
- Analysis targets the 27 PCAWG cancer types, which were processed by the prior isoform-specific
  interaction network study rather than by RNA-seq quantification of these four disease
  datasets
- Gene-level abundance estimates are required — this workflow deliberately quantifies
  at the transcript/isoform level to preserve the isoform switching definitions used
  in CanIsoNet
edam_operation: http://edamontology.org/operation_3806
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Kallisto
  role: Builds the Ensembl GRCh37.p13 cDNA index and runs quant in paired-end mode
    to produce per-sample transcript TPM estimates
- name: CanIsoNet
  role: Downstream database that consumes the aggregated TPM abundance matrix for
    per-sample isoform switching exploration with expression values
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:eef4df3990ebfabf7dfd3852516c54fe0df73c0e71cb33494b2a210ffedc3404
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - Kallisto
  edam_operation: http://edamontology.org/operation_3806
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# tpm-abundance-aggregation

## Summary

Quantifies transcript-level abundances for the four CanIsoNet disease RNA-seq datasets (OIH, Alzheimer's, Parkinson's, Behcet's) with Kallisto against Ensembl GRCh37.p13 human cDNA, then aggregates per-sample TPM values into a single transcripts-by-samples matrix used downstream by CanIsoNet.

## When to use

Use when raw paired-end FASTQ files from the disease RNA-seq datasets (GSE203206, GSE128177, GSE205867, PRJNA433105) must be converted into transcript-level TPM values compatible with CanIsoNet's transcript isoform definitions.

## When NOT to use

- Input is already a transcript-level TPM or expression matrix — aggregation has already been performed and re-running Kallisto would be redundant
- Analysis targets the 27 PCAWG cancer types, which were processed by the prior isoform-specific interaction network study rather than by RNA-seq quantification of these four disease datasets
- Gene-level abundance estimates are required — this workflow deliberately quantifies at the transcript/isoform level to preserve the isoform switching definitions used in CanIsoNet

## Inputs

- Paired-end RNA-seq FASTQ files from four public disease datasets (GSE203206, GSE128177, GSE205867 via GEO/SRA; PRJNA433105 via ENA run downloads)
- Ensembl human GRCh37.p13 transcript cDNA sequences (FASTA) for index building
- Ensembl mouse GRCm38.p6 transcript cDNA sequences (for mouse samples)

## Outputs

- tpm_abundance_matrix.tsv: transcripts as rows, samples as columns, TPM values aggregated from all four disease datasets
- Per-sample Kallisto quant output directories with transcript-level abundance estimates

## How to apply

Download the FASTQ files of the publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease — GSE203206, GSE128177 and GSE205867 via GEO/SRA accessions and PRJNA433105 via direct ENA run downloads. Build a Kallisto index from the Ensembl GRCh37.p13 human transcript cDNA sequences so that quantification is compatible with the transcript isoform definitions used downstream (a mouse GRCm38.p6 index is used for mouse data). Run Kallisto quant in paired-end mode on each FASTQ sample to obtain transcript-level abundance estimates. Collect the TPM values from the per-sample quantification files and aggregate them into one matrix with transcripts as rows and samples as columns, saved as tpm_abundance_matrix.tsv; building the index from the same Ensembl cDNA reference is the key decision point because transcript IDs must match the isoform identifiers consumed by CanIsoNet.

## Related tools

- **Kallisto** (Builds the Ensembl GRCh37.p13 cDNA index and runs quant in paired-end mode to produce per-sample transcript TPM estimates)
- **CanIsoNet** (Downstream database that consumes the aggregated TPM abundance matrix for per-sample isoform switching exploration with expression values) — https://github.com/kahramanlab/CanIsoNet_Web

## Examples

```
kallisto quant -i GRCh37.p13_cdna.idx -o sample1_out sample1_R1.fastq.gz sample1_R2.fastq.gz
```

## Evaluation signals

- tpm_abundance_matrix.tsv contains one column per sample from all four disease datasets (GSE203206, GSE128177, GSE205867, PRJNA433105)
- All TPM entries in tpm_abundance_matrix.tsv are non-negative numeric values
- Matrix orientation is transcripts as rows and samples as columns
- The Kallisto index was built from Ensembl GRCh37.p13 human transcript cDNA sequences so transcript IDs are compatible with the transcript isoform definitions used downstream
- Kallisto quant was run in paired-end mode on each FASTQ sample (paired-end read files supplied per sample)

## Limitations

- Reference transcriptomes are fixed to Ensembl human GRCh37.p13 and mouse GRCm38.p6; other genome builds would break compatibility with the isoform definitions used in CanIsoNet
- Workflow assumes paired-end FASTQ input; single-end libraries are not covered by the described quantification mode
- Only the four non-cancer disease RNA-seq datasets (OIH, Alzheimer's, Parkinson's, Behcet's) are quantified this way; the 27 PCAWG cancer types come from the earlier Kahraman et al. 2020 analysis

## Evidence

- [other] quantified with Kallisto against Ensembl transcript sequences (human GRCh37.p13, mouse GRCm38.p6), providing the TPM values used in CanIsoNet: [evidence span withheld — anchor status: not_found]
- [other] Build a Kallisto index from the Ensembl GRCh37.p13 human transcript cDNA sequences so that quantification is compatible: [evidence span withheld — anchor status: not_found]
- [readme] four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease: "We have recently analyzed the pathogenic impact of isoform switching events in 31 disease types covering 27 different cancer types from PCAWG project, and four publicly available RNA-Seq datasets on"
- [readme] Exploring the isoform switching event in each sample with their expression values: "4) Exploring the isoform switching event in each sample with their expression values. <br>"
