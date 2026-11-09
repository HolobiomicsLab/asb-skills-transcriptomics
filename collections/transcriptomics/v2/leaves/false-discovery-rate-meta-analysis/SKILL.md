---
name: false-discovery-rate-meta-analysis
description: Use when you have raw per-sample allele depths (AD field) for biallelic
  SNPs in a VCF, per-sample binomial p-values testing allelic imbalance against probability
  0.5, and need a per-variant significance call with FDR control.
when_to_use_negative:
- The goal is calling significant samples rather than significant variants — VADT
  offers a separate multi-dimensional p-value adjustment model (Part 4B) for that
  question
- Variants are multiallelic, indels, or within 75 bp of an indel — VADT's meta-analysis
  only runs on informative testable biallelic SNPs that pass filtering
- Genotype-quality annotation is absent (e.g. VCFs not produced by GATK HaplotypeCaller),
  since the README warns other VCF producers could expose bugs in VADT
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3512
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Orchestrates filtering, per-sample binomial testing, Fisher's Method meta-analysis,
    Benjamini–Hochberg FDR control, and reporting of significant ASE variants
  repo: https://github.com/mjtiv/VADT
- name: SciPy
  role: Supplies binom_test for per-sample allelic-imbalance p-values and combine_pvalues
    for Fisher's Method aggregation
- name: NumPy
  role: Numeric dependency required to run VADT's statistical analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:ee0ee04b989cd9260aa801d3c90da45bad6323e0606ac73b9b674720e46caf0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - NumPy
  - SciPy
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# false-discovery-rate-meta-analysis

## Summary

Per-variant meta-analysis for allele-specific expression detection: combine per-sample binomial-test p-values with Fisher's Method and control the false discovery rate using Benjamini–Hochberg adjustment, as implemented in VADT for biallelic SNPs from broiler chicken RNA-seq.

## When to use

Use when you have raw per-sample allele depths (AD field) for biallelic SNPs in a VCF, per-sample binomial p-values testing allelic imbalance against probability 0.5, and need a per-variant significance call with FDR control.

## When NOT to use

- The goal is calling significant samples rather than significant variants — VADT offers a separate multi-dimensional p-value adjustment model (Part 4B) for that question
- Variants are multiallelic, indels, or within 75 bp of an indel — VADT's meta-analysis only runs on informative testable biallelic SNPs that pass filtering
- Genotype-quality annotation is absent (e.g. VCFs not produced by GATK HaplotypeCaller), since the README warns other VCF producers could expose bugs in VADT

## Inputs

- Filtered informative biallelic SNP VCF (GATK HaplotypeCaller output, multi-sample per tissue, e.g. 148,860 SNPs breast muscle, 217,628 abdominal fat, 155,875 liver)
- Raw allele depths (AD field) per sample per variant
- User-defined parameter values: indel exclusion region 75 bp, quality score minimum 20, minimum read counts 20, BH adjusted p-value cutoff 0.05, binomial probability value 0.5

## Outputs

- Per-variant Fisher's Method meta p-values across samples
- Benjamini–Hochberg FDR-adjusted p-values per variant
- Set of significant ASE variants at adjusted p ≤ 0.05 (e.g. 20,775 breast muscle, 28,190 abdominal fat, 21,916 liver), written to sig_meta_analysis_variants.txt
- Meta-analysis summary report and all intermediate processing files from the VADT output directory

## How to apply

First filter the VCF (remove GATK-flagged failures, indels, SNPs within 75 bp of an indel, multiallelic SNPs, quality score below 20, and samples that are homozygous, low-coverage below 20 reads, or with minor allele count below 1%). Then run SciPy's binom_test on raw allele depths per sample per variant with Binomial_Probability_Value 0.5. Combine the per-sample p-values for each variant with Fisher's Method (scipy.stats.combine_pvalues), then apply the Benjamini–Hochberg FDR adjustment and call variants significant at an adjusted p-value cutoff of 0.05 (Meta_BH_adj_p_value_cutoff). VADT retains all intermediate files so each step of the meta-analysis chain can be audited.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Orchestrates filtering, per-sample binomial testing, Fisher's Method meta-analysis, Benjamini–Hochberg FDR control, and reporting of significant ASE variants) — https://github.com/mjtiv/VADT
- **SciPy** (Supplies binom_test for per-sample allelic-imbalance p-values and combine_pvalues for Fisher's Method aggregation)
- **NumPy** (Numeric dependency required to run VADT's statistical analysis)

## Examples

```
python VADT_"version" --File_Name tissue_snps.vcf --Output_File_Location out/ --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Meta_BH_adj_p_value_cutoff 0.05 --Binomial_Probability_Value 0.5
```

## Evaluation signals

- Number of ASE SNPs per tissue matches the reported values: 21,916 (liver), 28,190 (abdominal fat), 20,775 (breast muscle)
- ASE fraction of informative SNPs equals ~14.06% (breast muscle), ~12.95% (abdominal fat), ~14.06% (liver), consistent with the reported ~14% average
- sig_meta_analysis_variants.txt and the meta-analysis summary report exist in the VADT output directory
- Input counts per tissue are correct: 148,860 biallelic SNPs (breast muscle), 217,628 (abdominal fat), 155,875 (liver), averaging ~174,000 informative SNPs per tissue
- Only variants with 'PASS' in the FILTER column and no homozygous/low-coverage/minor-allele-count<1% samples entered the meta-analysis (check intermediate files)

## Limitations

- VADT is not a black box but its accuracy depends on GATK-formatted VCFs; VCFs from other callers may cause bugs
- Run times are extremely long for large datasets (30 minutes to 10 hours), so an HPC environment is recommended
- Reference allele bias can inflate reference allele mapping; VADT reports reference allele ratio but the masked-genome result still ranged from 50.17% to 50.19% rather than exactly 50%
- VADT was tested and validated on a different dataset not included in the paper; overall accuracy of the statistical models was overseen by a statistician (Dr. Jing Qiu)

## Evidence

- [intro] Fisher's Method meta-analysis with Benjamini–Hochberg FDR: "a meta-analysis (variants) or multi-dimensional p-value adjustment (samples)"
- [methods] Per-sample binomial test parameters: "the false discovery rate (FDR) is controlled using the Benjamini–Hochberg p-value adjustment method47 generating a final adjusted p-value"
- [abstract] Reported ASE counts and percentages: "On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE."
- [readme] VADT meta-analysis description: "VADT next performs a meta-analysis accross all tested samples on per variant basis using Fisher's Method"
- [readme] CLI invocation parameters: "--Meta_BH_adj_p_value_cutoff 0.05"
