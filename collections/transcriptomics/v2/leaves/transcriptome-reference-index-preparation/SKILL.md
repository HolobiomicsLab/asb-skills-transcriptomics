---
name: transcriptome-reference-index-preparation
description: Use when quantifying transcript abundances with Kallisto from RNA-Seq
  FASTQ files and an Ensembl transcript cDNA reference must first be converted into
  a Kallisto index that matches downstream isoform-switching analysis.
when_to_use_negative:
- When abundances must be gene-level or genome-alignment based (e.g. STAR/RSEM), since
  CanIsoNet's isoform-switching analysis depends on transcript-level TPM against Ensembl
  isoform definitions
- When the reference must be single-cell-compatible or alignment-free without paired-end
  FASTQ input; the described workflow is paired-end Kallisto quant only
edam_operation: http://edamontology.org/operation_0526
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Kallisto
  role: Builds the transcriptome index from Ensembl cDNA and performs paired-end transcript-level
    quantification (TPM) for each disease dataset sample
- name: CanIsoNet
  role: Downstream database consuming the TPM abundance matrix; the index must be
    built on Ensembl transcript definitions compatible with its isoform-switching
    analysis
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:08f9ef5bef2627d2a8bc79564a4a5600ff992b3b910d19eddf4d59c631bfb245
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - Kallisto
  edam_operation: http://edamontology.org/operation_0526
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# transcriptome-reference-index-preparation

## Summary

Prepare a transcript-level reference index for Kallisto from Ensembl cDNA sequences (human GRCh37.p13, mouse GRCm38.p6) so that transcript abundance quantification (TPM) for the four disease RNA-Seq datasets (OIH, Alzheimer's, Parkinson's, Behcet's) is compatible with the transcript isoform definitions used downstream in CanIsoNet.

## When to use

Use when quantifying transcript abundances with Kallisto from RNA-Seq FASTQ files and an Ensembl transcript cDNA reference must first be converted into a Kallisto index that matches downstream isoform-switching analysis.

## When NOT to use

- When abundances must be gene-level or genome-alignment based (e.g. STAR/RSEM), since CanIsoNet's isoform-switching analysis depends on transcript-level TPM against Ensembl isoform definitions
- When the reference must be single-cell-compatible or alignment-free without paired-end FASTQ input; the described workflow is paired-end Kallisto quant only

## Inputs

- RNA-Seq paired-end FASTQ files from disease datasets (GSE203206, GSE128177, GSE205867, PRJNA433105)
- Ensembl transcript cDNA sequences FASTA (human GRCh37.p13, mouse GRCm38.p6)

## Outputs

- Kallisto transcriptome index file (e.g. transcripts.idx)
- Per-sample Kallisto quantification directories with transcript-level TPM estimates
- Aggregated TPM abundance matrix (tpm_abundance_matrix.tsv) with transcripts as rows and samples as columns

## How to apply

Download the FASTQ files for the disease RNA-Seq datasets via their GEO/SRA accessions (GSE203206, GSE128177, GSE205867) or direct ENA run downloads (PRJNA433105). Obtain the Ensembl transcript cDNA sequences for the relevant organism — human GRCh37.p13 or mouse GRCm38.p6 — rather than genome+annotation, because Kallisto quantification must align to the same transcript isoform definitions used in the downstream isoform-switching analysis. Run 'kallisto index' on the cDNA FASTA to build the index, then run 'kallisto quant' in paired-end mode on each sample's FASTQ pair. Collect per-sample TPM values and aggregate them into a transcripts-by-samples matrix (e.g. tpm_abundance_matrix.tsv).

## Related tools

- **Kallisto** (Builds the transcriptome index from Ensembl cDNA and performs paired-end transcript-level quantification (TPM) for each disease dataset sample)
- **CanIsoNet** (Downstream database consuming the TPM abundance matrix; the index must be built on Ensembl transcript definitions compatible with its isoform-switching analysis) — https://github.com/kahramanlab/CanIsoNet_Web

## Examples

```
kallisto index -i GRCh37.p13.cdna.idx GRCh37.p13.cdna.all.fa && kallisto quant -i GRCh37.p13.cdna.idx -o sample1_out sample1_R1.fastq.gz sample1_R2.fastq.gz
```

## Evaluation signals

- tpm_abundance_matrix.tsv contains exactly one column per sample from all four disease datasets
- All TPM entries in the matrix are non-negative numeric values
- Index was built from Ensembl GRCh37.p13 (or GRCm38.p6) transcript cDNA, not from a genome reference, ensuring isoform-level compatibility
- Each sample's Kallisto quant output exists and yields a TPM column that aggregates cleanly into the matrix without missing transcripts

## Limitations

- Quantification is restricted to the four non-cancer disease RNA-Seq datasets; the 27 cancer types in CanIsoNet come from PCAWG, not this Kallisto workflow
- Index must match the exact Ensembl release/transcript set; a mismatched reference would break downstream isoform-switching event definitions
- The workflow assumes paired-end FASTQ data; single-end data would require different kallisto quant parameters not described in the source

## Evidence

- [other] Build a Kallisto index from the Ensembl GRCh37.p13 human transcript cDNA sequences so that quantification is compatible with the transcript isoform definitions used downstream: [evidence span withheld — anchor status: not_found]
- [other] Transcript abundances for the disease RNAseq data were quantified with Kallisto against Ensembl transcript sequences (human GRCh37.p13, mouse GRCm38.p6), providing the TPM values used in CanIsoNet.: [evidence span withheld — anchor status: not_found]
- [readme] four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease: "four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease"
