---
name: variant-calling-from-wes-tumor-normal-pairs
description: Use when you have paired tumor/normal WES or WGS FASTQ files and need
  somatic SNV/INDEL calls as the starting point for downstream neoantigen prediction.
when_to_use_negative:
- Input is already a VCF of somatic variants or pre-called mutations — skip straight
  to annotation/prediction steps
- Data are RNA-Seq only without WES/WGS — TSNAD's variant calling module requires
  tumor-normal DNA sequencing input
- Sequencing depth is below 15x for WGS or 50x for WES — calls would be unreliable
  per the article's depth guidance
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: TSNAD v2.0
  role: One-stop wrapper that chains trimming, mapping, recalibration, and Mutect2
    variant calling
  repo: https://github.com/jiujiezz/tsnad
- name: Trimmomatic 0.39
  role: Trims and crops raw WES/WGS reads and removes artifacts
- name: BWA 0.7.17
  role: Maps short reads to the GRCh37 or GRCh38 reference genome
- name: SAMtools 1.13
  role: Processes aligned reads (bundled in TSNAD Tools/)
- name: GATK 4.2.0.0
  role: Removes repeat sequences, recalibrates base quality scores to the final BAM,
    and calls SNVs/INDELs via Mutect2
  repo: https://github.com/broadinstitute/gatk
- name: Docker
  role: 'Distribution mechanism: docker pull biopharm/tsnad:latest installs the whole
    toolchain'
  repo: https://hub.docker.com/r/biopharm/tsnad
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:2241a2550c1ece9f22a07d41fd8c800903c1c5486c2e3a2c0edd6d72785f0f50
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
  - GATK 4.2.0.0
  - HISAT2
  - OptiType
  - SAMtools
  - SAMtools 1.13
  - StringTie
  - TSNAD v2.0
  - Trimmomatic
  - Trimmomatic 0.39
  - VEP
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# Variant calling from WES tumor-normal pairs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A one-stop pipeline step that turns raw tumor-normal whole-exome sequencing (WES) reads into somatic SNV/INDEL calls using TSNAD v2.0's embedded Trimmomatic 0.39, BWA 0.7.17, SAMtools 1.13, and GATK 4.2.0.0 Mutect2, under either GRCh37 or GRCh38.

## When to use

Use when you have paired tumor/normal WES or WGS FASTQ files and need somatic SNV/INDEL calls as the starting point for downstream neoantigen prediction.

## When NOT to use

- Input is already a VCF of somatic variants or pre-called mutations — skip straight to annotation/prediction steps
- Data are RNA-Seq only without WES/WGS — TSNAD's variant calling module requires tumor-normal DNA sequencing input
- Sequencing depth is below 15x for WGS or 50x for WES — calls would be unreliable per the article's depth guidance

## Inputs

- Raw tumor WES/WGS FASTQ files
- Raw matched normal WES/WGS FASTQ files
- Reference genome selection (GRCh37 or GRCh38)
- Optional tumor RNA-Seq FASTQ for expression filtering

## Outputs

- Recalibrated final BAM files
- Somatic SNV/INDEL variant calls from GATK Mutect2
- Downstream annotated mutant peptides (fed to VEP and DeepHLApan in TSNAD)

## How to apply

Run TSNAD v2.0 (Docker image biopharm/tsnad:latest) after mounting sample, RNA-seq, and output directories. The pipeline trims and crops raw reads with Trimmomatic 0.39, maps them with BWA 0.7.17 to a reference genome (GRCh37 or GRCh38 selected via the -V flag), and uses GATK 4.2.0.0 to remove repeat sequences and recalibrate base quality scores into the final BAM file. Somatic SNVs and INDELs are then called with the Mutect2 module of GATK. Check sequencing depth before trusting calls: minimum 15x for WGS and 50x for WES, with 30x/100x recommended.

## Related tools

- **TSNAD v2.0** (One-stop wrapper that chains trimming, mapping, recalibration, and Mutect2 variant calling) — https://github.com/jiujiezz/tsnad
- **Trimmomatic 0.39** (Trims and crops raw WES/WGS reads and removes artifacts)
- **BWA 0.7.17** (Maps short reads to the GRCh37 or GRCh38 reference genome)
- **SAMtools 1.13** (Processes aligned reads (bundled in TSNAD Tools/))
- **GATK 4.2.0.0** (Removes repeat sequences, recalibrates base quality scores to the final BAM, and calls SNVs/INDELs via Mutect2) — https://github.com/broadinstitute/gatk
- **Docker** (Distribution mechanism: docker pull biopharm/tsnad:latest installs the whole toolchain) — https://hub.docker.com/r/biopharm/tsnad

## Examples

```
docker pull biopharm/tsnad:latest; python TSNAD.py -I samples/ -R RNA-seq/ -V grch38 -O results/
```

## Evaluation signals

- Final BAM files exist after GATK base quality score recalibration
- Mutect2 outputs somatic SNV/INDEL calls for every tumor-normal pair supplied
- Sequencing depth meets the minimum 15x (WGS) / 50x (WES) threshold, ideally 30x/100x
- Calls were produced under the intended reference genome (GRCh37 vs GRCh38) matching the -V flag
- Called variants successfully feed the downstream VEP annotation and 8-11mer mutant peptide extraction steps

## Limitations

- TSNAD v1.0 only supports GRCh37; GRCh38 support is new in v2.0, so reference genome choice changes results
- Shallow sequencing (below 15x WGS / 50x WES) undermines call quality
- Docker image is large and the initial pull may take several hours

## Evidence
