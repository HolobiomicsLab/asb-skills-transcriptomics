---
name: base-quality-score-recalibration
description: Use when a tumor-normal WES/WGS SAM/BAM alignment (GRCh37 or GRCh38)
  needs base quality score recalibration and repeat-sequence removal before Mutect2
  somatic SNV/INDEL calling.
when_to_use_negative:
- Input is raw FASTQ reads that have not yet been trimmed with Trimmomatic and mapped
  with BWA — recalibration operates on aligned BAM data
- Goal is RNA-Seq gene expression or gene fusion analysis, which TSNAD v2.0 handles
  with hisat2/Stringtie and STAR/Arriba rather than GATK recalibration
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: GATK
  role: Removes repeat sequences and recalibrates base quality scores to create the
    final BAM; also provides the Mutect2 module for downstream SNV/INDEL calling
  repo: https://github.com/broadinstitute/gatk
- name: BWA
  role: Maps Trimmomatic-trimmed short reads to the GRCh37/GRCh38 reference genome,
    producing the SAM that becomes the recalibration input
- name: SAMtools
  role: Converts BWA SAM output to BAM format to save storage space before GATK processing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:521cd30d422dfedadf465a07e20c85364c59ed713b9f49702ab19258cf7da19b
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
  - Mutect2
  - SAMtools
  - TSNAD v2.0
  - Trimmomatic
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# base-quality-score-recalibration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Recalibrates base quality scores in BWA-mapped WES/WGS BAMs using GATK v4.2.0.0 as part of the TSNAD v2.0 pipeline, producing the final BAM that feeds Mutect2 somatic SNV/INDEL calling.

## When to use

Use when a tumor-normal WES/WGS SAM/BAM alignment (GRCh37 or GRCh38) needs base quality score recalibration and repeat-sequence removal before Mutect2 somatic SNV/INDEL calling.

## When NOT to use

- Input is raw FASTQ reads that have not yet been trimmed with Trimmomatic and mapped with BWA — recalibration operates on aligned BAM data
- Goal is RNA-Seq gene expression or gene fusion analysis, which TSNAD v2.0 handles with hisat2/Stringtie and STAR/Arriba rather than GATK recalibration

## Inputs

- BAM (or SAM) alignments of trimmed tumor-normal WES/WGS reads mapped to GRCh37 or GRCh38
- Reference genome GRCh37 or GRCh38
- dbSNP known-sites VCF (dbsnp_138.b37.vcf.gz from the GATK resource bundle)

## Outputs

- Final BAM file with recalibrated base quality scores and repeat sequences removed
- BAM suitable as direct input to GATK Mutect2 for somatic SNV/INDEL calling

## How to apply

In the TSNAD v2.0 workflow, after Trimmomatic v0.39 trims raw reads and BWA v0.7.17 maps them to GRCh37 or GRCh38, SAMtools v1.13 converts SAM to BAM to save storage space, and GATK (v4.2.0.0) is then applied to remove repeat sequences and recalibrate the base quality score to create the final BAM file. The recalibrated BAM for the tumor-normal pair is the direct input to the Mutect2 module of GATK, which writes somatic SNVs/INDELs to somatic_snvs_indels.vcf. The dbSNP bundle (dbsnp_138.b37.vcf.gz from the GATK resource bundle) is fetched as a known-sites resource. Adequate input depth is required: minimum 15x for WGS and 50x for WES, with 30x WGS / 100x WES recommended, since low-depth data degrade the recalibration and calling steps.

## Related tools

- **GATK** (Removes repeat sequences and recalibrates base quality scores to create the final BAM; also provides the Mutect2 module for downstream SNV/INDEL calling) — https://github.com/broadinstitute/gatk
- **BWA** (Maps Trimmomatic-trimmed short reads to the GRCh37/GRCh38 reference genome, producing the SAM that becomes the recalibration input)
- **SAMtools** (Converts BWA SAM output to BAM format to save storage space before GATK processing)

## Evaluation signals

- The final BAM file exists and can be read with SAMtools/GATK without errors
- Base quality scores in the recalibrated BAM differ from the pre-recalibration BAM (recalibration applied)
- The recalibrated tumor-normal BAM pair runs successfully through GATK Mutect2
- somatic_snvs_indels.vcf exists, is a valid VCF, and contains SNV/INDEL records called by Mutect2 from the final BAM
- Input read depth meets the minimum of 15x for WGS and 50x for WES

## Limitations

- TSNAD v1.0 only supports the reference genome GRCh37; GRCh38 support is new in v2.0
- TSNAD requires JAVA 1.8 and Python 2.7, while newer GATK releases require Java 17 — version pinning matters (TSNAD uses GATK 4.2.0.0)
- Low sequencing depth (below 15x WGS / 50x WES) undermines downstream calling quality

## Evidence

- [methods] GATK (v4.2.0.0) [16] is used to remove repeat sequences and recalibrate the base quality score to create the final BAM file: "GATK (v4.2.0.0) [16] is used to remove repeat sequences and recalibrate the base quality score to create the final BAM file"
- [methods] The Mutect2 module of GATK is used to call SNVs/INDELs.: "The Mutect2 module of GATK is used to call SNVs/INDELs."
- [methods] BWA (v0.7.17) [14] is used for mapping short sequences to a reference genome: "BWA (v0.7.17) [14] is used for mapping short sequences to a reference genome"
- [readme] GATK 4.2.0.0 (https://github.com/broadinstitute/gatk/releases/download/4.2.0.0/gatk-4.2.0.0.zip): "[GATK 4.2.0.0](https://github.com/broadinstitute/gatk/releases/download/4.2.0.0/gatk-4.2.0.0.zip)"
- [results] the minimum depth should be 15 × for WGS and 50 × for WES, the recommended depth should be 30 × for WGS and 100 × for WES: "the minimum depth should be 15 × for WGS and 50 × for WES, the recommended depth should be 30 × for WGS and 100 × for WES"
