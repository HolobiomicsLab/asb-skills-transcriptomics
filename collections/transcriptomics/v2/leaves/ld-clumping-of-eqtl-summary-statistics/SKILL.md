---
name: ld-clumping-of-eqtl-summary-statistics
description: 'Use when OTTERS Stage I input preparation is needed: eQTL summary statistics
  (CHROM, POS, A1, A2, Zscore, TargetID, N) plus a gene annotation file must be cis-window
  restricted and LD-clumped at r2=0.99 before P+T/lassosum/SDPR/PRS-CS weight training.'
when_to_use_negative:
- Individual-level reference expression and genotype data are available, in which
  case PrediXcan/FUSION/TIGAR-style training bypasses summary-statistic clumping
- The goal is trans-eQTL modeling — OTTERS only considers cis-eQTL effects, and this
  clumping operates per-gene cis-window only
- The eQTL summary statistics are already cis-restricted and clumped for the same
  LD reference, in which case re-clumping at r2=0.99 is redundant
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: PLINK 1.9
  role: Performs the LD clumping step at --r2=0.99 against the LD reference panel
    (Exp_geno.bed/.bim/.fam) in OTTERS STAGE I
  repo: https://www.cog-genomics.org/plink
- name: OTTERS
  role: Integrated tool that orchestrates LD-clumping and prepares LD reference and
    eQTL summary statistics in the formats required by P+T, lassosum, SDPR, and PRS-CS
  repo: https://github.com/daiqile96/OTTERS
- name: pandas
  role: Loads and sorts the annotation and eQTL summary statistic tables and restricts
    SNPs to each gene's cis-window
- name: TABIX
  role: Integrated by OTTERS to extract input data per target gene more efficiently
    (e.g. extract_vcf_region_with_tabix)
  repo: http://www.htslib.org/doc/tabix.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:1882763b137dfa1d1cf8e87c51a43e8c76e8da2bd124c50b3b5d2b4056ee81bc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OTTERS
  - PLINK 1.9
  - TABIX
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ld-clumping-of-eqtl-summary-statistics

## Summary

Prepares cis-restricted, LD-clumped eQTL summary statistics as the shared input for OTTERS Stage I eQTL weight training with P+T, lassosum, SDPR, and PRS-CS. Uses PLINK 1.9 clumping at --r2=0.99 against an LD reference panel so redundant SNPs are removed before PRS-style training.

## When to use

Use when OTTERS Stage I input preparation is needed: eQTL summary statistics (CHROM, POS, A1, A2, Zscore, TargetID, N) plus a gene annotation file must be cis-window restricted and LD-clumped at r2=0.99 before P+T/lassosum/SDPR/PRS-CS weight training.

## When NOT to use

- Individual-level reference expression and genotype data are available, in which case PrediXcan/FUSION/TIGAR-style training bypasses summary-statistic clumping
- The goal is trans-eQTL modeling — OTTERS only considers cis-eQTL effects, and this clumping operates per-gene cis-window only
- The eQTL summary statistics are already cis-restricted and clumped for the same LD reference, in which case re-clumping at r2=0.99 is redundant

## Inputs

- eQTL summary statistics table with columns CHROM, POS, A1, A2, Zscore, TargetID, N (sorted by CHROM then ascending POS)
- gene annotation file with columns CHROM, GeneStart, GeneEnd, TargetID
- LD reference panel in PLINK binary format (Exp_geno.bed/.bim/.fam)

## Outputs

- clumped_variants_per_gene.txt: per-gene (TargetID-keyed) retained SNP set after LD clumping, written to the STAGE I output directory

## How to apply

