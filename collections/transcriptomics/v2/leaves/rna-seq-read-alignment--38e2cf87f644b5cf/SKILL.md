---
name: rna-seq-read-alignment
description: Use when raw paired-end RNA-seq FASTQs from chicken (or other species)
  must be spliced-aligned for downstream per-sample variant calling (g.vcf/HaplotypeCaller)
  or allele-specific expression analysis.
when_to_use_negative:
- Input data are already aligned BAM files — the next step in this workflow is duplicate
  marking with Picard 2.8.1 and GATK 3.7 processing, not re-alignment
- The goal is ASE detection from an existing multi-sample VCF — use VADT (github.com/mjtiv/VADT)
  directly rather than re-aligning reads
- Reads are small-RNA or single-end libraries where spliced two-pass alignment of
  paired reads against a full genome build is not appropriate
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: STAR 2.5.2b
  role: Two-pass spliced read alignment of paired-end RNA-seq reads to the Gallus_gallus_5.0
    genome with soft-clipping enabled
- name: Picard 2.8.1
  role: Marks duplicate reads in STAR output SAM files and converts SAM to BAM prior
    to GATK 3.7 processing
- name: FastQC
  role: Pre-alignment sequence-quality review of all FASTQ files
- name: RSeQC 2.6.4
  role: geneBody_coverage.py used to assess gene coverage of aligned samples
- name: Bedtools
  role: Masks the reference genome using the global VCF so samples can be re-aligned
    to reduce reference allele bias
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:e5674a81a321fddc54e35a48cdef70018a5fac58a7467026ece83ff3626a3f50
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bedtools
  - FastQC
  - Picard
  - Picard 2.8.1
  - RSeQC 2.6.4
  - STAR 2.5.2b
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# rna-seq-read-alignment

## Summary

Align paired-end broiler RNA-seq reads with STAR 2.5.2b in two-pass mode against the Gallus_gallus_5.0 build, then mark duplicates with Picard 2.8.1, producing aligned BAMs ready for GATK HaplotypeCaller variant calling and ASE analysis.

## When to use

Use when raw paired-end RNA-seq FASTQs from chicken (or other species) must be spliced-aligned for downstream per-sample variant calling (g.vcf/HaplotypeCaller) or allele-specific expression analysis.

## When NOT to use

- Input data are already aligned BAM files — the next step in this workflow is duplicate marking with Picard 2.8.1 and GATK 3.7 processing, not re-alignment
- The goal is ASE detection from an existing multi-sample VCF — use VADT (github.com/mjtiv/VADT) directly rather than re-aligning reads
- Reads are small-RNA or single-end libraries where spliced two-pass alignment of paired reads against a full genome build is not appropriate

## Inputs

- Paired-end RNA-seq FASTQ files (e.g., 68 broiler samples from SRA SRP058295 abdominal fat and SRP255211 breast muscle/liver)
- Reference genome build (Gallus_gallus_5.0) and Ensembl GTF annotation (release 86) for STAR index construction

## Outputs

- Aligned BAM files with duplicate reads marked (Picard)
- STAR alignment statistics per sample (unique mapping rate, 'too short' read fraction)
- Gene-body coverage reports (RSeQC geneBody_coverage.py)

## How to apply

First review every FASTQ for sequence quality with FastQC; all samples in the broiler study passed this QC gate. Build a STAR index from the Gallus_gallus_5.0 genome together with its Ensembl release 86 GTF annotation, then align reads with STAR 2.5.2b using the two-pass setting, permitting soft-clipping of adapters/low-quality ends instead of trimming the reads. Mark duplicate reads in the resulting SAM files with Picard 2.8.1 (converting SAM to BAM) before submitting aligned reads to the GATK 3.7 pipeline (Split'N'Trim, Base Recalibration, HaplotypeCaller g.vcf mode). The two-pass STAR mode improves splice-junction discovery on a per-sample basis, which matters because junction accuracy feeds directly into downstream variant calls. After the first alignment round, the global VCF can be used with Bedtools to mask the reference genome and samples re-aligned to the masked genome to reduce reference allele bias before ASE testing. Gene coverage across the transcript body is assessed with RSeQC 2.6.4 geneBody_coverage.py as an alignment-quality check.

## Related tools

- **STAR 2.5.2b** (Two-pass spliced read alignment of paired-end RNA-seq reads to the Gallus_gallus_5.0 genome with soft-clipping enabled)
- **Picard 2.8.1** (Marks duplicate reads in STAR output SAM files and converts SAM to BAM prior to GATK 3.7 processing)
- **FastQC** (Pre-alignment sequence-quality review of all FASTQ files)
- **RSeQC 2.6.4** (geneBody_coverage.py used to assess gene coverage of aligned samples)
- **Bedtools** (Masks the reference genome using the global VCF so samples can be re-aligned to reduce reference allele bias)

## Examples

```
STAR --runMode alignReads --twoPassMode Basic --genomeDir Gallus_gallus_5.0_STARindex --readFilesIn sample_R1.fastq sample_R2.fastq --outFileNamePrefix sample_ && picard MarkDuplicates I=sample_Aligned.out.sam O=sample_markdup.bam M=sample_dup_metrics.txt
```

## Evaluation signals

- All FASTQ files pass FastQC quality review before alignment is attempted
- Average unique mapping rate across samples should approach the paper's reported value of 87.24%
- Per-sample STAR metrics: 'too short' read fraction should be compared across tissues (breast muscle avg. 15.39% vs. abdominal fat avg. 4.50% and liver avg. 5.33% in the study) to flag tissue-specific mapping problems
- RSeQC geneBody_coverage.py output shows reasonably uniform gene-body coverage for each sample
- After masking and re-alignment, the reference allele ratio moves toward 50% (e.g., 51.77% → 50.17% in breast muscle, 51.93% → 50.19% in abdominal fat, 51.91% → 50.17% in liver)

## Limitations

- Muscle tissue samples exhibited mapping problems attributed to the current genome build and annotation of muscle-related genes, described by the authors as the biggest problem in the entire pipeline
- Breast muscle samples had a significantly higher percentage of reads flagged by STAR as 'too short' (avg. 15.39%) than abdominal fat (avg. 4.50%) or liver (avg. 5.33%)
- Aligning to the unmasked reference introduces reference allele bias; re-alignment to a masked genome is required before ASE testing

## Evidence

- [methods] STAR two-pass alignment to Gallus_gallus_5.0: "aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting"
- [methods] Duplicate marking with Picard: "duplicate reads were marked using Picard (version 2.8.1)35"
- [methods] FastQC pre-alignment QC: "All fastq files were reviewed for sequence quality using FastQC24 and all samples passed this quality control step"
- [methods] Tissue-specific 'too short' read rates: "breast muscle samples had a significantly higher percentage of reads considered “too short” according to STAR (avg. 15.39% compared to avg. 4.50%"
- [other] Pipeline architecture from source card: [evidence span withheld — anchor status: not_found]
- [methods] Masked-genome re-alignment rationale: "masked reference genome reduced the reference allele ration from 51.77 to 50.17% in breast muscle, 51.93% to 50.19% in abdominal fat"
