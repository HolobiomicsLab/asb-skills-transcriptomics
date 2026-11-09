---
name: reference-allele-bias-quantification
description: Use when analyzing allele-specific expression from GATK HaplotypeCaller
  VCFs and you need to measure or mitigate reference allele mapping bias in per-tissue
  reference allele ratios before trusting ASE calls.
when_to_use_negative:
- Variants are multiallelic or within 75 bp of an indel — VADT excludes these from
  testable variants, so ratios would not be informative
- Variant calls were not produced by GATK's HaplotypeCaller — the VADT README warns
  VCFs from other programs 'could potentially have some bugs'
- The goal is de novo SNP discovery or genotyping rather than assessing mapping bias
  in allele ratios from RNA-seq ASE analysis
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Tests all 'testable' biallelic variants passing filters for reference allele
    bias and reports the total (reference allele count / total allele count) per VCF
  repo: https://github.com/mjtiv/VADT
- name: Bedtools
  role: Masks the reference genome by replacing reference bases at positions of variants
    in the global VCF
- name: STAR 2.5.2b
  role: Aligns reads (two-pass) to both unmasked and masked Gallus_gallus_5.0 genome
    builds for the bias comparison
- name: GATK 3.7
  role: Runs Split'N'Trim, Base Recalibration, and HaplotypeCaller (g.vcf setting)
    to call variants from both alignments
- name: Picard 2.8.1
  role: Marks duplicate reads after alignment to either genome build
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:74983e08c4b61586c8fe133594ca02e50e917e5a57899e513aaf4d3f91bc7c54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Bedtools
  - GATK 3.7
  - Picard
  - Picard 2.8.1
  - STAR 2.5.2b
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# reference-allele-bias-quantification

## Summary

Quantify reference allele mapping bias in heterozygous biallelic SNP calls from RNA-seq, reporting the mean reference allele ratio (reference allele count / total allele count) per tissue, and optionally reduce the bias by re-aligning reads to a Bedtools-masked reference genome.

## When to use

Use when analyzing allele-specific expression from GATK HaplotypeCaller VCFs and you need to measure or mitigate reference allele mapping bias in per-tissue reference allele ratios before trusting ASE calls.

## When NOT to use

- Variants are multiallelic or within 75 bp of an indel — VADT excludes these from testable variants, so ratios would not be informative
- Variant calls were not produced by GATK's HaplotypeCaller — the VADT README warns VCFs from other programs 'could potentially have some bugs'
- The goal is de novo SNP discovery or genotyping rather than assessing mapping bias in allele ratios from RNA-seq ASE analysis

## Inputs

- FASTQ files from RNA-seq samples (e.g., 68 chicken broiler samples from SRA SRP058295 and SRP255211)
- Reference genome FASTA (Gallus_gallus_5.0, Ensembl release 86) and Ensembl GTF release 86
- Global VCF merged from all sample g.vcf files (for masking)
- GATK HaplotypeCaller per-tissue VCF files with AD allele depth fields
- VADT parameter file (VADT_Parameter_File.txt)

## Outputs

- Per-tissue mean reference allele ratios (%) for masked and unmasked alignments
- VADT reference allele bias report totaling reference allele count / total allele count per VCF
- Masked reference genome FASTA (bases replaced at global VCF variant positions)
- Re-called variant VCFs from the masked-genome alignment

## How to apply

Establish an unmasked baseline by aligning FASTQs with STAR 2.5.2b (two-pass) to the Gallus_gallus_5.0 build (Ensembl release 86), mark duplicates with Picard 2.8.1, and call variants via the GATK 3.7 pipeline (Split'N'Trim, Base Recalibration, HaplotypeCaller with g.vcf setting). Merge all sample g.vcf files into a global VCF, mask the reference genome at variant positions with Bedtools, and re-align all samples to the masked genome with identical STAR settings before re-calling variants. For informative biallelic SNPs in each tissue, compute the mean reference allele ratio from the raw AD allele depths in both masked and unmasked analyses. Use VADT to quantify bias systematically: it tests all 'testable' variants passing filters (PASS variants, non-multiallelic, quality >= 20, read count >= 20, excluding SNPs within 75 bp of indels) and reports the total (reference allele count / total allele count). Validation requires masked-reference ratios to move toward the expected 50.0%.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Tests all 'testable' biallelic variants passing filters for reference allele bias and reports the total (reference allele count / total allele count) per VCF) — https://github.com/mjtiv/VADT
- **Bedtools** (Masks the reference genome by replacing reference bases at positions of variants in the global VCF)
- **STAR 2.5.2b** (Aligns reads (two-pass) to both unmasked and masked Gallus_gallus_5.0 genome builds for the bias comparison)
- **GATK 3.7** (Runs Split'N'Trim, Base Recalibration, and HaplotypeCaller (g.vcf setting) to call variants from both alignments)
- **Picard 2.8.1** (Marks duplicate reads after alignment to either genome build)

## Examples

```
python VADT_latest --File_Name tissue_variants.vcf --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Binomial_Probability_Value 0.5
```

## Evaluation signals

- Masked-reference reference allele ratios must move toward 50%: expected shifts from 51.77% to 50.17% in breast muscle, 51.93% to 50.19% in abdominal fat, and 51.91% to 50.17% in liver
- Only variants with 'PASS' in the FILTER column, non-multiallelic, quality >= 20, and read count >= 20 per sample should be counted as testable
- Unmasked baseline ratios should be consistently above 50% across all tissues; a ratio at or below 50% unmasked suggests the pipeline is misconfigured
- Masked and unmasked alignments must use identical STAR settings so the only difference is the masked genome

## Limitations

- Masking replaces reference bases with called variants, so any variant-calling error in the global VCF propagates into the masked genome
- VADT is not a black box and run times can be extremely long (30 minutes to 10 hours) for large datasets; the README recommends HPC execution
- The article did not include an external validation dataset for VADT; it was 'tested and validated on a different dataset (data not included)'
- A variant can fail sample-level filters for combinations of reasons (homozygous, low coverage, minor allele count < 1%, 'No data'), reducing the set of testable SNPs

## Evidence

- [methods] Masking the reference genome reduced reference allele ratio toward 50% in all three tissues: "masked reference genome reduced the reference allele ration from 51.77 to 50.17% in breast muscle, 51.93% to 50.19% in abdominal fat and 51.91% to 50.17% in liver"
- [methods] Bedtools masking and re-alignment workflow: "The global vcf file was then used to mask the reference genome using Bedtools36. All samples were re-aligned to the masked genome"
- [methods] STAR two-pass alignment parameters: "aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting29,30 to the Gallus_gallus_5.0 genome build"
- [readme] VADT reference allele bias detection part: "VADT tests all "testable" variants that pass the prior filters for reference allele bias and reports the final total of (reference allele count / total allele count) for a VCF file"
- [methods] GATK pipeline description: "which includes “Split’N’Trim,” “Base Recalibration” and variant calling using HaplotypeCaller (with the g.vcf setting)"
