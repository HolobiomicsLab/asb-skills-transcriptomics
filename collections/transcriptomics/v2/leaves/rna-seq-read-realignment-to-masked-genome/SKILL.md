---
name: rna-seq-read-realignment-to-masked-genome
description: Use when per-allele read counts at heterozygous biallelic SNPs show inflated
  reference allele ratios (e.g. ~51.8–51.9% instead of the expected 50%) and you suspect
  the reference genome biases alignment toward the reference allele.
when_to_use_negative:
- When reference allele ratios are already at the expected ~50% and no reference bias
  is detectable in the informative biallelic SNPs
- When the goal is de novo variant discovery rather than reducing mapping bias for
  allele-specific expression counting — the masking forces alignment to called variant
  alleles
- When samples are homozygous or have very low read counts at candidate SNPs (VADT
  removes such samples; the ratio comparison requires informative biallelic sites)
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: Bedtools
  role: Masks the reference genome by replacing reference bases at positions of variants
    in the global VCF
- name: STAR 2.5.2b
  role: Two-pass re-alignment of all RNA-seq samples to the masked genome with the
    same settings used for the unmasked baseline
- name: Picard 2.8.1
  role: Marks duplicate reads on the re-aligned BAMs before variant re-calling
- name: GATK 3.7
  role: Re-calls variants from masked alignments via Split'N'Trim, Base Recalibration,
    and HaplotypeCaller with the g.vcf setting
- name: VADT
  role: Tests informative biallelic SNPs for reference allele bias and reports reference
    allele count / total allele count for each VCF
  repo: https://github.com/mjtiv/VADT
- name: FastQC
  role: Pre-alignment quality review of all FASTQ files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:304b42d58d880f9954069d8af9cef0944d2962b1b055492e684cd45f101e2c06
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
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# rna-seq-read-realignment-to-masked-genome

## Summary

Reduces reference-allele mapping bias in RNA-seq ASE analyses by masking the reference genome at known variant positions with Bedtools and re-aligning all samples with STAR two-pass, then re-calling variants with GATK and comparing per-tissue reference allele ratios against the unmasked baseline.

## When to use

Use when per-allele read counts at heterozygous biallelic SNPs show inflated reference allele ratios (e.g. ~51.8–51.9% instead of the expected 50%) and you suspect the reference genome biases alignment toward the reference allele.

## When NOT to use

- When reference allele ratios are already at the expected ~50% and no reference bias is detectable in the informative biallelic SNPs
- When the goal is de novo variant discovery rather than reducing mapping bias for allele-specific expression counting — the masking forces alignment to called variant alleles
- When samples are homozygous or have very low read counts at candidate SNPs (VADT removes such samples; the ratio comparison requires informative biallelic sites)

## Inputs

- Raw RNA-seq FASTQ files (e.g. 68 chicken samples from SRA SRP058295 and SRP255211)
- Reference genome build (Gallus_gallus_5.0, Ensembl release 86) and matching GTF index for STAR
- Global VCF of called variants (merged g.vcf files from all study samples plus additional samples) used as masking positions

## Outputs

- Masked reference genome with reference bases replaced at variant positions
- BAM alignments of all samples to the masked genome (STAR two-pass)
- GATK-called VCF/g.vcf files from the masked alignment
- Per-tissue mean reference allele ratios (masked vs unmasked) and the shift toward 50.0%

## How to apply

First establish the unmasked baseline: QC the FASTQs with FastQC, align the 68 RNA-seq samples with STAR 2.5.2b in two-pass mode to the Gallus_gallus_5.0 build (Ensembl release 86 GTF), mark duplicates with Picard 2.8.1, and call variants via GATK 3.7 (Split'N'Trim, Base Recalibration, HaplotypeCaller with the g.vcf setting). Merge all g.vcf files (plus additional samples) into a global VCF, then mask the reference genome with Bedtools by replacing reference bases at every variant position in that VCF. Re-align all samples to the masked genome with identical STAR two-pass settings, repeat duplicate marking and GATK variant calling, and compute the mean reference allele ratio (reference allele count / total allele count) from the raw AD allele depths over informative biallelic SNPs in each tissue. Judge success by whether the masked-reference ratios shift toward the expected 50.0% in every tissue, reproducing the reported decrease from 51.77% to 50.17% in breast muscle, 51.93% to 50.19% in abdominal fat, and 51.91% to 50.17% in liver.

