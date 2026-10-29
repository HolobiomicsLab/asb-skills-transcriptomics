---
name: snp-count-validation-against-published-results
description: Use when re-running or reproducing the VADT filtering stage on per-tissue
  RNA-seq VCF files and you need to confirm the number of informative biallelic SNPs
  matches the published per-tissue counts before downstream ASE testing.
when_to_use_negative:
- The goal is to call ASE itself rather than validate filtering counts — the downstream
  binomial test, Fisher's-method meta-analysis, and Benjamini–Hochberg FDR (cutoff
  0.05) stages of VADT are a separate skill
- Input VCFs were produced by a caller other than GATK HaplotypeCaller — the VADT
  README warns VCFs from other programs 'could potentially have some bugs'
- Variants are multiallelic or indels — they are removed by VADT's filters and cannot
  contribute to the informative SNP tally
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3516
- http://edamontology.org/topic_0199
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Filters the per-tissue VCF (GATK-failing variants, indels, SNPs within 75
    bp of an indel, multiallelic SNPs, quality < 20) and applies sample-level filters
    (homozygous, read count < 20, minor allele count < 1%, no data), emitting Testable_Informative_Filt_Variants.txt
    used for the count tally
  repo: https://github.com/mjtiv/VADT
- name: GATK
  role: Upstream hard filtering (Fisher Strand > 30.0, Quality Depth < 2.0, Depth
    < 100) that defines which variants reach the VADT filtering stage
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:ceeaaee864daba7b785c953f5f6aa25231e88b905fe69c4f86151d2049dfbfc1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - GATK
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3516
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# snp-count-validation-against-published-results

## Summary

Validates a per-tissue ASE filtering pipeline by tallying the informative biallelic SNPs that survive VADT's variant- and sample-level filters and comparing the counts to the study's reported values (148,860 breast muscle, 217,628 abdominal fat, 155,875 liver; ~174,000 average).

## When to use

Use when re-running or reproducing the VADT filtering stage on per-tissue RNA-seq VCF files and you need to confirm the number of informative biallelic SNPs matches the published per-tissue counts before downstream ASE testing.

## When NOT to use

- The goal is to call ASE itself rather than validate filtering counts — the downstream binomial test, Fisher's-method meta-analysis, and Benjamini–Hochberg FDR (cutoff 0.05) stages of VADT are a separate skill
- Input VCFs were produced by a caller other than GATK HaplotypeCaller — the VADT README warns VCFs from other programs 'could potentially have some bugs'
- Variants are multiallelic or indels — they are removed by VADT's filters and cannot contribute to the informative SNP tally

## Inputs

- Per-tissue merged VCF files (breast muscle, abdominal fat, liver) generated from GATK HaplotypeCaller g.vcf output after GATK hard filtering (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100)
- VADT parameter file (VADT_Parameter_File.txt) with indel exclusion 75, quality minimum 20, minimum read counts 20

## Outputs

- Per-tissue informative biallelic SNP counts (expected: 148,860 breast muscle; 217,628 abdominal fat; 155,875 liver; average ~174,000)
- VADT intermediate filtering reports: GATK_Failing_RNA_Seq_variants.txt, Other_Failing_RNA_Seq_variants.txt, Identified_Indel_Regions.txt, Testable_Informative_Filt_Variants.txt (per tissue)

## How to apply

Load the per-tissue VCFs produced by merging tissue-specific g.vcf files after GATK hard filtering (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100). Run VADT (github.com/mjtiv/VADT) with the study's parameters: indel exclusion region of 75 bp, quality score minimum of 20, and minimum read counts of 20 per sample; VADT removes GATK-failing variants, indels, SNPs within 75 bp of an indel, and multiallelic SNPs, then drops sample-level calls that are homozygous, low-coverage, minor allele count < 1% of total counts, or missing. Inspect VADT's retained intermediate files (GATK_Failing_RNA_Seq_variants.txt, Other_Failing_RNA_Seq_variants.txt, Identified_Indel_Regions.txt, Testable_Informative_Filt_Variants.txt) per tissue, and tally the remaining biallelic SNPs in Testable_Informative_Filt_Variants.txt. Rationale: VADT is deliberately 'not a black box' — all intermediate files are retained so filtering outcomes can be audited and counts reconciled against the published totals rather than trusting a single output number.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Filters the per-tissue VCF (GATK-failing variants, indels, SNPs within 75 bp of an indel, multiallelic SNPs, quality < 20) and applies sample-level filters (homozygous, read count < 20, minor allele count < 1%, no data), emitting Testable_Informative_Filt_Variants.txt used for the count tally) — https://github.com/mjtiv/VADT
- **GATK** (Upstream hard filtering (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100) that defines which variants reach the VADT filtering stage)

## Examples

```
python VADT_1.0 --File_Name breast_muscle.vcf --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20
```

## Evaluation signals

- Per-tissue informative SNP counts match 148,860 (breast muscle), 217,628 (abdominal fat), and 155,875 (liver), averaging ~174,000
- Testable_Informative_Filt_Variants.txt exists per tissue and contains only biallelic, quality >= 20 SNPs at least 75 bp from any indel
- GATK_Failing_RNA_Seq_variants.txt accounts for all variants whose FILTER column was not 'PASS' in the input VCF
- Identified_Indel_Regions.txt reflects the 75 bp user-defined indel exclusion distance
- No retained variant has any sample with read counts below 20 or minor allele count below 1% of total counts

## Limitations

- VADT run times are long (30 minutes to 10 hours depending on dataset), so validation on large per-tissue VCFs may require HPC execution
- VADT was only tested on VCFs produced by GATK's HaplotypeCaller; other callers may break parsing
- The paper notes VADT 'was tested and validated on a different dataset (data not included)', so count reproduction depends on matching all user-defined parameters exactly

## Evidence

- [abstract] On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE.: "On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE."
- [methods] using GATK to remove variants with 'Fisher Strand > 30.0', 'Quality Depth < 2.0' and 'Depth < 100': "using GATK to remove variants with “Fisher Strand > 30.0”, “Quality Depth < 2.0” and “Depth < 100”"
- [methods] an indel filter distance of 75 base pairs (read length), a quality score minimum of 20 ... a SNP read count threshold of 20 for each sample: "The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20"
- [readme] VADT is not a black box program and all intermediate files are retained for the user to help the user better understand each step performed: "It is important to point out VADT is not a black box program and all intermediate files are retained for the user to help the user better understand each step performed"
- [readme] Sample level filtering consists of filtering based on removing all samples that are homozygous, low read count (user parameter), allele count is <1% of the total counts or 'No data'.: "Sample level filtering consists of filtering based on removing all samples that are homozygous, low read count (user parameter), allele count is <1% of the total counts or "No data"."
