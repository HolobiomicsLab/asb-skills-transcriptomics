---
name: per-tissue-allele-ratio-comparison
description: Use when ASE variant calls are available for multiple tissues and you
  need to quantify reference-allele mapping bias, or to validate that masking a reference
  genome with called variants reduces inflated reference allele ratios toward 50%.
when_to_use_negative:
- Variant calls contain multiallelic SNPs, indels, or SNPs within 75 bp of an indel
  — these fail VADT filtering and their allele depths are not trustworthy for ratio
  computation
- Samples are homozygous or have read counts below the minimum threshold (20 reads),
  since heterozygous informative sites are required for a 50% expectation
- You need biological ASE interpretation rather than technical bias assessment — the
  ratio comparison only measures mapping bias, not true allelic imbalance
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0199
tools:
- name: VADT (VCF ASE Detection Tool)
  role: Tests informative variants for reference allele bias and reports the total
    (reference allele count / total allele count) for a VCF file after filtering
  repo: https://github.com/mjtiv/VADT
- name: Bedtools
  role: Masks the reference genome with variant positions from the global VCF to create
    the comparison (masked) reference
- name: STAR 2.5.2b
  role: Two-pass aligner used identically for both masked and unmasked references
    so allele ratio differences reflect genome masking, not alignment settings
- name: GATK 3.7
  role: Variant calling pipeline (Split'N'Trim, Base Recalibration, HaplotypeCaller
    with g.vcf setting) producing the AD fields from which ratios are computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:785066ae5ec7cfe6f122ec3d8d8615c0f2e692d29170568252ed6e320f8c90a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Bedtools
  - GATK 3.7
  - Picard
  - STAR 2.5.2b
  - VADT (VCF ASE Detection Tool)
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# per-tissue-allele-ratio-comparison

## Summary

Quantifies reference-allele mapping bias in allele-specific expression studies by computing the mean reference allele ratio (reference allele count / total allele count) per tissue from raw AD fields, and comparing masked versus unmasked reference alignments to verify the ratio shifts toward the expected 50.0%.

## When to use

Use when ASE variant calls are available for multiple tissues and you need to quantify reference-allele mapping bias, or to validate that masking a reference genome with called variants reduces inflated reference allele ratios toward 50%.

## When NOT to use

- Variant calls contain multiallelic SNPs, indels, or SNPs within 75 bp of an indel — these fail VADT filtering and their allele depths are not trustworthy for ratio computation
- Samples are homozygous or have read counts below the minimum threshold (20 reads), since heterozygous informative sites are required for a 50% expectation
- You need biological ASE interpretation rather than technical bias assessment — the ratio comparison only measures mapping bias, not true allelic imbalance

## Inputs

- VCF/g.vcf files from GATK 3.7 HaplotypeCaller containing raw AD (allele depth) fields for informative biallelic SNPs, one set per tissue (breast muscle, abdominal fat, liver)
- Unmasked baseline alignments: STAR 2.5.2b two-pass alignments to Gallus_gallus_5.0 (Ensembl release 86)
- Masked-reference realignments: same STAR settings to the Bedtools-masked genome built from the global VCF (68 study g.vcf files plus 32 additional samples)

## Outputs

- Per-tissue mean reference allele ratios for masked and unmasked alignments (e.g., breast muscle 51.77% -> 50.17%, abdominal fat 51.93% -> 50.19%, liver 51.91% -> 50.17%)
- Tabulated per-tissue shift of reference allele ratio toward the expected 50.0% baseline

## How to apply

Compute the per-SNP reference allele ratio as reference allele count divided by total allele count using the raw AD allele depths from the GATK 3.7 HaplotypeCaller g.vcf outputs, restricted to informative biallelic SNPs that passed the VADT filters (quality score >= 20, read count >= 20 per sample, indel exclusion distance of 75 bp, minor allele count >= 1%). Average the ratios within each tissue separately (breast muscle, abdominal fat, liver) for both the unmasked Gallus_gallus_5.0 baseline and the Bedtools-masked genome realignment with the same STAR 2.5.2b two-pass settings. The rationale is that a reference genome carrying the reference base at variant sites preferentially attracts reference-allele reads, inflating the ratio above the 50% expectation for heterozygous sites; masking those sites should equalize mapping. Compare the per-tissue means of masked versus unmasked alignments and tabulate the shift toward 50.0%. VADT itself reports the total (reference allele count / total allele count) for a VCF file as its Part 2 reference allele bias detection step, which can be used to obtain these ratios.

## Related tools

- **VADT (VCF ASE Detection Tool)** (Tests informative variants for reference allele bias and reports the total (reference allele count / total allele count) for a VCF file after filtering) — https://github.com/mjtiv/VADT
- **Bedtools** (Masks the reference genome with variant positions from the global VCF to create the comparison (masked) reference)
- **STAR 2.5.2b** (Two-pass aligner used identically for both masked and unmasked references so allele ratio differences reflect genome masking, not alignment settings)
- **GATK 3.7** (Variant calling pipeline (Split'N'Trim, Base Recalibration, HaplotypeCaller with g.vcf setting) producing the AD fields from which ratios are computed)

## Examples

```
python VADT.py --File_Name masked_tissue.vcf --Indel_Exclusion_Region_Length 75 --Quality_Score_Minimum_for_Variants 20 --Minimum_Read_Counts 20 --Meta_BH_adj_p_value_cutoff 0.05 --Binomial_Probability_Value 0.5
```

## Evaluation signals

- Masked-reference mean reference allele ratio must move toward 50% in every tissue, reproducing the reported 51.77% -> 50.17% in breast muscle, 51.93% -> 50.19% in abdominal fat, and 51.91% -> 50.17% in liver
- All included SNPs must be biallelic, PASS-filtered, at least 75 bp from an indel, quality >= 20, and have >= 20 reads per sample
- The unmasked baseline ratios should be consistently above 50% (here ~51.8–51.9%) before any masking is applied
- Masked and unmasked pipelines must use identical STAR 2.5.2b two-pass settings, Picard 2.8.1 duplicate marking, and the GATK 3.7 workflow so the only differing variable is the reference genome

## Limitations

- Breast muscle samples had substantially more reads flagged as 'too short' by STAR (avg. 15.39% vs 4.50% for abdominal fat and 5.33% for liver), indicating mapping issues with muscle tissue likely tied to the current genome build
- Low overlap between RNA-seq variant calls and the 600K chicken genotyping panel (GSE131764, filtered at 97% call rate) excluded a large number of SNPs
- Reference allele ratios approaching 50% indicate reduced mapping bias but do not distinguish reduced bias from genuine allelic imbalance in ASE detection

## Evidence

- [methods] Masking the reference genome reduced reference allele ratio toward 50% in all three tissues: "masked reference genome reduced the reference allele ration from 51.77 to 50.17% in breast muscle, 51.93% to 50.19% in abdominal fat and 51.91% to 50.17% in liver"
- [methods] Reference genome masking workflow: "The global vcf file was then used to mask the reference genome using Bedtools36. All samples were re-aligned to the masked genome"
- [methods] STAR alignment settings held constant: "aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting29,30"
- [readme] VADT reference allele bias detection: "VADT tests all "testable" variants that pass the prior filters for reference allele bias and reports the final total of (reference allele count / total allele count) for a VCF file."