## Related tools

- **Bedtools** (Masks the reference genome by replacing reference bases at positions of variants in the global VCF)
- **STAR 2.5.2b** (Two-pass re-alignment of all RNA-seq samples to the masked genome with the same settings used for the unmasked baseline)
- **Picard 2.8.1** (Marks duplicate reads on the re-aligned BAMs before variant re-calling)
- **GATK 3.7** (Re-calls variants from masked alignments via Split'N'Trim, Base Recalibration, and HaplotypeCaller with the g.vcf setting)
- **VADT** (Tests informative biallelic SNPs for reference allele bias and reports reference allele count / total allele count for each VCF) — https://github.com/mjtiv/VADT
- **FastQC** (Pre-alignment quality review of all FASTQ files)

## Examples

```
bedtools maskfasta -fi Gallus_gallus_5.0.fa -bed global_variants.bed -fo masked_genome.fa; STAR --runMode genomeGenerate --genomeDir masked_index --genomeFastaFiles masked_genome.fa --sjdbGTFfile Gallus_gallus_5.0.gtf; STAR --twopassMode Basic --genomeDir masked_index --readFilesIn sample.fastq.gz --readFilesCommand zcat
```

## Evaluation signals

- Per-tissue mean reference allele ratio after re-alignment moves toward 50%: 51.77→50.17% (breast muscle), 51.93→50.19% (abdominal fat), 51.91→50.17% (liver)
- All samples pass FastQC QC and align with identical STAR two-pass parameters to both masked and unmasked genomes for a controlled comparison
- Reference allele ratios are computed from raw AD allele depths over informative biallelic SNPs in each tissue (mean of reference allele count / total allele count)
- Variant re-calling on the masked alignment completes the full GATK 3.7 pipeline (Split'N'Trim, Base Recalibration, HaplotypeCaller g.vcf) without pipeline changes
- Only informative biallelic SNPs (informative per VADT filtering: non-homozygous, ≥20 reads per sample, quality ≥20) contribute to the ratio comparison

## Limitations

- Mapping issues persisted for muscle tissue samples, likely due to the current genome build and annotation of muscle-related genes; breast muscle samples had a significantly higher percentage of reads flagged by STAR as 'too short' (avg. 15.39% vs 4.50% for abdominal fat and 5.33% for liver)
- Bias reduction depends on the completeness of the variant catalog used for masking — the global VCF was merged from study g.vcf files plus 32 additional samples, so uncalled variant positions remain biased toward the reference allele
- VADT's VCF input was produced by GATK's HaplotypeCaller; VCFs from other variant callers 'could potentially have some bugs'

## Evidence

- [methods] Masking the reference genome with called variants and re-aligning reads: "The global vcf file was then used to mask the reference genome using Bedtools36. All samples were re-aligned to the masked genome"
- [methods] Quantified effect of masking on reference allele ratios: "masked reference genome reduced the reference allele ration from 51.77 to 50.17% in breast muscle, 51.93% to 50.19% in abdominal fat and 51.91% to 50.17% in liver"
- [methods] Unmasked baseline alignment parameters: "aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting29,30 to the Gallus_gallus_5.0 genome build"
- [readme] VADT reference allele bias quantification role: "So, VADT tests all "testable" variants that pass the prior filters for reference allele bias and reports the final total of (reference allele count / total allele count) for a VCF file."
- [readme] Rationale: why reference bias threatens ASE detection: "A concern with detection of ASE variants is reference allele bias where the reference genome causes the reference allele to be mapped more."
