---
name: allele-specific-expression-detection
description: Use when multi-sample VCF files (GATK HaplotypeCaller output) from RNA-seq
  of heterozygous individuals are available and the goal is to identify which biallelic
  SNPs show statistically significant allele-specific expression per tissue.
when_to_use_negative:
- VCFs not produced by GATK's HaplotypeCaller — the README warns VCFs from other programs
  'could potentially have some bugs'
- Multiallelic sites or indels, which are removed by design before ASE testing
- Questions about allele- or isoform-level quantification that require eQTL or phased
  haplotype data — the article notes ASE-translation associations are speculative
  'without eQTL data'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0625
tools:
- name: VADT (VCF ASE Detection Tool)
  role: 'Main tool: filters the VCF, detects reference allele bias, runs the per-sample
    binomial test, combines p-values via Fisher''s Method, and applies Benjamini–Hochberg
    FDR to call significant ASE variants'
  repo: https://github.com/mjtiv/VADT
- name: SciPy
  role: Provides scipy.stats.binom_test for the per-sample binomial test and scipy.stats.combine_pvalues
    for Fisher's Method meta-analysis
  repo: https://github.com/mjtiv/VADT
- name: NumPy
  role: Numerical dependency of VADT for array handling during filtering and statistical
    analysis
  repo: https://github.com/mjtiv/VADT
- name: Compare_VCF_Files
  role: Companion tool to assess concordance of matching samples and SNPs between
    genotyping and RNA-seq VCFs when validating the informative SNP set
  repo: https://github.com/mjtiv/Compare_VCF_Files
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:a7cf1be228ad6d330b70a0102f8b254ed30c876393725015122993ef355c5a8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Compare_VCF_Files
  - NumPy
  - SciPy
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# allele-specific-expression-detection

## Summary

Detects allele-specific expression (ASE) of biallelic SNPs from RNA-seq VCFs using VADT, which filters variants/samples, runs a per-sample binomial test on raw allele depths, and combines p-values with Fisher's Method plus Benjamini–Hochberg FDR control.

## When to use

Use when multi-sample VCF files (GATK HaplotypeCaller output) from RNA-seq of heterozygous individuals are available and the goal is to identify which biallelic SNPs show statistically significant allele-specific expression per tissue.

## When NOT to use

- VCFs not produced by GATK's HaplotypeCaller — the README warns VCFs from other programs 'could potentially have some bugs'
- Multiallelic sites or indels, which are removed by design before ASE testing
- Questions about allele- or isoform-level quantification that require eQTL or phased haplotype data — the article notes ASE-translation associations are speculative 'without eQTL data'

## Inputs

- Per-tissue multi-sample VCF files of biallelic SNPs from GATK HaplotypeCaller (GATK 3.7) RNA-seq variant calling
- VADT parameter file (VADT_Parameter_File.txt) with indel exclusion length, quality score minimum, read count threshold, p-value cutoffs, and binomial probability
- Optional: 600K chicken genotyping panel VCF (GSE131764) filtered to SNPs with ≥97% call rates for overlap/concordance checks

## Outputs

- Per-tissue list of significant ASE variants (sig_meta_analysis_variants.txt)
- VADT meta-analysis summary report from the output directory
- Documented intermediate files for every filtering, reference-bias, and binomial-test step
- Counts of ASE SNPs and percentages of informative SNPs per tissue (e.g., 20,775; 28,190; 21,916)

## How to apply

Load each per-tissue filtered VCF (e.g., 148,860 breast muscle, 217,628 abdominal fat, 155,875 liver biallelic SNPs) into VADT with an indel exclusion region of 75 bp, quality score minimum of 20, minimum read count of 20, and binomial probability 0.5. VADT first removes GATK-flagged failing variants, indels, SNPs within the indel exclusion distance, multiallelic SNPs, and samples that are homozygous, low-coverage, or have minor allele count below 1%. For surviving informative variants it tests reference allele bias, then performs SciPy's binomial test on raw AD-field read counts per sample. Per-variant p-values are combined across samples with Fisher's Method and adjusted with Benjamini–Hochberg FDR; variants with adjusted p ≤ 0.05 are called ASE. Expected output for the broiler study is ~24,000 ASE SNPs per tissue (~14% of ~174,000 informative SNPs), with reported values of 20,775 (breast muscle), 28,190 (abdominal fat), and 21,916 (liver).

