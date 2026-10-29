---
name: hla-allele-typing-from-wes-reads
description: Use when raw WES/WGS FASTQ reads of a tumor-normal pair must be typed
  for HLA class I alleles as a prerequisite step before peptide-MHC binding prediction
  (e.g., DeepHLApan) in a neoantigen discovery workflow.
when_to_use_negative:
- Input is already a sorted BAM/VCF or an annotated variant list with HLA types provided
  clinically (e.g., the TESLA clinical-grade HLA typing) — typing from reads is redundant
- Only HLA class II typing is needed — OptiType identifies HLA class I alleles
- Input is RNA-Seq-only without WES/WGS data and no support for the TSNAD WES/WGS
  mutation-calling modules is intended
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: OptiType 1.3.5
  role: Embedded HLA allele identification module that types class I alleles from
    processed reads within TSNAD v2.0
- name: TSNAD v2.0
  role: One-stop pipeline orchestrating trimming, alignment, recalibration, and HLA
    typing invocation
  repo: https://github.com/jiujiezz/tsnad
- name: Docker
  role: Installation method bundling all needed tools and reference files (biopharm/tsnad
    image)
  repo: https://hub.docker.com/r/biopharm/tsnad
- name: BWA 0.7.17
  role: Maps short WES/WGS reads to GRCh37 or GRCh38 prior to HLA typing
- name: Trimmomatic 0.39
  role: Trims and crops raw WES/WGS reads and removes artifacts before alignment and
    typing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:2a7867a609f2d65da95758813853531aabeef38bd27a17e0e1820b4b843b54af
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jiujiezz/tsnad
  repo_url: https://github.com/jiujiezz/tsnad
  tools:
  - BWA
  - BWA 0.7.17
  - DeepHLApan
  - Docker
  - GATK
  - HISAT2
  - OptiType
  - OptiType 1.3.5
  - SAMtools
  - StringTie
  - TSNAD v2.0
  - Trimmomatic
  - Trimmomatic 0.39
  - VEP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# hla-allele-typing-from-wes-reads

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify the HLA alleles of a patient from WES/WGS reads (with optional tumor RNA-Seq support) using OptiType 1.3.5 embedded in the TSNAD v2.0 one-stop pipeline, producing the patient-specific pMHC typing required for downstream neoantigen prediction.

## When to use

Use when raw WES/WGS FASTQ reads of a tumor-normal pair must be typed for HLA class I alleles as a prerequisite step before peptide-MHC binding prediction (e.g., DeepHLApan) in a neoantigen discovery workflow.

## When NOT to use

- Input is already a sorted BAM/VCF or an annotated variant list with HLA types provided clinically (e.g., the TESLA clinical-grade HLA typing) — typing from reads is redundant
- Only HLA class II typing is needed — OptiType identifies HLA class I alleles
- Input is RNA-Seq-only without WES/WGS data and no support for the TSNAD WES/WGS mutation-calling modules is intended

## Inputs

- Raw WES or WGS FASTQ reads of tumor and normal samples (depth >= 50x WES / 15x WGS minimum; 100x WES / 30x WGS recommended)
- Optional tumor RNA-Seq FASTQ reads
- Reference genome selection: GRCh37 or GRCh38

## Outputs

- Patient HLA class I allele calls from OptiType, consumed internally by TSNAD v2.0 for peptide-MHC binding and immunogenicity prediction
- TSNAD v2.0 run outputs under results/ (final neoantigens in results/deephlapan_results/)

## How to apply

Run TSNAD v2.0 (Docker image biopharm/tsnad:latest or the GitHub install with all Tools/ bundled) which invokes OptiType (v1.3.5) as its HLA allele identification module after the upstream processing: Trimmomatic 0.39 trims and crops raw WES/WGS reads, BWA 0.7.17 maps them to GRCh37 or GRCh38, and GATK 4.2.0.0 performs duplicate/repeat removal and base-quality recalibration to create the final BAM file. OptiType then outputs the patient's HLA class I alleles, which TSNAD v2.0 passes forward so DeepHLApan can predict binding and immunogenicity for the extracted 8-11 amino acid mutant peptides against the correct MHC molecules. Quality of typing can be sanity-checked against clinical-grade HLA typing as was done on the TESLA dataset (five patients from Synapse syn21048999). Selecting the matching reference genome version (-V grch37 or grch38) is required since TSNAD v2.0 supports both while TSNAD v1.0 only supported GRCh37.

## Related tools

- **OptiType 1.3.5** (Embedded HLA allele identification module that types class I alleles from processed reads within TSNAD v2.0)
- **TSNAD v2.0** (One-stop pipeline orchestrating trimming, alignment, recalibration, and HLA typing invocation) — https://github.com/jiujiezz/tsnad
- **Docker** (Installation method bundling all needed tools and reference files (biopharm/tsnad image)) — https://hub.docker.com/r/biopharm/tsnad
- **BWA 0.7.17** (Maps short WES/WGS reads to GRCh37 or GRCh38 prior to HLA typing)
- **Trimmomatic 0.39** (Trims and crops raw WES/WGS reads and removes artifacts before alignment and typing)

## Examples

```
docker pull biopharm/tsnad:latest; docker run -it -v [dir of WES/WGS]/:/home/tsnad/samples -v [output dir]:/home/tsnad/results biopharm/tsnad:latest /bin/bash; python TSNAD.py -I samples/ -V grch38 -O results/
```

## Evaluation signals

- OptiType module completes without error inside the TSNAD v2.0 run and its allele calls are fed into DeepHLApan (results/deephlapan_results/ contains final predictions)
- HLA typing is concordant with clinical-grade HLA typing, as available for the five TESLA patients (Synapse syn21048999)
- The chosen reference genome flag (-V grch37 or grch38) matches the genome actually used for BWA alignment
- WES depth meets the recommended 100x (minimum 50x) so allele calls are reliable

## Limitations

- OptiType provides HLA class I typing only; clinical-grade typing may still be preferable for validation
- Low sequencing depth (below the 15x WGS / 50x WES minimums) degrades typing and downstream neoantigen calls
- TSNAD v2.0 has no TCR sequencing analysis or TCR-pMHC binding prediction model; the HLA typing supports peptide-MHC binding only

## Evidence

- [results] TESLA dataset provides clinical-grade HLA typing for validation: [evidence span withheld — anchor status: not_found]