Load the annotation file (CHROM, GeneStart, GeneEnd, TargetID) and the eQTL summary statistics (CHROM, POS, A1, A2, Zscore, TargetID, N) with pandas, verifying the summary statistics are sorted by chromosome and then by SNP position in ascending order. Restrict the summary statistics to each gene's cis-window using the GeneStart/GeneEnd coordinates keyed by TargetID. Within each gene's cis-window, rank SNPs by eQTL Zscore significance and run PLINK 1.9 LD clumping against the LD reference panel (Exp_geno.bed/.bim/.fam) with the OTTERS STAGE I threshold --r2=0.99, dropping any SNP in LD r2 > 0.99 with a more significant SNP and any SNP whose marginal eQTL p value exceeds the P+T threshold. Write the retained per-gene SNP set to clumped_variants_per_gene.txt in the STAGE I output directory; this clumped set then feeds P+T, lassosum, SDPR, and PRS-CS weight training. OTTERS integrates TABIX and PLINK 1.9 to extract input data per target gene efficiently.

## Related tools

- **PLINK 1.9** (Performs the LD clumping step at --r2=0.99 against the LD reference panel (Exp_geno.bed/.bim/.fam) in OTTERS STAGE I) — https://www.cog-genomics.org/plink
- **OTTERS** (Integrated tool that orchestrates LD-clumping and prepares LD reference and eQTL summary statistics in the formats required by P+T, lassosum, SDPR, and PRS-CS) — https://github.com/daiqile96/OTTERS
- **pandas** (Loads and sorts the annotation and eQTL summary statistic tables and restricts SNPs to each gene's cis-window)
- **TABIX** (Integrated by OTTERS to extract input data per target gene more efficiently (e.g. extract_vcf_region_with_tabix)) — http://www.htslib.org/doc/tabix.html

## Examples

```
plink --bfile Exp_geno --clump eqtl_sumstats.txt --clump-r2 0.99 --out clumped_variants_per_gene
```

## Evaluation signals

- eQTL summary statistics are verified sorted by CHROM and then by SNP POS in ascending order before filtering
- Every retained SNP falls within the cis-window (GeneStart, GeneEnd) of its TargetID gene in the annotation file
- No retained SNP has LD r2 > 0.99 with a more significant SNP in the same gene region (PLINK --r2=0.99 clump threshold)
- SNPs with marginal eQTL p value above the P+T threshold are excluded from the clumped set
- Output clumped_variants_per_gene.txt exists in the STAGE I output directory and is keyed by TargetID

## Limitations

- OTTERS currently considers only cis-eQTL effects; trans-eQTL effects are not incorporated, so clumping is cis-window-only
- Available eQTL summary statistics are mainly from individuals of European descent, limiting cross-population transferability of the resulting weights
- Retention of SNPs depends on the P+T significance threshold, so a stricter threshold yields a sparser clumped set

## Evidence

- [readme] prepare the required inputs (LD reference and eQTL summary statistics) in the required formats for P+T, lassosum, SDPR, and PRS-CS: "prepare the required inputs (LD reference and eQTL summary statistics) in the required formats for P+T, lassosum, SDPR, and PRS-CS"
- [intro] we adapt four representative summary-data-based PRS methods—p value thresholding with linkage disequilibrium (LD) clumping (P+T): "we adapt four representative summary-data-based PRS methods—p value thresholding with linkage disequilibrium (LD) clumping (P+T)26"
- [readme] integrate [TABIX](http://www.htslib.org/doc/tabix.html) and [PLINK 1.9](https://www.cog-genomics.org/plink) tools to extract input data per target gene more efficiently: "integrate [TABIX](http://www.htslib.org/doc/tabix.html) and [PLINK 1.9](https://www.cog-genomics.org/plink) tools to extract input data per target gene more efficiently"
- [methods] See the functions [extract_vcf_region_with_tabix](https://github.com/daiqile96/OTTERS/blob/main/ottersutils.py#L233): "See the functions [extract_vcf_region_with_tabix](https://github.com/daiqile96/OTTERS/blob/main/ottersutils.py#L233)"
- [methods] STAGE I: train eQTL weights using P+T, lassosum, SDPR and PRS-CS.: "# STAGE I
  # train eQTL weights using P+T, lassosum, SDPR and PRS-CS."
