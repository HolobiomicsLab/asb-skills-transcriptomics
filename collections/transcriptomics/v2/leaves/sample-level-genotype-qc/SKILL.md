---
name: sample-level-genotype-qc
description: Use when preparing per-sample biallelic SNP calls from RNA-seq VCFs for
  allele-specific expression analysis, where homozygous, low-coverage, or allele-imbalanced
  genotype calls must be excluded before statistical testing.
when_to_use_negative:
- Input is WGS/genotyping array data where heterozygosity is not the informative state
  — this QC targets RNA-seq ASE contexts where only heterozygous calls contribute
  allelic read counts
- VCFs produced by callers other than GATK HaplotypeCaller — the README warns VCF
  files from other programs could potentially have bugs in VADT
- Goal is variant-level or genotype-level QC of the global VCF itself (GATK hard filtering
  stage), rather than per-sample call filtering after variant-level filtering has
  passed
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3516
- http://edamontology.org/topic_0199
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Performs the sample-level genotype QC (homozygous, low read count < 20, minor
    allele count < 1%, no data) within its Part 1 filtering stage before ASE binomial
    testing
  repo: https://github.com/mjtiv/VADT
- name: GATK 3.7
  role: 'Upstream: produces the per-sample g.vcf files and hard-filtered VCFs that
    VADT consumes for sample-level QC'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:54e732b56da0ea2389bd679a489805969ef6e8ad290d95f1dd81bad53612ea9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - GATK 3.7
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3516
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# sample-level-genotype-qc

## Summary

Per-sample genotype quality control within a VCF at each biallelic SNP: VADT removes genotype calls from samples that are homozygous, have total read counts below a user-defined minimum (20 in this study), have a minor allele count below 1% of total counts, or have no data, so only informative heterozygous calls feed the downstream ASE binomial test.

## When to use

Use when preparing per-sample biallelic SNP calls from RNA-seq VCFs for allele-specific expression analysis, where homozygous, low-coverage, or allele-imbalanced genotype calls must be excluded before statistical testing.

## When NOT to use

- Input is WGS/genotyping array data where heterozygosity is not the informative state — this QC targets RNA-seq ASE contexts where only heterozygous calls contribute allelic read counts
- VCFs produced by callers other than GATK HaplotypeCaller — the README warns VCF files from other programs could potentially have bugs in VADT
- Goal is variant-level or genotype-level QC of the global VCF itself (GATK hard filtering stage), rather than per-sample call filtering after variant-level filtering has passed

## Inputs

- Per-tissue multi-sample VCF produced by GATK HaplotypeCaller (g.vcf setting) merging, after GATK hard filtering (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100 removed)
- VADT parameter file or CLI parameters: --Minimum_Read_Counts 20, --Indel_Exclusion_Region_Length 75, --Quality_Score_Minimum_for_Variants 20

## Outputs

- Set of informative (testable) heterozygous sample genotype calls per biallelic SNP
- Per-tissue counts of informative biallelic SNPs (e.g., 148,860 breast muscle, 217,628 abdominal fat, 155,875 liver)
- VADT intermediate filtering files (GATK_Failing_RNA_Seq_variants.txt, Other_Failing_RNA_Seq_variants.txt, Identified_Indel_Regions.txt, Testable_Informative_Filt_Variants.txt)

## How to apply

Load the per-tissue VCF (merged g.vcf files post-GATK hard filtering) and, at each variant that survived variant-level filtering (PASS only, no indels or SNPs within 75 bp of an indel, biallelic, quality score >= 20), apply sample-level filters: drop genotype calls from samples that are homozygous (only informative heterozygotes are testable for ASE), have reference plus alternative read counts below the minimum read count threshold of 20, have a minor allele count below 1% of the total counts, or have no genotype data. A variant can fail for one or a combination of these filters. Retain VADT's documented intermediate files (e.g., Testable_Informative_Filt_Variants.txt) so each sample-level exclusion is auditable, then tally the remaining informative biallelic SNPs per tissue.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Performs the sample-level genotype QC (homozygous, low read count < 20, minor allele count < 1%, no data) within its Part 1 filtering stage before ASE binomial testing) — https://github.com/mjtiv/VADT
- **GATK 3.7** (Upstream: produces the per-sample g.vcf files and hard-filtered VCFs that VADT consumes for sample-level QC)

## Examples

```
python VADT_1.0.py --File_Name tissue.vcf --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Meta_BH_adj_p_value_cutoff 0.05
```

## Evaluation signals

- Per-tissue informative biallelic SNP counts match the reported 148,860 (breast muscle), 217,628 (abdominal fat), and 155,875 (liver), averaging ~174,000 per tissue
- No homozygous, low-coverage (< 20 total reads), minor-allele < 1%, or no-data genotype calls remain among samples marked testable in Testable_Informative_Filt_Variants.txt
- Every sample excluded from a variant can be attributed to at least one of the four documented sample-level filters (homozygous, low read count, minor allele count, no data)
- All variants entering sample-level QC carry a PASS value in the VCF FILTER column, are biallelic, and have quality score >= 20
- Only a small fraction of informative SNPs (~14%, ~24,000 per tissue) subsequently show significant ASE, so near-100% ASE rates would suggest misapplied filters

## Limitations

- Homozygous samples cannot be tested for ASE at all, so sample-level QC inherently restricts analysis to heterozygous individuals per SNP
- VADT is not a black box but run times are long (30 minutes to 10 hours per dataset); the article also notes VADT was validated on a dataset not included in the paper
- Filtering is sensitive to the read-length-derived indel exclusion distance (75 bp) and minimum read count (20) parameters, which are user-defined and dataset-dependent

## Evidence

- [methods] variant calls are removed for samples that are homozygous, have low read count (coverage) or that have a minor allele count below 1% of the total counts: "variant calls are removed for samples that are homozygous, have low read count (coverage) or that have a minor allele count below 1% of the total counts"
- [methods] The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20: "The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20"
- [readme] Sample level filtering consists of filtering based on removing all samples that are homozygous, low read count (user parameter), allele count is <1% of the total counts or "No data".: "Sample level filtering consists of filtering based on removing all samples that are homozygous, low read count (user parameter), allele count is <1% of the total counts or "No data"."
- [abstract] On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE.: "On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE."
- [readme] VCF files utilized by VADT were produced by GATK's Haplotypecaller and so VCF files produced by other programs could potentially have some bugs.: "VCF files utilized by VADT were produced by GATK's Haplotypecaller and so VCF files produced by other programs could potentially have some bugs."
