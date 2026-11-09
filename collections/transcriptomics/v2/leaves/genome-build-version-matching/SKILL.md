---
name: genome-build-version-matching
description: Use when quantifying transcript abundances from RNA-seq FASTQ files that
  feed isoform-level analyses (e.g. CanIsoNet), and transcript identifiers must match
  a specific Ensembl genome build such as human GRCh37.p13.
when_to_use_negative:
- Input is already a gene-level count or TPM table — quantifying transcript abundances
  with a genome-build-matched index is unnecessary
- Downstream analysis uses gene symbols or a different reference (e.g. GRCh38 transcripts)
  — an index built on GRCh37.p13 isoforms would misalign identifiers
- The sample is from an organism whose transcriptome build has not been defined (only
  human GRCh37.p13 and mouse GRCm38.p6 are supported here)
edam_operation: http://edamontology.org/operation_0266
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Kallisto
  role: Builds the Ensembl GRCh37.p13 transcript cDNA index and runs paired-end quantification
    to produce transcript-level TPM estimates
- name: CanIsoNet
  role: 'Downstream consumer: its isoform switching events and interaction networks
    are defined on Ensembl transcript isoforms, which dictates the genome-build choice
    for the index'
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:4bb2641c731223f4b3ecead8d106cdc9fe1505b383f9bc2d554fb3928330f5c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - Kallisto
  edam_operation: http://edamontology.org/operation_0266
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# genome-build-version-matching

## Summary

Build the Kallisto transcriptome index from the Ensembl genome-build-specific cDNA (human GRCh37.p13, mouse GRCm38.p6) so that transcript-level TPM quantification of RNA-seq data remains compatible with the transcript isoform definitions used downstream in CanIsoNet.

## When to use

Use when quantifying transcript abundances from RNA-seq FASTQ files that feed isoform-level analyses (e.g. CanIsoNet), and transcript identifiers must match a specific Ensembl genome build such as human GRCh37.p13.

## When NOT to use

- Input is already a gene-level count or TPM table — quantifying transcript abundances with a genome-build-matched index is unnecessary
- Downstream analysis uses gene symbols or a different reference (e.g. GRCh38 transcripts) — an index built on GRCh37.p13 isoforms would misalign identifiers
- The sample is from an organism whose transcriptome build has not been defined (only human GRCh37.p13 and mouse GRCm38.p6 are supported here)

## Inputs

- Paired-end RNA-seq FASTQ files for the four disease datasets (OIH: GSE203206 and GSE205867; Alzheimer's Disease: GSE128177; Parkinson's/Behçet's: PRJNA433105)
- Ensembl human GRCh37.p13 transcript cDNA FASTA sequences (Ensembl mouse GRCm38.p6 cDNA for mouse samples)

## Outputs

- Kallisto index built from Ensembl GRCh37.p13 transcript cDNA sequences
- Per-sample Kallisto quant output files with transcript-level abundance estimates (TPM)
- Aggregated transcript-by-sample TPM table, tpm_abundance_matrix.tsv

## How to apply

Download the paired-end FASTQ files of the disease RNA-seq datasets (GSE203206, GSE128177 and GSE205867 via GEO/SRA accessions; PRJNA433105 via direct ENA run downloads) rather than quantifying at gene level. Build the Kallisto index from Ensembl human GRCh37.p13 transcript cDNA sequences — not the current Ensembl release — so that quantification is compatible with the transcript isoform definitions used downstream (for mouse data, use GRCm38.p6). Run kallisto quant in paired-end mode on each FASTQ sample to obtain transcript-level abundance estimates. Collect TPM values from the per-sample quantification files and aggregate them into one matrix with transcripts as rows and samples as columns, saved as tpm_abundance_matrix.tsv. The rationale is that isoform switching events in CanIsoNet are defined on specific Ensembl isoform IDs, so quantifying against a mismatched build would misalign TPM values to the wrong isoform definitions.

## Related tools

- **Kallisto** (Builds the Ensembl GRCh37.p13 transcript cDNA index and runs paired-end quantification to produce transcript-level TPM estimates)
- **CanIsoNet** (Downstream consumer: its isoform switching events and interaction networks are defined on Ensembl transcript isoforms, which dictates the genome-build choice for the index) — https://github.com/kahramanlab/CanIsoNet_Web

## Examples

```
kallisto index -i grch37p13_cdna.idx Ensembl_GRCh37.p13.cdna.all.fa && kallisto quant -i grch37p13_cdna.idx -o sample1_quant sample1_R1.fastq.gz sample1_R2.fastq.gz
```

## Evaluation signals

- tpm_abundance_matrix.tsv contains one column per sample from all four disease datasets (GSE203206, GSE128177, GSE205867, PRJNA433105)
- All TPM entries in tpm_abundance_matrix.tsv are non-negative numeric values
- Transcript identifiers in the TPM matrix resolve to Ensembl GRCh37.p13 isoform definitions (build provenance of the index FASTA is documented)
- Kallisto quant was run in paired-end mode for every sample (pairing consistency check in quant logs)
- TPM values are finite and row-normalizable (sum of TPMs per sample is bounded), consistent with transcript-level abundance output

## Limitations

- Only human GRCh37.p13 and mouse GRCm38.p6 Ensembl transcriptome builds are documented; other organisms or builds are not covered
- Building the index from a newer Ensembl release (e.g. GRCh38) would break compatibility with the transcript isoform definitions used in CanIsoNet
- The README states 11,528 isoforms while the article abstract states 11,811 isoforms annotated, so downstream isoform coverage may differ between database versions

## Evidence

- [methods] Transcript abundances for the disease RNAseq data were quantified with Kallisto against Ensembl transcript sequences (human GRCh37.p13, mouse GRCm38.p6): [evidence span withheld — anchor status: not_found]
- [readme] four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease: "four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease"
- [abstract] Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available.: "Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available."
