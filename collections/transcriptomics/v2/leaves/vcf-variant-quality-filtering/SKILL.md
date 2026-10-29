---
name: vcf-variant-quality-filtering
description: Use when a raw multi-sample VCF (e.g., from GATK HaplotypeCaller) must
  be cleaned of low-confidence, indel-proximate, multiallelic, and homozygous calls
  before per-sample allele-based statistics such as ASE binomial tests.
when_to_use_negative:
- The input is a genotype-array VCF needing call-rate filtering instead (e.g., remove
  SNPs with < 97% call rates from the 600K Axiom panel), which is a different filtering
  regime
- The VCF is already restricted to informative testable biallelic SNPs and re-filtering
  would discard valid variants
- The downstream analysis requires indel or multiallelic variants, which this filter
  removes by design
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0199
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Performs the filtering of the VCF (GATK-fail removal, indel exclusion, quality
    and per-sample read/allele filters) before ASE testing
  repo: https://github.com/mjtiv/VADT
- name: GATK 3.7
  role: Produced the HaplotypeCaller VCF inputs and the variant-level hard filters
    (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100) whose 'failing' flags
    VADT removes
- name: Compare_VCF_Files
  role: Applies analogous user-defined VCF filters (Exclusion_Region_Length, Minimum_Quality_Score,
    Minimum_Read_Count) in the genotyping-concordance workflow
  repo: https://github.com/mjtiv/Compare_VCF_Files
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:252c2a4190b74502d33b27e72759d33c3e801d9ebec622eb509e180111b4fbac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Compare_VCF_Files
  - GATK 3.7
  - NumPy
  - SciPy
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# vcf-variant-quality-filtering

## Summary

A filtering scheme for multi-sample VCFs that removes GATK-flagged variants, indels and SNPs within 75 bp of indels, multiallelic SNPs, variants with Phred quality < 20, and per-sample calls that are homozygous, low-coverage (< 20 reads), or have minor allele count < 1%, leaving only 'informative' biallelic SNPs for downstream statistical testing.

## When to use

Use when a raw multi-sample VCF (e.g., from GATK HaplotypeCaller) must be cleaned of low-confidence, indel-proximate, multiallelic, and homozygous calls before per-sample allele-based statistics such as ASE binomial tests.

## When NOT to use

- The input is a genotype-array VCF needing call-rate filtering instead (e.g., remove SNPs with < 97% call rates from the 600K Axiom panel), which is a different filtering regime
- The VCF is already restricted to informative testable biallelic SNPs and re-filtering would discard valid variants
- The downstream analysis requires indel or multiallelic variants, which this filter removes by design

## Inputs

- Multi-sample VCF file in GATK HaplotypeCaller format (with FILTER and AD fields)
- User-defined parameters: indel exclusion region length (75 bp), minimum quality score (20), minimum read count (20)

## Outputs

- Filtered VCF containing only informative biallelic SNPs passing all variant- and sample-level criteria
- Per-tissue counts of informative SNPs (e.g., 148,860 breast muscle, 217,628 abdominal fat, 155,875 liver)

## How to apply

First remove every variant whose FILTER column is not 'PASS', then drop indels, multiallelic SNPs, and SNPs within a user-defined indel exclusion region (75 bp, chosen to match the read length) since indel misalignment corrupts neighboring variant calls. Apply a minimum Phred quality score of 20 and a per-sample read count threshold of 20 (reference + alternative depth from the AD field). At the sample level, remove calls from homozygous samples, samples with low read count, samples whose minor allele count is below 1% of total counts, and 'No data' calls. Track counts before and after filtering so the number of informative SNPs per tissue can be reported and validated (e.g., ~148,860 breast muscle, 217,628 abdominal fat, 155,875 liver in the broiler ASE study).

## Related tools

- **VADT (VCF ASE Detection Tool)** (Performs the filtering of the VCF (GATK-fail removal, indel exclusion, quality and per-sample read/allele filters) before ASE testing) — https://github.com/mjtiv/VADT
- **GATK 3.7** (Produced the HaplotypeCaller VCF inputs and the variant-level hard filters (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100) whose 'failing' flags VADT removes)
- **Compare_VCF_Files** (Applies analogous user-defined VCF filters (Exclusion_Region_Length, Minimum_Quality_Score, Minimum_Read_Count) in the genotyping-concordance workflow) — https://github.com/mjtiv/Compare_VCF_Files

## Examples

```
python VADT_1.0 --File_Name tissue_informative_snps.vcf --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Meta_BH_adj_p_value_cutoff 0.05 --Binomial_Probability_Value 0.5
```

## Evaluation signals

- Informative SNP counts per tissue fall near the reported values (~148,860 breast muscle, 217,628 abdominal fat, 155,875 liver; average ~174,000)
- No remaining variants carry a non-PASS FILTER flag, indels, multiallelic sites, or SNPs within 75 bp of an indel
- All retained samples for each variant have >= 20 read counts and a minor allele count >= 1% of total counts, with no homozygous calls
- Downstream ASE results are consistent with expectations: ~24,000 significant SNPs per tissue (~14% of informative SNPs), e.g., 20,775 breast muscle, 28,190 abdominal fat, 21,916 liver

## Limitations

- VADT expects VCFs produced by GATK HaplotypeCaller; VCFs from other callers may trigger bugs
- SNPs are excluded by proximity to indels and by multiallelic status, so a large number of variants can be lost; in the concordance analysis many SNPs were excluded due to low overlap with the 600K chicken genotyping panel
- Parameter values are study-specific (75 bp indel distance from read length, quality 20, 20 read count) and must be re-tuned for other read lengths or coverages

## Evidence

- [methods] VADT first filters out variants marked as failing by GATK, indels, SNPs within a certain distance of an indel (user defined), multiallelic SNPs: "VADT first filters out variants marked as failing by GATK, indels, SNPs within a certain distance of an indel (user defined), multiallelic SNPs"
- [methods] The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20: "The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20"
- [methods] variant calls are removed for samples that are homozygous, have low read count (coverage) or that have a minor allele count below 1% of the total counts: "variant calls are removed for samples that are homozygous, have low read count (coverage) or that have a minor allele count below 1% of the total counts"
- [readme] Sample level filtering consists of filtering based on removing all samples that are homozygous, low read count (user parameter), allele count is <1% of the total counts or "No data".: "Sample level filtering consists of filtering based on removing all samples that are homozygous, low read count (user parameter), allele count is <1% of the total counts or "No data"."
- [readme] Note: VCF files utilized by VADT were produced by GATK's Haplotypecaller and so VCF files produced by other programs could potentially have some bugs.: "Note: VCF files utilized by VADT were produced by GATK's Haplotypecaller and so VCF files produced by other programs could potentially have some bugs."
