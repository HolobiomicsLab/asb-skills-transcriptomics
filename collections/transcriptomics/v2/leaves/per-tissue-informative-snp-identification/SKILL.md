---
name: per-tissue-informative-snp-identification
description: Use when you have per-tissue RNA-seq VCF files (e.g., GATK HaplotypeCaller
  g.vcf-derived) and need to count or extract biallelic SNPs eligible for binomial-test
  ASE analysis before running the ASE testing stage.
when_to_use_negative:
- Input is a VCF produced by a caller other than GATK HaplotypeCaller — the README
  warns that 'VCF files produced by other programs could potentially have some bugs'
- Goal is to detect ASE itself (binomial testing, Fisher's-method meta-analysis, FDR
  control) rather than to derive the pre-test set of informative SNPs
- Input variants are indels, multiallelic, or GATK-failing — VADT removes these by
  design before informative SNP enumeration
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0199
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Filters the per-tissue VCF (removes GATK-failing variants, indels, SNPs within
    75 bp of an indel, multiallelic SNPs, quality < 20) and applies sample-level filters
    (homozygous, read counts < 20, minor allele count < 1%, no data) to emit the informative
    biallelic SNP set
  repo: https://github.com/mjtiv/VADT
- name: GATK
  role: Upstream pipeline (Split'N'Trim, Base Recalibration, HaplotypeCaller) that
    produces the per-tissue VCF input and flags variants as failing before VADT filtering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:3bb9821ff381e65e52a9ebbc2cb9dd4112bae85437e9ab10cabb61a40870f7da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - GATK
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# per-tissue-informative-snp-identification

## Summary

Derives the set of biallelic SNPs per tissue that pass VADT's variant- and sample-level filtering criteria and are therefore 'informative' (testable) for allele-specific expression (ASE) analysis from per-tissue RNA-seq VCF files.

## When to use

Use when you have per-tissue RNA-seq VCF files (e.g., GATK HaplotypeCaller g.vcf-derived) and need to count or extract biallelic SNPs eligible for binomial-test ASE analysis before running the ASE testing stage.

## When NOT to use

- Input is a VCF produced by a caller other than GATK HaplotypeCaller — the README warns that 'VCF files produced by other programs could potentially have some bugs'
- Goal is to detect ASE itself (binomial testing, Fisher's-method meta-analysis, FDR control) rather than to derive the pre-test set of informative SNPs
- Input variants are indels, multiallelic, or GATK-failing — VADT removes these by design before informative SNP enumeration

## Inputs

- Per-tissue multi-sample VCF files created by merging g.vcf files by tissue after GATK hard filtering (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100)
- Optional VADT parameter file (VADT_Parameter_File.txt) or command-line parameters

## Outputs

- Testable_Informative_Filt_Variants.txt per tissue (list of remaining informative biallelic SNPs)
- GATK_Failing_RNA_Seq_variants.txt, Other_Failing_RNA_Seq_variants.txt, and Identified_Indel_Regions.txt intermediate filtering files per tissue
- Per-tissue counts of informative biallelic SNPs (expected: 148,860 breast muscle; 217,628 abdominal fat; 155,875 liver)

## How to apply

First ensure each per-tissue VCF was produced by merging tissue g.vcf files after GATK hard filtering that removed variants with Fisher Strand > 30.0, Quality Depth < 2.0, and Depth < 100. Run VADT (github.com/mjtiv/VADT) with the study's parameters: indel exclusion region length of 75 bp (read length), variant quality score minimum of 20, and minimum read counts of 20 per sample; VADT drops GATK-failing variants (non-PASS FILTER), indels, SNPs within 75 bp of an indel, multiallelic SNPs, and low-quality variants, then applies sample-level filtering removing genotype calls that are homozygous, below the 20-read coverage threshold, have minor allele count below 1% of total counts, or have no data. Collect the remaining biallelic SNPs from VADT's documented intermediate output Testable_Informative_Filt_Variants.txt (along with GATK_Failing_RNA_Seq_variants.txt, Other_Failing_RNA_Seq_variants.txt, and Identified_Indel_Regions.txt) and tally them per tissue. Correct application is judged by whether the per-tissue informative SNP counts match the reported values of 148,860 (breast muscle), 217,628 (abdominal fat), and 155,875 (liver), averaging ~174,000 informative SNPs per tissue, of which ~24,000 (~14%) showed ASE.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Filters the per-tissue VCF (removes GATK-failing variants, indels, SNPs within 75 bp of an indel, multiallelic SNPs, quality < 20) and applies sample-level filters (homozygous, read counts < 20, minor allele count < 1%, no data) to emit the informative biallelic SNP set) — https://github.com/mjtiv/VADT
- **GATK** (Upstream pipeline (Split'N'Trim, Base Recalibration, HaplotypeCaller) that produces the per-tissue VCF input and flags variants as failing before VADT filtering)

## Examples

```
python VADT_1.0 --File_Name breast_muscle.vcf --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Meta_BH_adj_p_value_cutoff 0.05 --Multi_Dim_adjust_pvalue_cutoff 0.05
```

## Evaluation signals

- Per-tissue informative SNP counts match the reported values: 148,860 (breast muscle), 217,628 (abdominal fat), and 155,875 (liver), averaging ~174,000 per tissue
- Testable_Informative_Filt_Variants.txt exists per tissue and contains only PASS biallelic SNPs with variant quality >= 20 and >= 75 bp from any indel
- All remaining SNPs are biallelic — no multiallelic records appear in the informative SNP set
- Every retained sample genotype at an informative SNP is heterozygous with total read counts >= 20 and minor allele count >= 1% of total counts
- Of the informative SNPs, ~24,000 (~14%) per tissue should subsequently show ASE, providing a downstream sanity check

## Limitations

- VADT expects VCF files produced by GATK's HaplotypeCaller; VCFs from other callers may cause bugs
- Run times can be extremely long for large datasets (30 minutes to 10 hours), so HPC execution is recommended
- The study noted a large number of SNPs excluded due to low overlap with the 600K chicken genotyping panel, attributed to limitations of commercial genotyping panels
- Mapping issues with breast muscle tissue samples (avg. 15.39% of reads flagged 'too short' by STAR) affected the analysis pipeline

## Evidence

- [abstract] On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE.: "On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE."
- [methods] GATK hard filtering thresholds: "using GATK to remove variants with “Fisher Strand > 30.0”, “Quality Depth < 2.0” and “Depth < 100”"
- [methods] Study VADT parameters: "The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20"
- [methods] Sample-level filtering rule: "variant calls are removed for samples that are homozygous, have low read count (coverage) or that have a minor allele count below 1% of the total counts"
- [readme] VADT README filtering overview: "VADT removes all variants within a certain distance of an indel defined by the user. VADT also  removes all variants with multiple reference and alternative alleles and variants with less than the"
- [readme] GATK HaplotypeCaller input caveat: "VCF files utilized by VADT were produced by GATK's Haplotypecaller and so VCF files produced by other programs could potentially have some bugs."
