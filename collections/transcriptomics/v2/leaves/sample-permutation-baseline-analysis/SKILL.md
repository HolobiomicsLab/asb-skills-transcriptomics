---
name: sample-permutation-baseline-analysis
description: Use when validating RNA-seq variant calls against an orthogonal genotyping
  platform and you need a null (randomly-paired) concordance baseline to prove that
  high self-by-self genotype agreement is not an artifact of SNP overlap alone.
when_to_use_negative:
- You have not yet produced correctly matched self-by-self concordance values — the
  permutation baseline is only meaningful as a contrast to the true pairing result
- The two VCFs contain no overlapping samples between genotyping and RNA-seq data,
  since shuffling pairings requires a valid matching key of genuine sample correspondences
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3516
- http://edamontology.org/topic_0622
tools:
- name: Compare_VCF_Files
  role: Program that filters the genotyping and RNA-seq VCFs, matches overlapping
    SNPs, compares genotypes of sample pairs, and is re-run with shuffled sample pairings
    to compute the random-pairing baseline concordance
  repo: https://github.com/mjtiv/Compare_VCF_Files
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-83459-8
    title: Investigation of allele specific expression in various tissues of broiler
      chickens using the detection tool VADT
  source_content_hash: sha256:693575364ac30a6eca701a21e57a9aa72a6541e417b7f2f563fc19eab4569fdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mjtiv/Compare_VCF_Files
  tools:
  - Axiom Analysis Suite Software
  - Compare_VCF_Files
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3516
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Sample permutation baseline analysis

## Summary

Builds a random-pairing baseline concordance matrix by shuffling sample pairings between two VCFs (e.g., Axiom 600K genotyping array calls from GSE131764 and RNA-seq variant calls from 68 broiler samples) so that true self-by-self concordance can be contrasted against chance-level genotype agreement.

## When to use

Use when validating RNA-seq variant calls against an orthogonal genotyping platform and you need a null (randomly-paired) concordance baseline to prove that high self-by-self genotype agreement is not an artifact of SNP overlap alone.

## When NOT to use

- You have not yet produced correctly matched self-by-self concordance values — the permutation baseline is only meaningful as a contrast to the true pairing result
- The two VCFs contain no overlapping samples between genotyping and RNA-seq data, since shuffling pairings requires a valid matching key of genuine sample correspondences

## Inputs

- Genotyping VCF exported from Axiom Analysis Suite Software (version 3.0.1, 64-bit, Best Practices Workflow for agricultural animals, Gallus gallus 5.0) with SNPs < 97% call rate removed
- RNA-seq multi-sample VCF from GATK 3.7 variant calling of the n = 68 sample subset, filtered on Fisher Strand > 30.0, Quality Depth < 2.0, Depth < 100, SNPs within 75 bp of INDELs, quality score < 20, read count < 20
- Sample matching key file relating genotyping panel sample IDs to RNA-seq sample IDs (used as the source of pairings to shuffle)

## Outputs

- Random-pairing baseline concordance matrix (per shuffled sample pair concordance values)
- Contrast of average shuffled-pair concordance (e.g., 54.30%) against self-by-self concordance (e.g., 99.50%)
- Updated genotype_concordance_report.csv containing per-sample and average concordance values for both matched and shuffled configurations

## How to apply

After running the concordance comparison (Compare_VCF_Files) on correctly matched sample pairs, reshuffle the sample ID pairings from the matching key file so each genotyping sample is compared against the wrong RNA-seq sample, and recompute per-sample concordance over the same filtered, overlapping SNP set (~13,997 SNPs on average). Repeating this for permutations yields a random-pairing baseline concordance matrix that reflects expected agreement by chance given allele frequencies and genotyping error. The rationale is that correctly matched pairs should show self-by-self concordance near the reported 99.46% (99.50% self-by-self), whereas shuffled pairs should drop to a low baseline (54.30% in this study). Report both matrices together (e.g., in genotype_concordance_report.csv) so the gap between true and baseline concordance serves as the validation statistic.

## Related tools

- **Compare_VCF_Files** (Program that filters the genotyping and RNA-seq VCFs, matches overlapping SNPs, compares genotypes of sample pairs, and is re-run with shuffled sample pairings to compute the random-pairing baseline concordance) — https://github.com/mjtiv/Compare_VCF_Files

## Examples

```
python Compare_VCF_Files.py --Genotyping_VCF axiom_gse131764.vcf --Probeset_ID passing_probesets.txt --RNA_Seq_VCF rnaseq_variants.vcf --Matching_Key shuffled_key.txt --Exclusion_Region_Length 75 --Minimum_Quality_Score 20 --Minimum_Read_Count 20
```

## Evaluation signals

- Average self-by-self concordance approximates the reported 99.46% (99.50% self-by-self)
- Randomly-paired (shuffled) baseline concordance falls near the reported 54.30%, far below the self-by-self value
- Per-sample concordance values for shuffled pairs cluster around the low baseline rather than the high matched values
- Both matched and shuffled matrices are computed over the same overlapping SNP set (~13,997 SNPs on average), so differences reflect pairing not variant content
- Results are written to genotype_concordance_report.csv with per-sample and average concordance values

## Limitations

- Low overlap between RNA-seq variant calls and the commercial 600K chicken genotyping panel restricts the SNP set available for concordance estimation
- Baseline concordance is dataset- and allele-frequency-dependent; the 54.30% baseline should not be reused as a universal null for other datasets
- Shuffled pairings assume the original matching key correctly associates genotyping and RNA-seq sample IDs; key errors invalidate both matched and baseline results

## Evidence

- [methods] compares concordance of matching samples and SNPs: "compares concordance of matching samples and SNPs (https://github.com/mjtiv/Compare_VCF_Files)"
- [abstract] using variants called from a large sub-set of the samples (n = 68): "using variants called from a large sub-set of the samples (n = 68)"
- [readme] Once the matching variants are identified the program then compares genotypes of matching samples: [evidence span withheld — anchor status: external_doc]
- [readme] compares the overall genotyping calls for matching samples and reports all the final statistics: [evidence span withheld — anchor status: external_doc]