## Related tools

- **VADT (VCF ASE Detection Tool)** (Main tool: filters the VCF, detects reference allele bias, runs the per-sample binomial test, combines p-values via Fisher's Method, and applies Benjamini–Hochberg FDR to call significant ASE variants) — https://github.com/mjtiv/VADT
- **SciPy** (Provides scipy.stats.binom_test for the per-sample binomial test and scipy.stats.combine_pvalues for Fisher's Method meta-analysis) — https://github.com/mjtiv/VADT
- **NumPy** (Numerical dependency of VADT for array handling during filtering and statistical analysis) — https://github.com/mjtiv/VADT
- **Compare_VCF_Files** (Companion tool to assess concordance of matching samples and SNPs between genotyping and RNA-seq VCFs when validating the informative SNP set) — https://github.com/mjtiv/Compare_VCF_Files

## Examples

```
python VADT_1.0 --File_Name breast_muscle_informative_snps.vcf --Output_File_Location ./vadt_out --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Meta_BH_adj_p_value_cutoff 0.05 --Binomial_Probability_Value 0.5
```

## Evaluation signals

- Per-tissue ASE counts reproduce the reported values: 21,916 (liver), 28,190 (abdominal fat), 20,775 (breast muscle) significant variants at adjusted p ≤ 0.05
- ASE percentages of informative SNPs match ~14.06% (breast muscle), ~12.95% (abdominal fat), ~14.06% (liver), consistent with ~24,000 of ~174,000 informative SNPs per tissue
- Reference allele ratio reported by VADT's bias check is near 50% (the masked reference genome reduced it from ~51.8–51.9% to ~50.17–50.19%)
- sig_meta_analysis_variants.txt and the meta-analysis summary report exist in the VADT output directory and variant counts are internally consistent with intermediate filtering files
- Shared-ASE overlap check: only ~3.7% of ASE SNPs (2,196) should appear in all three tissues, with ~83% tissue-specific

## Limitations

- VCF files from programs other than GATK HaplotypeCaller may cause bugs; run times are long (30 minutes to 10 hours) so HPC execution is recommended for large datasets
- VADT was validated on a different dataset not included in the paper; a formal validation dataset is absent
- Many SNPs were excluded due to low overlap with the 600K chicken genotyping panel, attributed to limitations of commercial genotyping panels
- Breast muscle samples had mapping issues (avg. 15.39% of reads flagged 'too short' by STAR vs. 4.50–5.33% for other tissues), likely due to the genome build/annotation
- Detection of significant ASE lacks eQTL support, so downstream biological interpretation is speculative

## Evidence

- [abstract] ASE analysis was performed using a custom software called VCF ASE Detection Tool (VADT), which detects ASE of biallelic SNPs using a binomial test.: "ASE analysis was performed using a custom software called VCF ASE Detection Tool (VADT), which detects ASE of biallelic SNPs using a binomial test."
- [methods] The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20: "The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20"
- [methods] the false discovery rate (FDR) is controlled using the Benjamini–Hochberg p-value adjustment method47 generating a final adjusted p-value: "the false discovery rate (FDR) is controlled using the Benjamini–Hochberg p-value adjustment method47 generating a final adjusted p-value"
- [readme] VADT performs a binomial test on all informative "testable" biallelic samples in the dataset using SciPy's binomial test module.: "VADT performs a binomial test on all informative "testable" biallelic samples in the dataset using SciPy's binomial test module."
- [readme] VADT next performs a meta-analysis accross all tested samples on per variant basis using Fisher's Method (Fisher 1958).: "VADT next performs a meta-analysis accross all tested samples on per variant basis using Fisher's Method (Fisher 1958)."
- [abstract] On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE.: "On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE."
