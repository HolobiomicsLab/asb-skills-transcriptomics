---
name: short-read-reference-alignment
description: Use when raw tumor-normal WES/WGS FASTQ reads have been adapter-trimmed
  and must be aligned to a human reference genome to produce BAM files for downstream
  somatic SNV/INDEL calling.
when_to_use_negative:
- Input is RNA-Seq data intended for fusion detection — TSNAD v2.0 aligns those reads
  with STAR v2.7 (hisat2 2.2.1 for expression), not BWA
- Reads are long-read sequences rather than short WES/WGS reads, which BWA v0.7.17
  short-read mapping targets
- The task only requires neoantigen prediction from a pre-existing VCF, so no re-alignment
  is needed
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3673
tools:
- name: BWA 0.7.17
  role: Maps the Trimmomatic-cleaned short WES/WGS reads to the GRCh37 or GRCh38 reference
    genome
- name: SAMtools 1.13
  role: Converts the BWA SAM alignment output to BAM format to save storage space
- name: GATK 4.2.0.0
  role: Removes repeat sequences and recalibrates base quality scores on the aligned
    BAM to create the final BAM
  repo: https://github.com/broadinstitute/gatk
- name: Trimmomatic 0.39
  role: Upstream trimming/cropping of raw reads to remove adapter artifacts harmful
    to mapping
- name: TSNAD v2.0
  role: Integrated pipeline that orchestrates the alignment and downstream variant
    calling
  repo: https://github.com/jiujiezz/tsnad
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:5f2813946c6f93c85c649126e7e8cd65f95193db744bb54f4d4bfbf2f49935f2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jiujiezz/tsnad
  repo_url: https://github.com/broadinstitute/gatk
  tools:
  - BWA
  - BWA 0.7.17
  - GATK
  - GATK 4.2.0.0
  - Mutect2
  - SAMtools
  - SAMtools 1.13
  - TSNAD v2.0
  - Trimmomatic
  - Trimmomatic 0.39
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# short-read-reference-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps cleaned short sequencing reads (WES/WGS) to a reference genome (GRCh37 or GRCh38) using BWA v0.7.17, as the alignment step of the TSNAD v2.0 somatic variant calling pipeline.

## When to use

Use when raw tumor-normal WES/WGS FASTQ reads have been adapter-trimmed and must be aligned to a human reference genome to produce BAM files for downstream somatic SNV/INDEL calling.

## When NOT to use

- Input is RNA-Seq data intended for fusion detection — TSNAD v2.0 aligns those reads with STAR v2.7 (hisat2 2.2.1 for expression), not BWA
- Reads are long-read sequences rather than short WES/WGS reads, which BWA v0.7.17 short-read mapping targets
- The task only requires neoantigen prediction from a pre-existing VCF, so no re-alignment is needed

## Inputs

- Adapter-trimmed paired-end WES/WGS FASTQ reads of a tumor-normal pair (output of Trimmomatic 0.39)
- Reference genome GRCh37 or GRCh38

## Outputs

- SAM alignment file per sample
- BAM alignment file per sample (after SAMtools 1.13 conversion)
- Final BAM after GATK 4.2.0.0 repeat removal and base quality score recalibration

## How to apply

After Trimmomatic v0.39 removes adapter artifacts from the raw WES/WGS reads of the tumor-normal pair, map the cleaned short reads to the reference genome (GRCh37 or GRCh38, matching the reference selected for downstream neoantigen identification) using BWA v0.7.17. Convert the resulting SAM output to BAM format with SAMtools v1.13 to save storage space. The BAM then undergoes repeat removal and base quality score recalibration by GATK v4.2.0.0, producing the final BAM from which Mutect2 calls somatic SNVs/INDELs into somatic_snvs_indels.vcf. Choosing GRCh37 versus GRCh38 is a decision point: TSNAD v1.0 only supported GRCh37, while v2.0 allows selection of GRCh38 when identifying neoantigens. Validation confirms the alignment chain produced a valid final BAM that Mutect2 can consume to yield a valid VCF containing SNV/INDEL records.

## Related tools

- **BWA 0.7.17** (Maps the Trimmomatic-cleaned short WES/WGS reads to the GRCh37 or GRCh38 reference genome)
- **SAMtools 1.13** (Converts the BWA SAM alignment output to BAM format to save storage space)
- **GATK 4.2.0.0** (Removes repeat sequences and recalibrates base quality scores on the aligned BAM to create the final BAM) — https://github.com/broadinstitute/gatk
- **Trimmomatic 0.39** (Upstream trimming/cropping of raw reads to remove adapter artifacts harmful to mapping)
- **TSNAD v2.0** (Integrated pipeline that orchestrates the alignment and downstream variant calling) — https://github.com/jiujiezz/tsnad

## Examples

```
docker run -it -v [dir of WES/WGS]/:/home/tsnad/samples -v [output dir]:/home/tsnad/results biopharm/tsnad:latest /bin/bash; cd /home/tsnad; python TSNAD.py -I samples/ -V grch37 -O results/
```

## Evaluation signals

- The SAM output converts cleanly to BAM with SAMtools 1.13 and passes BAM validation
- The final GATK-processed BAM (post repeat removal and base quality recalibration) exists and is a valid BAM
- Mutect2 succeeds on the tumor-normal BAM pair and produces somatic_snvs_indels.vcf that is a valid VCF containing SNV/INDEL records
- Mapped read coordinates are consistent with the declared reference build (GRCh37 vs GRCh38) so downstream neoantigen identification uses the matching genome version
- Sequencing depth of aligned data meets the minimum 15x for WGS and 50x for WES (recommended 30x WGS / 100x WES)

## Limitations

- Read depth matters: minimum 15x for WGS and 50x for WES, with recommended depth 30x for WGS and 100x for WES
- Reference build must match downstream neoantigen identification; TSNAD v1.0 supported only GRCh37
- Un-trimmed adapter artifacts in raw reads harm downstream processing, so trimming must precede alignment

## Evidence
