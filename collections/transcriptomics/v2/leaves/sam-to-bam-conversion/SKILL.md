---
name: sam-to-bam-conversion
description: Use when raw WES/WGS reads of a tumor-normal pair have just been aligned
  to GRCh37/GRCh38 with BWA v0.7.17 and the SAM output must be converted to BAM before
  GATK v4.2.0.0 processing and Mutect2 variant calling.
when_to_use_negative:
- Input is already a BAM or CRAM alignment — converting SAM to BAM only applies when
  the aligner (BWA v0.7.17) has just emitted SAM text output.
- The goal is RNA-Seq expression quantification — TSNAD's RNA-Seq module uses HISAT2
  v2.2.1 alignments with SAMtools for sorted BAM, but this skill is scoped to the
  WES/WGS somatic-variant (Trimmomatic + BWA + GATK Mutect2) module.
- Sequencing depth is below the stated minimums (15x WGS / 50x WES) — the variant-calling
  pipeline this conversion feeds is not recommended for such data.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SAMtools
  role: Performs the SAM-to-BAM conversion (v1.13) of BWA alignment output to save
    storage space in the TSNAD v2.0 WES/WGS module
- name: BWA
  role: Produces the SAM alignment input (v0.7.17) by mapping Trimmomatic-trimmed
    short reads to the GRCh37/GRCh38 reference genome
- name: GATK
  role: 'Consumes the BAM downstream: removes repeat sequences, recalibrates base
    quality scores to create the final BAM, and calls SNVs/INDELs with Mutect2'
  repo: https://github.com/broadinstitute/gatk
- name: TSNAD v2.0
  role: One-stop pipeline that embeds SAMtools 1.13 in Tools/ and orchestrates the
    trim-map-convert-call workflow from WES/WGS tumor-normal data
  repo: https://github.com/jiujiezz/tsnad
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:2313fdc4e81c01a006ea6be63b52fa8c6bcef1df67b814acd63c50861bf8ce88
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
  - GATK
  - HISAT2
  - Mutect2
  - SAMtools
  - StringTie
  - TSNAD v2.0
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# SAM-to-BAM conversion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Convert BWA alignment output from SAM to (sorted) BAM with SAMtools v1.13 as an intermediate step of the TSNAD v2.0 neoantigen pipeline, so that downstream GATK repeat-removal, base-quality-score recalibration, and Mutect2 SNV/INDEL calling operate on a compact, standard binary alignment format.

## When to use

Use when raw WES/WGS reads of a tumor-normal pair have just been aligned to GRCh37/GRCh38 with BWA v0.7.17 and the SAM output must be converted to BAM before GATK v4.2.0.0 processing and Mutect2 variant calling.

## When NOT to use

- Input is already a BAM or CRAM alignment — converting SAM to BAM only applies when the aligner (BWA v0.7.17) has just emitted SAM text output.
- The goal is RNA-Seq expression quantification — TSNAD's RNA-Seq module uses HISAT2 v2.2.1 alignments with SAMtools for sorted BAM, but this skill is scoped to the WES/WGS somatic-variant (Trimmomatic + BWA + GATK Mutect2) module.
- Sequencing depth is below the stated minimums (15x WGS / 50x WES) — the variant-calling pipeline this conversion feeds is not recommended for such data.

## Inputs

- SAM alignment file produced by BWA v0.7.17 mapping of Trimmomatic-trimmed WES/WGS reads to GRCh37 or GRCh38
- Reference genome FASTA/index files (GRCh37 or GRCh38)

## Outputs

- BAM (binary alignment) file, subsequently processed by GATK v4.2.0.0 into the final recalibrated BAM used by Mutect2

## How to apply

In the TSNAD v2.0 workflow, trim and crop raw WES/WGS reads with Trimmomatic v0.39, map the cleaned short reads to the reference genome (GRCh37 or GRCh38) with BWA v0.7.17, then convert the resulting SAM alignment to BAM format using SAMtools v1.13 to save storage space. The BAM produced here is not the final analysis artifact: GATK v4.2.0.0 subsequently removes repeat sequences and recalibrates base quality scores to create the final BAM file, from which the Mutect2 module calls somatic SNVs/INDELs into somatic_snvs_indels.vcf. SAMtools 1.13 is bundled in the Tools/ folder of the TSNAD repository (and in the biopharm/tsnad Docker image), so invoke the bundled version for version consistency. Judging correctness: the BAM should be a valid binary alignment of the same reads as the input SAM, compatible with GATK processing and with the recommended WGS/WES depth minimums (15x/50x, recommended 30x/100x).

## Related tools

- **SAMtools** (Performs the SAM-to-BAM conversion (v1.13) of BWA alignment output to save storage space in the TSNAD v2.0 WES/WGS module)
- **BWA** (Produces the SAM alignment input (v0.7.17) by mapping Trimmomatic-trimmed short reads to the GRCh37/GRCh38 reference genome)
- **GATK** (Consumes the BAM downstream: removes repeat sequences, recalibrates base quality scores to create the final BAM, and calls SNVs/INDELs with Mutect2) — https://github.com/broadinstitute/gatk
- **TSNAD v2.0** (One-stop pipeline that embeds SAMtools 1.13 in Tools/ and orchestrates the trim-map-convert-call workflow from WES/WGS tumor-normal data) — https://github.com/jiujiezz/tsnad

## Evaluation signals

- The output is a valid, parseable BAM (binary) file containing the same aligned reads as the input SAM from BWA v0.7.17.
- The BAM can be processed by GATK v4.2.0.0 (repeat removal and base quality score recalibration) without format errors.
- The downstream final BAM supports Mutect2 somatic SNV/INDEL calling that yields a valid somatic_snvs_indels.vcf with SNV/INDEL records.
- Storage footprint of the converted BAM is substantially smaller than the source SAM, consistent with the stated purpose 'to save storage space'.
- The bundled SAMtools version 1.13 (In Tools/ of the TSNAD repo) is used, matching the pipeline's version pinning of Trimmomatic 0.39, BWA 0.7.17, and GATK 4.2.0.0.

## Limitations

- SAM-to-BAM conversion alone does not produce the final analysis-ready BAM: GATK v4.2.0.0 must additionally remove repeat sequences and recalibrate base quality scores before Mutect2 calling.
- The conversion is only as good as the upstream alignment; TSNAD v1.0 supported only GRCh37, whereas v2.0 supports both GRCh37 and GRCh38 — using a mismatched reference breaks downstream steps.
- Minimum sequencing depth should be 15x for WGS and 50x for WES (recommended 30x/100x); lower depth data will yield unreliable variant calls regardless of correct format conversion.

## Evidence
