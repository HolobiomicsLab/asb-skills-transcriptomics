---
name: vcf-variant-filtering
description: Use when you have per-tissue multi-sample VCFs (e.g., GATK HaplotypeCaller
  g.vcf output merged by tissue) and need to remove low-quality variants, indels,
  indel-proximate and multiallelic SNPs before ASE or allele-count-based testing.
when_to_use_negative:
- Your variants come from whole-genome/genotyping-array data rather than RNA-seq-aligned
  reads — the read-count thresholds (20 reads) and indel exclusion (75 bp read length)
  are calibrated to RNA-seq depth and alignment error
- You need multiallelic or indel-level ASE quantification — VADT explicitly removes
  indels, multiallelic variants, and SNPs within 75 bp of an indel
- Input VCFs are not in the GATK HaplotypeCaller format — the README warns VCFs from
  other callers 'could potentially have some bugs'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3170
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Performs the variant-level filtering (GATK failures, indels, 75 bp indel proximity,
    multiallelic, quality score < 20) and sample-level filtering (homozygous, read
    count < 20, minor allele count < 1%, no data) that yield the informative biallelic
    SNP sets
  repo: https://github.com/mjtiv/VADT
- name: GATK 3.7
  role: Upstream hard filtering of variants (Fisher Strand > 30.0, Quality Depth <
    2.0, Depth < 100) and production of the HaplotypeCaller g.vcf inputs that VADT
    filters
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:dceccbb4300e39ba11cc9bf62a81fb672864de2d4526ec38043530207dd86870
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - GATK 3.7
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# vcf-variant-filtering

## Summary

Derive per-tissue informative biallelic SNP sets from RNA-seq-derived VCFs by combining GATK hard filters (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100) with VADT's variant- and sample-level filtering, so that only heterozygous, well-covered SNPs are testable for allele-specific expression.

## When to use

Use when you have per-tissue multi-sample VCFs (e.g., GATK HaplotypeCaller g.vcf output merged by tissue) and need to remove low-quality variants, indels, indel-proximate and multiallelic SNPs before ASE or allele-count-based testing.

## When NOT to use

- Your variants come from whole-genome/genotyping-array data rather than RNA-seq-aligned reads — the read-count thresholds (20 reads) and indel exclusion (75 bp read length) are calibrated to RNA-seq depth and alignment error
- You need multiallelic or indel-level ASE quantification — VADT explicitly removes indels, multiallelic variants, and SNPs within 75 bp of an indel
- Input VCFs are not in the GATK HaplotypeCaller format — the README warns VCFs from other callers 'could potentially have some bugs'

## Inputs

- Per-tissue multi-sample VCF files (GATK HaplotypeCaller g.vcf-derived, merged by tissue)
- VADT parameter file (VADT_Parameter_File.txt) or command-line parameters (Indel_Exclusion_Region_Length=75, Quality_Score_Minimum_for_Variants=20, Minimum_Read_Counts=20)

## Outputs

- Testable_Informative_Filt_Variants.txt per tissue (filtered informative biallelic SNP set)
- GATK_Failing_RNA_Seq_variants.txt per tissue
- Other_Failing_RNA_Seq_variants.txt per tissue
- Identified_Indel_Regions.txt per tissue
- Per-tissue counts of informative biallelic SNPs

## How to apply

Start from per-tissue VCFs produced by merging g.vcf files after GATK hard filtering that removed variants with Fisher Strand > 30.0, Quality Depth < 2.0, and Depth < 100. Run VADT's filtering stage: drop variants flagged as failing by GATK (only 'PASS' variants are retained), drop indels, SNPs within 75 base pairs of an indel (read length), multiallelic SNPs, and variants below the minimum quality score of 20. Then apply VADT's sample-level filtering at each remaining SNP: remove genotype calls from samples that are homozygous, have read counts below the minimum threshold of 20, have a minor allele count below 1% of total counts, or have no data. Retain VADT's documented intermediate files (GATK_Failing_RNA_Seq_variants.txt, Other_Failing_RNA_Seq_variants.txt, Identified_Indel_Regions.txt, Testable_Informative_Filt_Variants.txt) per tissue so every exclusion is traceable, then tally the remaining informative biallelic SNPs per tissue. The rationale is that only heterozygous sites with sufficient, balanced read counts in a sample can support a binomial ASE test.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Performs the variant-level filtering (GATK failures, indels, 75 bp indel proximity, multiallelic, quality score < 20) and sample-level filtering (homozygous, read count < 20, minor allele count < 1%, no data) that yield the informative biallelic SNP sets) — https://github.com/mjtiv/VADT
- **GATK 3.7** (Upstream hard filtering of variants (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100) and production of the HaplotypeCaller g.vcf inputs that VADT filters)

## Examples

```
python VADT_1.0 --File_Name breast_muscle.vcf --Output_File_Location ./out --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Meta_BH_adj_p_value_cutoff 0.05 --Binomial_Probability_Value 0.5
```

## Evaluation signals

- Per-tissue informative SNP counts match the reported values: 148,860 (breast muscle), 217,628 (abdominal fat), 155,875 (liver), averaging ~174,000 per tissue
- All variants retained in the filtered set carry 'PASS' in the FILTER column; failing variants appear in GATK_Failing_RNA_Seq_variants.txt or Other_Failing_RNA_Seq_variants.txt
- No indels, multiallelic SNPs, or SNPs within 75 bp of an indel remain; indel-proximate regions are listed in Identified_Indel_Regions.txt
- Every retained sample-genotype call is heterozygous with ≥ 20 reads and minor allele count ≥ 1% of total counts
- Reference allele ratio of the filtered set can be checked against VADT's reference allele bias report (unmasked ratios were ~51.8–51.9% across tissues)

## Limitations

- VCF files produced by callers other than GATK HaplotypeCaller may cause bugs, per the README
- Run times are long for large datasets (30 minutes to 10 hours), so HPC execution is recommended
- VADT is not validated on every dataset; the paper notes it 'was tested and validated on a different dataset (data not included)'
- Low overlap between RNA-seq variant calls and the 600K chicken genotyping panel excluded many SNPs, attributed to commercial genotyping panel limitations

## Evidence

- [methods] GATK hard filters: "using GATK to remove variants with “Fisher Strand > 30.0”, “Quality Depth < 2.0” and “Depth < 100”"
- [methods] sample-level filtering rationale: "variant calls are removed for samples that are homozygous, have low read count (coverage) or that have a minor allele count below 1% of the total counts"
- [methods] filtering parameters: "The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20"
- [readme] PASS-only retention: "The first filter performed is removing all variants that have been flagged as failing (FILTER column). Specifically the program looks for a "PASS" in the filter column"
- [abstract] informative SNP counts: "On average ~ 174,000 SNPs in each tissue passed our filtering criteria and were considered informative, of which ~ 24,000 (~ 14%) showed ASE."
