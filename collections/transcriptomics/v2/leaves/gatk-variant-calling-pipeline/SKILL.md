---
name: gatk-variant-calling-pipeline
description: Use when RNA-seq FASTQ files (e.g., paired-end broiler samples from SRA
  SRP058295/SRP255211) need variant calls in VCF/g.vcf format to support downstream
  per-sample ASE or genotype-concordance analyses.
when_to_use_negative:
- Input is DNA-seq rather than RNA-seq data — the STAR two-pass transcriptome-aware
  alignment and Split'N'Trim steps are RNA-seq specific
- A global VCF already exists; this skill should not be re-run before ASE analysis
  (e.g., VADT), which consumes the merged VCF directly
- Samples are not from Gallus gallus / the aligned genome build — STAR index, GTF,
  and masked reference must match the organism's assembly
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: FastQC
  role: Sequence quality screening of all FASTQ files before alignment (all samples
    passed this QC step)
- name: STAR 2.5.2b
  role: Two-pass splice-aware alignment of reads to the Gallus_gallus genome build
    with Ensembl GTF index
- name: Picard 2.8.1
  role: Marks duplicate reads and converts SAM to BAM prior to GATK processing
- name: GATK 3.7
  role: Runs Split'N'Trim, Base Recalibration, and HaplotypeCaller in g.vcf mode,
    then merges per-sample g.vcf files into the global VCF
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:e2bac566702d1d30cae7f5bfe12a4605bb97aa0c1834201d2761ee23aaf4d65c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
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
yes_rate: 1.0
n_claims: 12
---

# gatk-variant-calling-pipeline

## Summary

Reconstructs the RNA-seq variant-calling stage used in the broiler ASE study: STAR 2.5.2b two-pass alignment, Picard 2.8.1 duplicate marking, and GATK 3.7 Split'N'Trim + Base Recalibration + HaplotypeCaller (g.vcf mode), followed by merging per-sample g.vcf files into a global VCF for 68 samples.

## When to use

Use when RNA-seq FASTQ files (e.g., paired-end broiler samples from SRA SRP058295/SRP255211) need variant calls in VCF/g.vcf format to support downstream per-sample ASE or genotype-concordance analyses.

## When NOT to use

- Input is DNA-seq rather than RNA-seq data — the STAR two-pass transcriptome-aware alignment and Split'N'Trim steps are RNA-seq specific
- A global VCF already exists; this skill should not be re-run before ASE analysis (e.g., VADT), which consumes the merged VCF directly
- Samples are not from Gallus gallus / the aligned genome build — STAR index, GTF, and masked reference must match the organism's assembly

## Inputs

- Paired-end RNA-seq FASTQ files (68 broiler samples from SRA SRP058295 abdominal fat and SRP255211 breast muscle/liver)
- Gallus_gallus reference genome build (Gallus_gallus_5.0 per article; GTF from Ensembl release 86)
- Ensembl GTF annotation file for STAR index construction

## Outputs

- Per-sample g.vcf files from GATK 3.7 HaplotypeCaller (g.vcf setting)
- A single merged global VCF containing variant records for all 68 samples
- STAR alignment statistics (unique mapping rate, 'too short' read percentages)

## How to apply

Download paired-end FASTQ files and screen them with FastQC; build a STAR index from the Gallus_gallus genome build with its Ensembl GTF annotation (release 86 GTF was initially used) and align with STAR 2.5.2b in two-pass mode, permitting soft-clipping instead of trimming reads. Mark duplicate reads in the SAM/BAM output with Picard 2.8.1 (converting SAM to BAM). Run the GATK 3.7 steps 'Split'N'Trim' and 'Base Recalibration' on the aligned BAMs, then call variants per sample with HaplotypeCaller in g.vcf mode. Finally, merge the per-sample g.vcf files into one global VCF, which in this study fed reference-genome masking via Bedtools and VADT-based ASE testing. Rationale: HaplotypeCaller g.vcf output per sample enables joint merging across tissues, and the global VCF is the expected input for VADT, whose authors note VCFs 'were produced by GATK's Haplotypecaller'.

## Related tools

- **FastQC** (Sequence quality screening of all FASTQ files before alignment (all samples passed this QC step))
- **STAR 2.5.2b** (Two-pass splice-aware alignment of reads to the Gallus_gallus genome build with Ensembl GTF index)
- **Picard 2.8.1** (Marks duplicate reads and converts SAM to BAM prior to GATK processing)
- **GATK 3.7** (Runs Split'N'Trim, Base Recalibration, and HaplotypeCaller in g.vcf mode, then merges per-sample g.vcf files into the global VCF)

## Examples

```
STAR --runMode genomeGenerate --genomeDir gg_index --sjdbGTFfile Gallus_gallus.gtf; STAR --twopassMode Basic --genomeDir gg_index --readFilesIn sample.fq.gz --readFilesCommand zcat; picard MarkDuplicates I=Aligned.sam O=dedup.bam; gatk SplitNCigarReads -R gg.fa -I dedup.bam; gatk HaplotypeCaller -R gg.fa -I dedup.bam -O sample.g.vcf -ERC GVCF
```

## Evaluation signals

- Global VCF file exists and is parseable, containing variant records from all 68 samples
- STAR alignment statistics are consistent with the paper's reported average unique mapping rate of 87.24%
- Per-sample g.vcf files are present for every sample before merging, all in GATK g.vcf format
- Tissue-level STAR flags are plausible: breast muscle samples show a higher 'too short' read percentage (avg. 15.39% vs 4.50% abdominal fat and 5.33% liver)
- The merged VCF downstream supports ~174,000 SNPs per tissue passing VADT filtering, matching the study's reported scale

## Limitations

- Mapping issues with breast muscle tissue samples were the biggest pipeline problem, likely due to the current genome build and annotation of muscle-related genes
- HaplotypeCaller-formatted VCFs are expected by downstream tools; VCFs from other callers could trigger bugs in VADT
- A large number of SNPs were excluded due to low overlap with the 600K chicken genotyping panel, attributed to limitations of commercial genotyping panels

## Evidence

- [methods] STAR two-pass alignment: "aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting29,30 to the Gallus_gallus_5.0 genome build"
- [methods] Picard duplicate marking: "duplicate reads were marked using Picard (version 2.8.1)35"
- [methods] GATK pipeline steps: "which includes “Split’N’Trim,” “Base Recalibration” and variant calling using HaplotypeCaller (with the g.vcf setting)"
- [methods] Global VCF merge: [evidence span withheld — anchor status: not_found]
- [readme] VADT expects GATK VCFs: "VCF files utilized by VADT were produced by GATK's Haplotypecaller and so VCF files produced by other programs could potentially have some bugs"
- [discussion] Muscle mapping issues: "The biggest problem we encountered with our entire analysis pipeline was mapping issues with muscle tissue samples"
