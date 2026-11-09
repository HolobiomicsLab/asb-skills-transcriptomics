---
name: gvcf-merging-into-global-vcf
description: Use when per-sample gVCF outputs from GATK HaplotypeCaller must be combined
  into one multi-sample VCF to support joint downstream analyses such as reference
  masking or allele-specific expression testing.
when_to_use_negative:
- Inputs are already joint-called multi-sample VCFs or genotyping-array VCFs (e.g.
  the 600K Axiom panel data) rather than per-sample g.vcf files
- Variant calls were produced by tools other than GATK HaplotypeCaller, since VADT
  and this merge workflow assume GATK-format g.vcf input and other callers could introduce
  format bugs
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0199
tools:
- name: GATK 3.7
  role: HaplotypeCaller with the g.vcf setting produces the per-sample g.vcf files
    that are merged into the global VCF
- name: STAR 2.5.2b
  role: Two-pass RNA-seq alignment producing the BAMs that enter the GATK pipeline
    upstream of g.vcf generation
- name: Picard 2.8.1
  role: Marks duplicate reads in aligned SAM/BAM files prior to GATK variant calling
- name: Bedtools
  role: Uses the merged global vcf file to mask the reference genome before samples
    are re-aligned
- name: VADT
  role: Downstream ASE detection tool that consumes the GATK-format VCF for biallelic
    SNP binomial testing
  repo: https://github.com/mjtiv/VADT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:3fbe7b65466f7f28af3f8557be2d7b48960e795c8dd00f185d10a6ca1593fa84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Bedtools
  - FastQC
  - GATK 3.7
  - Picard
  - Picard 2.8.1
  - STAR 2.5.2b
  - VADT
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gvcf-merging-into-global-vcf

## Summary

Merges per-sample g.vcf files produced by GATK 3.7 HaplotypeCaller (g.vcf mode) across all samples into a single global multi-sample VCF, which downstream steps (Bedtools genome masking and VADT ASE detection) require as input.

## When to use

Use when per-sample gVCF outputs from GATK HaplotypeCaller must be combined into one multi-sample VCF to support joint downstream analyses such as reference masking or allele-specific expression testing.

## When NOT to use

- Inputs are already joint-called multi-sample VCFs or genotyping-array VCFs (e.g. the 600K Axiom panel data) rather than per-sample g.vcf files
- Variant calls were produced by tools other than GATK HaplotypeCaller, since VADT and this merge workflow assume GATK-format g.vcf input and other callers could introduce format bugs

## Inputs

- Per-sample g.vcf files from GATK 3.7 HaplotypeCaller (g.vcf mode), one per sample (n = 68)
- Aligned BAM files from STAR two-pass alignment with Picard duplicate marking (upstream)

## Outputs

- A single global VCF file (global_variants.vcf) containing variant records from all 68 samples
- A masked reference genome derived from the global VCF via Bedtools (downstream artifact)

## How to apply

After calling variants for each of the 68 samples (paired-end broiler RNA-seq from SRA SRP058295 and SRP255211) with GATK 3.7 HaplotypeCaller run with the g.vcf setting — following STAR 2.5.2b two-pass alignment, Picard 2.8.1 duplicate marking, and the GATK 'Split'N'Trim' and 'Base Recalibration' steps — combine the per-sample g.vcf files into one global VCF file covering all samples. The merged global VCF is then used to mask the reference genome with Bedtools before re-alignment, and VADT consumes VCFs formatted per GATK's specification, so the merge must preserve GATK VCF structure (FILTER column, per-sample genotype fields). Validate that the global VCF parses as a valid VCF and contains variant records attributable to all 68 samples.

## Related tools

- **GATK 3.7** (HaplotypeCaller with the g.vcf setting produces the per-sample g.vcf files that are merged into the global VCF)
- **STAR 2.5.2b** (Two-pass RNA-seq alignment producing the BAMs that enter the GATK pipeline upstream of g.vcf generation)
- **Picard 2.8.1** (Marks duplicate reads in aligned SAM/BAM files prior to GATK variant calling)
- **Bedtools** (Uses the merged global vcf file to mask the reference genome before samples are re-aligned)
- **VADT** (Downstream ASE detection tool that consumes the GATK-format VCF for biallelic SNP binomial testing) — https://github.com/mjtiv/VADT

## Evaluation signals

- global_variants.vcf exists and parses as a valid VCF per the GATK VCF specification
- Variant records in the global VCF are attributable to all 68 samples (n = 68)
- STAR alignment statistics for the underlying samples are consistent with the paper's reported average unique mapping rate of 87.24%
- The FILTER column in the merged VCF retains GATK flags so VADT can remove variants marked as failing ('PASS' check)
- The global VCF can be applied with Bedtools to mask the reference genome and reduce the reference allele ratio toward ~50% (e.g. 51.77% to 50.17% in breast muscle)

## Limitations

- VADT notes that VCF files it uses were produced by GATK's HaplotypeCaller, so VCFs from other callers merged into a global file 'could potentially have some bugs'
- Mapping issues with breast muscle tissue samples (avg. 15.39% of reads 'too short' per STAR) limit variant-call quality for those samples
- Low overlap between RNA-seq variant calls and the 600K chicken genotyping panel means many merged SNPs cannot be externally validated

## Evidence

- [methods] Merge g.vcf files into a global vcf file: [evidence span withheld — anchor status: not_found]
- [methods] GATK pipeline producing per-sample g.vcfs: "which includes “Split’N’Trim,” “Base Recalibration” and variant calling using HaplotypeCaller (with the g.vcf setting)"
- [methods] Global VCF used for masking: "The global vcf file was then used to mask the reference genome using Bedtools36"
- [readme] VADT expects GATK HaplotypeCaller VCFs: "VCF files utilized by VADT were produced by GATK's Haplotypecaller and so VCF files produced by other programs could potentially have some bugs."
- [abstract] n = 68 subset variant calling: "using variants called from a large sub-set of the samples (n = 68)"
