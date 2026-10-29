---
name: indel-proximity-exclusion
description: Use when filtering a VCF for allele-specific expression analysis and
  SNPs must be excluded if they fall within a user-defined base-pair window of an
  indel.
when_to_use_negative:
- Input VCF has already been indel-filtered or contains only biallelic SNPs with no
  indels within the exclusion window
- The goal is indel-level ASE or structural variant analysis rather than biallelic
  SNP allele-specific expression
- Variant calls come from a non-GATK caller whose VCF formatting may break VADT (the
  README warns VADT VCFs were produced by HaplotypeCaller)
edam_operation: http://edamontology.org/operation_3673
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Performs the indel-proximity exclusion (--Indel_Exclusion_Region_Length 75)
    as part of its first filtering pass before binomial ASE testing
  repo: https://github.com/mjtiv/VADT
- name: GATK 3.7
  role: Upstream producer of the HaplotypeCaller g.vcf-based VCF that the indel-proximity
    filter is applied to
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:f2418d4baea6b123f94384ead52db5df2fe4cd4e3f2e221cd784332707da6b0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - GATK 3.7
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3673
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# indel-proximity-exclusion

## Summary

Removes SNPs located within a user-defined distance of indels (and removes the indels themselves) prior to ASE testing with VADT, because indel alignment artifacts can corrupt neighboring SNP allele counts. In the broiler ASE study this exclusion window was set to 75 base pairs, the RNA-seq read length.

## When to use

Use when filtering a VCF for allele-specific expression analysis and SNPs must be excluded if they fall within a user-defined base-pair window of an indel.

## When NOT to use

- Input VCF has already been indel-filtered or contains only biallelic SNPs with no indels within the exclusion window
- The goal is indel-level ASE or structural variant analysis rather than biallelic SNP allele-specific expression
- Variant calls come from a non-GATK caller whose VCF formatting may break VADT (the README warns VADT VCFs were produced by HaplotypeCaller)

## Inputs

- Per-tissue multi-sample VCF produced by merging GATK HaplotypeCaller g.vcf files after GATK hard filtering (Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100)
- VADT parameter file or command-line parameters specifying Indel_Exclusion_Region_Length (75) and Quality_Score_Minimum_for_Variants (20)

## Outputs

- VADT intermediate file Identified_Indel_Regions.txt (indel regions used for exclusion)
- VADT intermediate file Other_Failing_RNA_Seq_variants.txt (SNPs removed, including indel-proximity exclusions)
- Filtered set of testable informative biallelic SNPs carried forward to binomial ASE testing

## How to apply

During VADT's first filtering pass on the per-tissue VCF, drop all variants in indel regions and remove any biallelic SNP within the indel exclusion region before testing for ASE with a binomial test. Set --Indel_Exclusion_Region_Length to 75 base pairs, matching the read length, so that SNPs whose read alignment could be distorted by a nearby indel are not treated as informative. Combine this with removal of GATK-failing variants, multiallelic SNPs, and variants below the minimum quality score of 20 in the same pass. Emitting Identified_Indel_Regions.txt and Other_Failing_RNA_Seq_variants.txt documents which SNPs were excluded by this step. Only the remaining 'testable' SNPs enter sample-level filtering (homozygous calls, read counts < 20, minor allele count < 1%) and downstream ASE analysis.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Performs the indel-proximity exclusion (--Indel_Exclusion_Region_Length 75) as part of its first filtering pass before binomial ASE testing) — https://github.com/mjtiv/VADT
- **GATK 3.7** (Upstream producer of the HaplotypeCaller g.vcf-based VCF that the indel-proximity filter is applied to)

## Examples

```
python VADT_1.0 --File_Name tissue.vcf --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20
```

## Evaluation signals

- VADT emits Identified_Indel_Regions.txt and Other_Failing_RNA_Seq_variants.txt; every SNP removed by this filter should lie within 75 bp of a listed indel
- All variants surviving the filter are biallelic, GATK 'PASS', and have quality >= 20
- Per-tissue counts of remaining informative biallelic SNPs should reproduce the published values: 148,860 (breast muscle), 217,628 (abdominal fat), 155,875 (liver), averaging ~174,000
- Reference allele ratio in the ASE results should stay near 50% (reference allele bias check), as reported after genome masking (50.17–50.19% across tissues)

## Limitations

- The exclusion distance is a user parameter (set to the read length, 75 bp); an inappropriate window may either leave corrupted SNPs or over-filter informative variants
- VADT expects VCFs formatted by GATK HaplotypeCaller; other callers may cause parsing bugs in this filtering step
- Removing all indel-adjacent SNPs sacrifices variant density, contributing to a large number of SNPs being excluded overall

## Evidence

- [methods] VADT first filters out variants marked as failing by GATK, indels, SNPs within a certain distance of an indel (user defined), multiallelic SNPs: "VADT first filters out variants marked as failing by GATK, indels, SNPs within a certain distance of an indel (user defined), multiallelic SNPs"
- [methods] The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20: "The present study utilized an indel filter distance of 75 base pairs (read length), a quality score minimum of 20"
- [readme] VADT also removes all variants within a certain distance of an indel defined by the user.: "VADT also removes all variants within a certain distance of an indel defined by the user."
- [readme] possible sequencing errors caused by INDEL alignment issues that could affect neighboring variants: "possible sequencing errors caused by INDEL alignment issues
	that could affect neighboring variants."
