---
name: fastq-quality-assessment
description: Use when raw paired-end FASTQ files (e.g. SRA accessions SRP058295, SRP255211)
  must be QC-checked before STAR alignment or variant calling, or when read origin
  needs verification via sampled BLAST.
when_to_use_negative:
- Input is already aligned BAM/SAM output from STAR — proceed directly to Picard duplicate
  marking instead of FASTQ QC
- Files are not FASTQ (e.g. VCF or genotyping panel data) — use Compare_VCF_Files-style
  VCF filtering instead
edam_operation: http://edamontology.org/operation_0370
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: FastQC
  role: Reviews all fastq files for sequence quality as the pipeline's QC gate
- name: FastqBLAST
  role: Samples fastq sequences, trims low-quality ends, and BLASTs them via Biopython
    to verify read provenance and produce a summary report
  repo: https://github.com/juniper-lake/FastqBLAST
- name: Biopython
  role: Package used by FastqBLAST to remotely submit random sampled sequences to
    NCBI's BLAST
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:009406ab2fae7bbfd68586c22b0fbbbcf057289aefc010b9b1dd1a4fb8f93d19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/juniper-lake/FastqBLAST
  tools:
  - Biopython
  - FastQC
  - FastqBLAST
  - Picard
  edam_operation: http://edamontology.org/operation_0370
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# fastq-quality-assessment

## Summary

Screens RNA-seq FASTQ files for sequence quality with FastQC before downstream alignment, supplemented by FastqBLAST sampling and BLAST verification of read provenance. Quality screening is the mandatory first gate of the broiler chicken ASE variant-calling pipeline.

## When to use

Use when raw paired-end FASTQ files (e.g. SRA accessions SRP058295, SRP255211) must be QC-checked before STAR alignment or variant calling, or when read origin needs verification via sampled BLAST.

## When NOT to use

- Input is already aligned BAM/SAM output from STAR — proceed directly to Picard duplicate marking instead of FASTQ QC
- Files are not FASTQ (e.g. VCF or genotyping panel data) — use Compare_VCF_Files-style VCF filtering instead

## Inputs

- Paired-end FASTQ files from SRA accessions SRP058295 (abdominal fat) and SRP255211 (breast muscle/liver)
- FastQC parameters (default settings)

## Outputs

- FastQC per-sample sequence quality reports confirming files pass QC
- FastqBLAST tabular summary report splitting sampled reads into hit vs. no-hit results

## How to apply

Download the paired-end broiler RNA-seq fastq files (abdominal fat from SRA SRP058295, breast muscle/liver from SRA SRP255211) and run FastQC on every file to review per-base sequence quality; in this study all samples passed this quality control step before proceeding. Optionally corroborate read provenance with FastqBLAST, which samples sequences from the fastq, trims low-quality ends (ASCII-64 quality encoding handled via the --ascii64 flag), and remotely submits sampled sequences to NCBI BLAST via Biopython to produce a tabular hit/no-hit report. Only fastq files that pass quality review are carried into STAR 2.5.2b two-pass alignment; reads are soft-clipped rather than trimmed at the alignment stage. Note that per-sample STAR statistics (e.g. the ~87.24% average unique mapping rate and higher 'too short' read fractions in breast muscle) provide a downstream check that upstream QC and read integrity were sound.

## Related tools

- **FastQC** (Reviews all fastq files for sequence quality as the pipeline's QC gate)
- **FastqBLAST** (Samples fastq sequences, trims low-quality ends, and BLASTs them via Biopython to verify read provenance and produce a summary report) — https://github.com/juniper-lake/FastqBLAST
- **Biopython** (Package used by FastqBLAST to remotely submit random sampled sequences to NCBI's BLAST)

## Examples

```
python FastqBLAST.py demo_data.fastq --email myemail@udel.edu
```

## Evaluation signals

- FastQC reports are generated for every fastq file and all samples pass the quality control step before alignment
- STAR alignment statistics of processed samples are consistent with the paper's reported average unique mapping rate of 87.24%
- FastqBLAST produces a summary report with sequences correctly split into hit vs. no-hit files
- Per-tissue STAR 'too short' read percentages fall in the reported ranges (avg. 15.39% breast muscle vs. 4.50% abdominal fat and 5.33% liver)

## Limitations

- Mapping issues with muscle tissue samples were the biggest pipeline problem, likely due to the current genome build and annotation of muscle-related genes — QC pass does not guarantee good downstream mapping
- FastqBLAST requires Python 3 with Biopython and remote NCBI BLAST access; Phred score encoding (ASCII 33 vs ASCII 64) must be set correctly via --ascii64

## Evidence

- [methods] All fastq files were reviewed for sequence quality using FastQC and all samples passed this quality control step: "All fastq files were reviewed for sequence quality using FastQC24 and all samples passed this quality control step"
- [readme] This program takes a sample of sequences from a fastq file, trims the low quality ends, BLASTs them, fetches additional info from NCBI, and produces a tabular report.: [evidence span withheld — anchor status: external_doc]
- [methods] remotely submit random samples of sequences to NCBI’s BLAST using the Biopython package: "remotely submit random samples of sequences to NCBI’s BLAST using the Biopython package39"
- [other] Download the 68 paired-end broiler RNA-seq fastq files from SRA accessions SRP058295 (abdominal fat) and SRP255211 (breast muscle/liver) and screen each fastq for sequence quality with FastQC.: [evidence span withheld — anchor status: not_found]
