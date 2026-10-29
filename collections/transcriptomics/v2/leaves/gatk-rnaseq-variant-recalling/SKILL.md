---
name: gatk-rnaseq-variant-recalling
description: Use when RNA-seq FASTQs (e.g., chicken SRP058295/SRP255211 samples) need
  per-sample variant calls in g.vcf format for downstream ASE analysis, or when reference
  allele bias must be reduced by re-calling variants against a masked genome.
when_to_use_negative:
- Input is DNA whole-genome sequencing data rather than RNA-seq reads requiring the
  Split'N'Trim RNA-seq GATK best-practices workflow
- The reference genome has already been masked at variant positions and reference
  allele ratios are already near 50%, so re-masking adds no benefit
- Variants cannot be called from a sufficiently large sample subset (the paper merged
  g.vcfs from n = 68 samples plus 32 additional samples to build the global VCF)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: GATK 3.7
  role: Runs the Split'N'Trim, Base Recalibration, and HaplotypeCaller (g.vcf setting)
    steps for variant calling
- name: STAR 2.5.2b
  role: Two-pass alignment of RNA-seq reads to the Gallus_gallus_5.0 genome build,
    before and after masking
- name: Picard 2.8.1
  role: Marks duplicate reads after STAR alignment and before GATK processing
- name: Bedtools
  role: Masks the reference genome using variant positions from the global VCF prior
    to re-alignment
- name: FastQC
  role: Pre-alignment quality review of all FASTQ files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:308a80f58386121c126c35ae289e931d57bd3fa0b5be952158782d2d714e2307
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bedtools
  - FastQC
  - GATK
  - GATK 3.7
  - Picard
  - Picard 2.8.1
  - STAR 2.5.2b
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gatk-rnaseq-variant-recalling

## Summary

A GATK 3.7 RNA-seq variant-calling pipeline (Split'N'Trim, Base Recalibration, HaplotypeCaller with g.vcf setting) applied to STAR-aligned reads, optionally repeated after re-aligning to a Bedtools-masked reference genome to reduce reference-allele mapping bias.

## When to use

Use when RNA-seq FASTQs (e.g., chicken SRP058295/SRP255211 samples) need per-sample variant calls in g.vcf format for downstream ASE analysis, or when reference allele bias must be reduced by re-calling variants against a masked genome.

## When NOT to use

- Input is DNA whole-genome sequencing data rather than RNA-seq reads requiring the Split'N'Trim RNA-seq GATK best-practices workflow
- The reference genome has already been masked at variant positions and reference allele ratios are already near 50%, so re-masking adds no benefit
- Variants cannot be called from a sufficiently large sample subset (the paper merged g.vcfs from n = 68 samples plus 32 additional samples to build the global VCF)

## Inputs

- FASTQ files from RNA-seq samples (e.g., SRA SRP058295 abdominal fat, SRP255211 breast muscle/liver)
- Gallus_gallus_5.0 reference genome build (Ensembl release 86) with Ensembl GTF release 86 for the STAR index
- Optional: global VCF of variant positions for Bedtools genome masking

## Outputs

- Per-sample g.vcf files from HaplotypeCaller
- Merged global VCF file from all samples
- Recalibrated BAM files (duplicate-marked, base-recalibrated)
- Per-tissue reference allele ratios (reference allele count / total allele count) for masked vs. unmasked alignments

## How to apply

First review FASTQ quality with FastQC, then align reads with STAR 2.5.2b in two-pass mode to the Gallus_gallus_5.0 build (Ensembl release 86 GTF) and mark duplicates with Picard 2.8.1. Submit aligned reads to the GATK 3.7 pipeline: run "Split'N'Trim", then Base Recalibration, then variant calling with HaplotypeCaller using the g.vcf setting, producing per-sample g.vcf files. Merge all g.vcf files into a global VCF, optionally mask the reference genome at variant positions with Bedtools, and re-align/re-call against the masked genome to shift reference allele ratios toward the expected 50%. Filter called variants with GATK hard filters (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100) before ASE detection in VADT.

## Related tools

- **GATK 3.7** (Runs the Split'N'Trim, Base Recalibration, and HaplotypeCaller (g.vcf setting) steps for variant calling)
- **STAR 2.5.2b** (Two-pass alignment of RNA-seq reads to the Gallus_gallus_5.0 genome build, before and after masking)
- **Picard 2.8.1** (Marks duplicate reads after STAR alignment and before GATK processing)
- **Bedtools** (Masks the reference genome using variant positions from the global VCF prior to re-alignment)
- **FastQC** (Pre-alignment quality review of all FASTQ files)

## Evaluation signals

- After re-alignment to the masked genome, per-tissue mean reference allele ratios should shift toward 50%: 51.77→50.17% in breast muscle, 51.93→50.19% in abdominal fat, 51.91→50.17% in liver
- All FASTQ files pass FastQC quality control before alignment
- HaplotypeCaller is run with the g.vcf setting so per-sample g.vcfs can be merged into a single global VCF
- GATK variant filtering removes variants with Fisher Strand > 30.0, Quality Depth < 2.0, and Depth < 100
- STAR alignment logs show acceptable 'too short' read percentages (breast muscle averaging ~15.39% is a known flag worth inspecting)

## Limitations

- Breast muscle samples showed a significantly higher percentage of reads flagged as 'too short' by STAR (avg. 15.39% vs. 4.50% abdominal fat and 5.33% liver), indicating mapping issues likely tied to the current genome build
- Reference allele bias reduction depends on variant call completeness; the paper notes low overlap between RNA-seq variant calls and the 600K chicken genotyping panel due to commercial panel limitations
- VCFs from callers other than GATK HaplotypeCaller could produce bugs in downstream VADT processing, as VADT was built around GATK-formatted VCFs

## Evidence

- [methods] Two-pass STAR alignment and GATK pipeline: "aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting29,30 to the Gallus_gallus_5.0 genome build"
- [methods] GATK pipeline steps: "which includes “Split’N’Trim,” “Base Recalibration” and variant calling using HaplotypeCaller (with the g.vcf setting)"
- [methods] Masking and re-alignment: "The global vcf file was then used to mask the reference genome using Bedtools36. All samples were re-aligned to the masked genome"
- [methods] Masking reduces reference allele ratio: "masked reference genome reduced the reference allele ration from 51.77 to 50.17% in breast muscle, 51.93% to 50.19% in abdominal fat and 51.91% to 50.17% in liver"
- [methods] GATK hard filters: "using GATK to remove variants with “Fisher Strand > 30.0”, “Quality Depth < 2.0” and “Depth < 100”"
- [methods] Muscle mapping issue: "breast muscle samples had a significantly higher percentage of reads considered “too short” according to STAR (avg. 15.39% compared to avg. 4.50% for abdominal fat and avg 5.33% for liver"
- [readme] VADT expects GATK-formatted VCFs: "VCF files utilized by VADT were produced by GATK's Haplotypecaller and so VCF files produced by other programs could potentially have some bugs."
