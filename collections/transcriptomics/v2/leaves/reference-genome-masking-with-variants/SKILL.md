---
name: reference-genome-masking-with-variants
description: Use when RNA-seq-based ASE or allele-count analyses show inflated reference
  allele ratios (e.g., >51%) and you need to quantify or mitigate mapping bias before
  calling ASE from biallelic SNP allele depths.
when_to_use_negative:
- Input is whole-genome DNA-seq or already reference-unbiased data where no reference
  allele inflation is observed
- No variant call set is available to build a global VCF for masking, so variant positions
  cannot be defined
- The analysis targets non-biallelic or multi-allelic variants that VADT filtering
  would remove anyway
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: Bedtools
  role: Masks the reference genome by replacing bases at positions of variants in
    the global VCF
- name: STAR 2.5.2b
  role: Two-pass alignment of reads to both unmasked and masked Gallus_gallus_5.0
    genome builds for the before/after comparison
- name: GATK 3.7
  role: Split'N'Trim, Base Recalibration, and HaplotypeCaller (g.vcf setting) to call
    variants and produce the global VCF and masked-analysis calls
- name: Picard 2.8.1
  role: Marks duplicate reads on aligned BAMs in both masked and unmasked pipelines
- name: VADT
  role: Downstream ASE detection that quantifies reference allele bias (reference
    allele count / total allele count) and reports the bias reduction after masking
  repo: https://github.com/mjtiv/VADT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:0ab667ce302426c08e0e502a87431a834c0f25a33f4b76c572dd370a02410b1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/VADT
  tools:
  - Bedtools
  - GATK 3.7
  - Picard
  - Picard 2.8.1
  - STAR 2.5.2b
  - VADT
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# reference-genome-masking-with-variants

## Summary

Reduces reference-allele mapping bias in allele-specific expression (ASE) analyses by masking the reference genome (Gallus_gallus_5.0) at variant positions using Bedtools and re-aligning RNA-seq reads with STAR two-pass, shifting per-tissue reference allele ratios toward the expected 50%.

## When to use

Use when RNA-seq-based ASE or allele-count analyses show inflated reference allele ratios (e.g., >51%) and you need to quantify or mitigate mapping bias before calling ASE from biallelic SNP allele depths.

## When NOT to use

- Input is whole-genome DNA-seq or already reference-unbiased data where no reference allele inflation is observed
- No variant call set is available to build a global VCF for masking, so variant positions cannot be defined
- The analysis targets non-biallelic or multi-allelic variants that VADT filtering would remove anyway

## Inputs

- RNA-seq FASTQ files (e.g., 68 chicken broiler samples from SRA SRP058295 and SRP255211)
- Unmasked reference genome FASTA (Gallus_gallus_5.0, Ensembl release 86) and STAR index
- Global merged VCF of called variants (from all study g.vcf files plus additional samples)

## Outputs

- Masked reference genome FASTA with variant positions replaced
- Re-aligned BAMs from STAR two-pass alignment against the masked genome
- Per-tissue mean reference allele ratios (masked vs. unmasked) from raw AD allele depths at informative biallelic SNPs
- Re-called variants from the GATK 3.7 pipeline on the masked alignment

## How to apply

First establish an unmasked baseline: align all samples (e.g., 68 chicken RNA-seq samples from SRA SRP058295 and SRP255211) with STAR 2.5.2b in two-pass mode to the Gallus_gallus_5.0 build (Ensembl release 86), then run Picard 2.8.1 duplicate marking and the GATK 3.7 pipeline (Split'N'Trim, Base Recalibration, HaplotypeCaller with g.vcf setting). Merge all sample g.vcf files (plus any additional samples) into a global VCF, and use Bedtools to replace reference bases at every called variant position in the reference genome. Re-align all samples to the masked genome with identical STAR settings and repeat the GATK variant calling. Finally, for informative biallelic SNPs per tissue, compute the mean reference allele ratio (reference allele count / total allele count from raw AD allele depths) in both masked and unmasked analyses; the masked ratios should move toward 50% (reported: 51.77%→50.17% breast muscle, 51.93%→50.19% abdominal fat, 51.91%→50.17% liver), confirming reduced bias for downstream VADT binomial testing.

## Related tools

- **Bedtools** (Masks the reference genome by replacing bases at positions of variants in the global VCF)
- **STAR 2.5.2b** (Two-pass alignment of reads to both unmasked and masked Gallus_gallus_5.0 genome builds for the before/after comparison)
- **GATK 3.7** (Split'N'Trim, Base Recalibration, and HaplotypeCaller (g.vcf setting) to call variants and produce the global VCF and masked-analysis calls)
- **Picard 2.8.1** (Marks duplicate reads on aligned BAMs in both masked and unmasked pipelines)
- **VADT** (Downstream ASE detection that quantifies reference allele bias (reference allele count / total allele count) and reports the bias reduction after masking) — https://github.com/mjtiv/VADT

## Examples

```
bedtools maskfasta -fi Gallus_gallus_5.0.fa -bed global_variants.vcf_merged.bed -fo Gallus_gallus_5.0_masked.fa
```

## Evaluation signals

- Masked-reference mean reference allele ratios move toward 50.0% in every tissue (reported: 51.77%→50.17% breast muscle, 51.93%→50.19% abdominal fat, 51.91%→50.17% liver)
- Identical STAR two-pass settings and GATK 3.7 pipeline (Split'N'Trim, Base Recalibration, HaplotypeCaller g.vcf) applied to both masked and unmasked alignments so ratios are comparable
- Masking sites derived from a global VCF merged from all sample g.vcf files (68 study samples plus 32 additional samples)
- Per-tissue ratios computed only on informative biallelic SNPs (~174,000 per tissue passing filters) using raw AD allele depths
- VADT's reference allele bias report shows a lower total reference allele count / total allele count for the masked VCF

## Limitations

- Mapping issues with breast muscle samples (avg. 15.39% of reads flagged as too short by STAR vs. 4.50% abdominal fat and 5.33% liver), likely due to the current genome build and annotation of muscle-related genes, may confound bias estimates
- Masking depends on variant call completeness; the low overlap between RNA-seq variant calls and the 600K chicken genotyping panel (GSE131764, <97% call-rate SNPs removed) limits validation of called positions
- The masked genome requires re-running the full STAR + Picard + GATK pipeline, roughly doubling compute compared with a single unmasked analysis

## Evidence

- [methods] masked reference genome reduced the reference allele ration from 51.77 to 50.17% in breast muscle, 51.93% to 50.19% in abdominal fat and 51.91% to 50.17% in liver: "masked reference genome reduced the reference allele ration from 51.77 to 50.17% in breast muscle, 51.93% to 50.19% in abdominal fat and 51.91% to 50.17% in liver"
- [methods] The global vcf file was then used to mask the reference genome using Bedtools. All samples were re-aligned to the masked genome: "The global vcf file was then used to mask the reference genome using Bedtools36. All samples were re-aligned to the masked genome"
- [methods] aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting: "aligning sequencing reads using STAR (version 2.5.2b) with the two-pass setting29,30 to the Gallus_gallus_5.0 genome build"
- [readme] VADT tests all "testable" variants that pass the prior filters for reference allele bias and reports the final total of (reference allele count / total allele count) for a VCF file: "So, VADT tests all "testable" variants that pass the prior filters for reference allele bias and reports the final total of (reference allele count / total allele count) for a VCF file."
