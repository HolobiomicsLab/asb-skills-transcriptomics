---
name: wgs-read-quality-trimming
description: Use when raw tumor-normal WES/WGS FASTQ reads are about to be mapped
  to a reference genome (GRCh37/GRCh38) and adapter or low-quality artifacts must
  be removed first.
when_to_use_negative:
- Input reads are already aligned (BAM/SAM) — trimming applies only to raw reads at
  the start of the pipeline
- The data is RNA-Seq rather than WES/WGS — TSNAD v2.0 handles RNA-Seq with the separate
  STAR/Arriba module instead
- Reads are synthetic or pre-cleaned test data with no adapters, where trimming adds
  no value before alignment
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Trimmomatic 0.39
  role: Trims and crops raw WES/WGS reads and removes adapter artifacts before genome
    mapping
  repo: https://github.com/jiujiezz/tsnad
- name: BWA 0.7.17
  role: Consumes the trimmed reads to map short sequences to GRCh37/GRCh38
  repo: https://github.com/jiujiezz/tsnad
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:c28b2d817629cdc69f0c883234903267ddfc3251ba3eb85030c5f467bbd348c7
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
  - GATK
  - Mutect2
  - SAMtools
  - TSNAD v2.0
  - Trimmomatic
  - Trimmomatic 0.39
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# wgs-read-quality-trimming

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality/adapter trimming of raw WES/WGS reads as the first step of the TSNAD v2.0 neoantigen pipeline, using Trimmomatic v0.39 to remove adapter artifacts before alignment.

## When to use

Use when raw tumor-normal WES/WGS FASTQ reads are about to be mapped to a reference genome (GRCh37/GRCh38) and adapter or low-quality artifacts must be removed first.

## When NOT to use

- Input reads are already aligned (BAM/SAM) — trimming applies only to raw reads at the start of the pipeline
- The data is RNA-Seq rather than WES/WGS — TSNAD v2.0 handles RNA-Seq with the separate STAR/Arriba module instead
- Reads are synthetic or pre-cleaned test data with no adapters, where trimming adds no value before alignment

## Inputs

- Raw paired-end WES/WGS FASTQ reads from tumor-normal sample pairs
- Adapter sequence files bundled with Trimmomatic v0.39

## Outputs

- Cleaned (trimmed and cropped) paired-end FASTQ reads ready for BWA v0.7.17 alignment

## How to apply

Run Trimmomatic v0.39 on the raw paired-end WES/WGS reads of each tumor-normal pair as the first module of the TSNAD v2.0 pipeline, before BWA v0.7.17 mapping. Its purpose is to trim and crop the reads and remove adapter artifacts that would otherwise harm downstream alignment, SAM-to-BAM conversion with SAMtools v1.13, GATK v4.2.0.0 base-quality recalibration, and Mutect2 somatic SNV/INDEL calling. Trimmomatic is embedded in the TSNAD Docker image (biopharm/tsnad) and, in the GitHub install, is expected in the Tools/ folder alongside BWA, SAMtools, and other pipeline tools. Because the trimmed reads feed a variant-calling chain, retain both FASTQ mates consistently so the paired structure is preserved for BWA. Verify success downstream: if the trimming was applied correctly, cleaned reads map to GRCh37/GRCh38 and the pipeline ultimately yields a valid somatic_snvs_indels.vcf from Mutect2.

## Related tools

- **Trimmomatic 0.39** (Trims and crops raw WES/WGS reads and removes adapter artifacts before genome mapping) — https://github.com/jiujiezz/tsnad
- **BWA 0.7.17** (Consumes the trimmed reads to map short sequences to GRCh37/GRCh38) — https://github.com/jiujiezz/tsnad

## Examples

```
cd /home/tsnad && bash uncompress.sh && python TSNAD.py -I samples/ -R RNA-seq/ -V grch38 -O results/
```

## Evaluation signals

- Trimmed FASTQ files retain paired-end structure and contain no residual adapter sequences
- Downstream BWA alignment of the cleaned reads to GRCh37 or GRCh38 succeeds and SAMtools converts the output to BAM
- The full chain terminates in a somatic_snvs_indels.vcf that is a valid VCF containing Mutect2-called SNV/INDEL records
- Trimmomatic v0.39 is present and executed from the Tools/ folder (GitHub install) or runs inside the biopharm/tsnad Docker image

## Limitations

- Trimmomatic trimming is specific to WES/WGS reads; RNA-Seq inputs follow a different module (hisat2/StringTie, STAR/Arriba) in TSNAD v2.0
- The GitHub (non-Docker) installation only works on Linux; Docker install is recommended and may take hours to pull due to large image size
- TSNAD v2.0 requires sufficient sequencing depth — minimum 15x WGS / 50x WES, recommended 30x WGS / 100x WES — which trimming alone cannot compensate for

## Evidence
