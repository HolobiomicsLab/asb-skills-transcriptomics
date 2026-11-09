---
name: binomial-test-allele-count-analysis
description: Use when you have a multi-sample VCF with raw allele depths per sample
  and need to classify biallelic SNPs as showing allele-specific expression versus
  equal (50/50) allelic balance.
when_to_use_negative:
- Input VCF contains multiallelic sites or indels — VADT removes multiallelic SNPs,
  indels, and SNPs within 75 bp of an indel before testing
- Per-sample raw allele depths (AD field) are absent or the VCF was not produced by
  GATK's HaplotypeCaller — VADT may have bugs with other callers' VCF formats
- The question concerns which samples (not variants) show ASE — then the multi-dimensional
  p-value adjustment per sample model, not the meta-analysis binomial variant model,
  is the appropriate analysis
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0199
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Filters the VCF, performs per-sample binomial tests on raw read counts, combines
    p-values with Fisher's Method, applies Benjamini–Hochberg FDR, and reports significant
    ASE variants
  repo: https://github.com/mjtiv/VADT
- name: SciPy
  role: Provides binom_test (null probability 0.5) for the per-sample allele-balance
    test and combine_pvalues (Fisher's Method) for the per-variant meta-analysis
- name: NumPy
  role: Required numerical package for VADT execution (installed via pip or whl)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:10eaaf3249dc13b9e3e586be8dc3d9253f1812b2b539ef8e8906eaf701388128
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
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# binomial-test-allele-count-analysis

## Summary

Per-sample binomial testing of raw allele depths (AD field) from a filtered biallelic VCF, combined across samples per variant with Fisher's Method and Benjamini–Hochberg FDR control, to detect allele-specific expression (ASE) SNPs as implemented in VADT.

## When to use

Use when you have a multi-sample VCF with raw allele depths per sample and need to classify biallelic SNPs as showing allele-specific expression versus equal (50/50) allelic balance.

## When NOT to use

- Input VCF contains multiallelic sites or indels — VADT removes multiallelic SNPs, indels, and SNPs within 75 bp of an indel before testing
- Per-sample raw allele depths (AD field) are absent or the VCF was not produced by GATK's HaplotypeCaller — VADT may have bugs with other callers' VCF formats
- The question concerns which samples (not variants) show ASE — then the multi-dimensional p-value adjustment per sample model, not the meta-analysis binomial variant model, is the appropriate analysis

## Inputs

- Multi-sample biallelic SNP VCF produced by GATK HaplotypeCaller, containing per-sample allele depths (AD field) and FILTER/QUAL columns
- VADT parameter file (VADT_Parameter_File.txt) or command-line parameters: Indel_Exclusion_Region_Length, Quality_Score_Minimum_for_Variants, Minimum_Read_Counts, Meta_BH_adj_p_value_cutoff, Binomial_Probability_Value

## Outputs

- Per-tissue significant ASE variant set (sig_meta_analysis_variants.txt) with variants passing the Benjamini–Hochberg adjusted p-value cutoff of 0.05
- Meta-analysis summary report from the VADT output directory
- Reference allele bias report (reference allele count / total allele count for the VCF)
- Retained intermediate files documenting each filtering and testing step

## How to apply

Load each per-tissue filtered informative SNP VCF (e.g., 148,860 biallelic SNPs in breast muscle, 217,628 in abdominal fat, 155,875 in liver) into VADT. First filter: retain only variants with PASS in the FILTER column, remove indels and SNPs within 75 bp of an indel, remove multiallelic SNPs below quality score 20, and drop samples that are homozygous, have read counts below 20, or have minor allele count under 1% of total counts. For each remaining informative biallelic sample, run SciPy's binom_test on the raw allele depths with probability value 0.5 (the null of equal reference/alternative allelic balance). Combine per-sample binomial p-values per variant across samples using Fisher's Method (scipy.stats.combine_pvalues) and control the false discovery rate with Benjamini–Hochberg adjustment, calling variants significant at adjusted p-value cutoff 0.05. Optionally check reference allele bias (reference allele count / total allele count) before testing, since the reference genome can cause the reference allele to map more often.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Filters the VCF, performs per-sample binomial tests on raw read counts, combines p-values with Fisher's Method, applies Benjamini–Hochberg FDR, and reports significant ASE variants) — https://github.com/mjtiv/VADT
- **SciPy** (Provides binom_test (null probability 0.5) for the per-sample allele-balance test and combine_pvalues (Fisher's Method) for the per-variant meta-analysis)
- **NumPy** (Required numerical package for VADT execution (installed via pip or whl))

## Examples

```
python VADT_"version" --File_Name breast_muscle_informative_snps.vcf --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Meta_BH_adj_p_value_cutoff 0.05 --Binomial_Probability_Value 0.5
```

## Evaluation signals

- Number of significant ASE SNPs per tissue matches reported values: 21,916 liver, 28,190 abdominal fat, 20,775 breast muscle
- ASE fraction of informative SNPs approximates 14.06% (breast muscle), 12.95% (abdominal fat), and 14.06% (liver), consistent with ~24,000 of ~174,000 informative SNPs per tissue
- Reference allele ratio after genome masking sits near 50% (50.17% in all three tissues), confirming reference allele bias is controlled before testing
- All significant variants have a Benjamini–Hochberg adjusted meta-analysis p-value at or below the 0.05 cutoff
- Per-tissue informative SNP counts match filtered totals: 148,860 (breast muscle), 217,628 (abdominal fat), 155,875 (liver)

## Limitations

- VCF files produced by tools other than GATK's HaplotypeCaller could trigger bugs in VADT, which was developed against GATK-formatted VCFs
- Run times are extremely long for large datasets (30 minutes to 10 hours), so an HPC environment is highly recommended
- Reference allele bias can inflate reference allele mapping; VADT reports the ratio but masking the reference genome was needed to bring it to ~50%
- VADT's validation dataset was not included in the paper (tested and validated on a different, unpublished dataset)

## Evidence

- [abstract] ASE analysis was performed using a custom software called VCF ASE Detection Tool (VADT), which detects ASE of biallelic SNPs using a binomial test.: "ASE analysis was performed using a custom software called VCF ASE Detection Tool (VADT), which detects ASE of biallelic SNPs using a binomial test."
- [readme] VADT performs a binomial test on all informative testable biallelic samples in the dataset using SciPy's binomial test module.: "VADT performs a binomial test on all informative "testable" biallelic samples in the dataset using SciPy's binomial test module."
- [methods] an indel filter distance of 75 base pairs (read length), a quality score minimum of 20: "The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20"
- [methods] variant calls are removed for samples that are homozygous, have low read count (coverage) or that have a minor allele count below 1% of the total counts: "variant calls are removed for samples that are homozygous, have low read count (coverage) or that have a minor allele count below 1% of the total counts"
- [abstract] On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE.: "On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE."
