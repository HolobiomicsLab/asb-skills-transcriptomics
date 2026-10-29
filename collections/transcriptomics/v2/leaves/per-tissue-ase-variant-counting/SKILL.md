---
name: per-tissue-ase-variant-counting
description: Use when you have per-tissue filtered biallelic SNP VCFs and need to
  quantify how many SNPs show allele-specific expression and what fraction of informative
  SNPs this represents.
when_to_use_negative:
- Input VCFs are unfiltered raw HaplotypeCaller output without prior GATK variant
  filtering (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100) and reference-genome
  masking — VADT expects pre-filtered informative SNP sets
- The goal is multi-allelic or indel ASE detection — VADT removes indels, SNPs within
  75 bp of indels, and multiallelic SNPs before testing
- Per-sample ASE calls (multi-dimensional p-value adjustment) are needed rather than
  per-variant meta-analysis counts
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Filters the VCF, runs the per-sample binomial test, performs Fisher's Method
    meta-analysis with Benjamini–Hochberg FDR, and writes the significant ASE variant
    set
  repo: https://github.com/mjtiv/VADT
- name: SciPy
  role: Provides scipy.stats.binom_test (p = 0.5) and scipy.stats.combine_pvalues
    used inside VADT for the binomial test and Fisher's Method
  repo: https://github.com/mjtiv/VADT
- name: NumPy
  role: Required dependency of VADT for numerical handling of allele counts
  repo: https://github.com/mjtiv/VADT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:8eecd7b6d6addee1d1afcc0d632f7db2e6c8b0b6384cf565455f6acc34c9eac2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - NumPy
  - SciPy
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9047619047619048
n_claims: 21
---

# per-tissue-ase-variant-counting

## Summary

Reproduces per-tissue allele-specific expression (ASE) SNP counts from filtered biallelic SNP VCFs using VADT's binomial test with Fisher's-method meta-analysis and Benjamini–Hochberg FDR control, yielding counts and percentages of informative SNPs showing ASE per tissue.

## When to use

Use when you have per-tissue filtered biallelic SNP VCFs and need to quantify how many SNPs show allele-specific expression and what fraction of informative SNPs this represents.

## When NOT to use

- Input VCFs are unfiltered raw HaplotypeCaller output without prior GATK variant filtering (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100) and reference-genome masking — VADT expects pre-filtered informative SNP sets
- The goal is multi-allelic or indel ASE detection — VADT removes indels, SNPs within 75 bp of indels, and multiallelic SNPs before testing
- Per-sample ASE calls (multi-dimensional p-value adjustment) are needed rather than per-variant meta-analysis counts

## Inputs

- Per-tissue filtered biallelic SNP VCF (one per tissue: breast muscle, abdominal fat, liver), produced by GATK HaplotypeCaller
- VADT parameter file (VADT_Parameter_File.txt) or CLI parameters
- Known reference counts for validation (e.g., 21,916 liver, 28,190 abdominal fat, 20,775 breast muscle ASE SNPs)

## Outputs

- sig_meta_analysis_variants.txt listing per-tissue significant ASE variants
- Meta-analysis summary report from the VADT output directory
- Per-tissue ASE SNP counts and percentages of informative SNPs (expected ~14.06% breast muscle, 12.95% abdominal fat, 14.06% liver)

## How to apply

Load each per-tissue filtered informative SNP VCF (e.g., 148,860 biallelic SNPs in breast muscle, 217,628 in abdominal fat, 155,875 in liver) into VADT. Apply VADT filtering with an indel exclusion region of 75 bp, minimum variant quality score of 20, minimum per-sample read count of 20, removing homozygous samples and those with minor allele count below 1% of total counts. Run SciPy's binom_test on raw allele depths (AD field) with probability 0.5, then combine per-sample p-values per variant with Fisher's Method and control FDR via Benjamini–Hochberg, calling variants significant at adjusted p-value cutoff 0.05. Write per-tissue significant variant sets to sig_meta_analysis_variants.txt plus the meta-analysis summary report from the VADT output directory, then compare counts and ASE percentages against expected values.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Filters the VCF, runs the per-sample binomial test, performs Fisher's Method meta-analysis with Benjamini–Hochberg FDR, and writes the significant ASE variant set) — https://github.com/mjtiv/VADT
- **SciPy** (Provides scipy.stats.binom_test (p = 0.5) and scipy.stats.combine_pvalues used inside VADT for the binomial test and Fisher's Method) — https://github.com/mjtiv/VADT
- **NumPy** (Required dependency of VADT for numerical handling of allele counts) — https://github.com/mjtiv/VADT

## Examples

```
python VADT_"version" --File_name breast_muscle_snps.vcf --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Meta_BH_adj_p_value_cutoff 0.05 --Binomial_Probability_Value 0.5
```

## Evaluation signals

- ASE SNP counts match reported values: 21,916 (liver), 28,190 (abdominal fat), 20,775 (breast muscle)
- ASE percentages of informative SNPs approximate 14.06% (breast muscle), 12.95% (abdominal fat), 14.06% (liver)
- Input VCF SNP totals match per-tissue informative counts: 148,860 (breast muscle), 217,628 (abdominal fat), 155,875 (liver)
- sig_meta_analysis_variants.txt is produced for each tissue and contains only variants passing BH-adjusted p < 0.05
- VADT's reference allele ratio report stays near 50% (masking previously reduced ratios from ~51.8–51.9% to ~50.17–50.19%)

## Limitations

- VCF files were produced by GATK's HaplotypeCaller; VCFs from other callers may cause bugs per the README
- Only ~3.7% of ASE SNPs were shared among all three tissues, so per-tissue results are largely tissue-specific and should not be pooled as a universal ASE set
- Large SNP exclusions occurred due to low overlap with the 600K chicken genotyping panel, attributed to commercial genotyping panel limitations
- Mapping issues with breast muscle samples (avg. 15.39% reads flagged 'too short' by STAR) may affect muscle-tissue ASE counts
- VADT's validation dataset was not included in the paper; runtime is long (30 minutes to 10 hours) for large datasets

## Evidence

- [abstract] On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE.: "On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE."
- [methods] Only 2196 (3.7%) SNPs were shared among all three tissues: "Only 2196 (3.7%) SNPs were shared among all three tissues"
- [methods] an indel filter distance of 75 base pairs (read length), a quality score minimum of 20: "The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20"
- [methods] variant calls are removed for samples that are homozygous, have low read count (coverage) or that have a minor allele count below 1% of the total counts: "variant calls are removed for samples that are homozygous, have low read count (coverage) or that have a minor allele count below 1% of the total counts"
- [methods] the false discovery rate (FDR) is controlled using the Benjamini–Hochberg p-value adjustment method47 generating a final adjusted p-value: "the false discovery rate (FDR) is controlled using the Benjamini–Hochberg p-value adjustment method47 generating a final adjusted p-value"
- [readme] VADT performs a binomial test on all informative "testable" biallelic samples in the dataset using SciPy's binomial test module.: "VADT performs a binomial test on all informative "testable" biallelic samples in the dataset using SciPy's binomial test module."
