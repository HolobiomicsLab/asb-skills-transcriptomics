---
name: cis-eqtl-variant-filtering
description: Use when summary-level eQTL statistics must be converted into per-gene
  cis-window SNP sets and LD-clumped reference inputs for PRS/TWAS Stage I weight
  training from summary data.
when_to_use_negative:
- The analysis aims to include trans-eQTL effects — OTTERS only considers cis-eQTL
  effects and cannot incorporate trans signals
- Individual-level reference expression and genetic data are available — tools like
  PrediXcan or FUSION operate directly on individual-level data without this summary-based
  filtering step
- The eQTL summary statistics are already clumped/restricted to the target genes'
  cis-windows and formatted for the PRS tools
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3053
tools:
- name: OTTERS
  role: Integrated framework that prepares LD reference and eQTL summary statistics
    in the required formats and defines the STAGE I clumping threshold (--r2=0.99)
    in training.py
  repo: https://github.com/daiqile96/OTTERS
- name: PLINK 1.9
  role: Performs LD clumping of cis-window SNPs against the Exp_geno LD reference
    panel
  repo: https://www.cog-genomics.org/plink
- name: TABIX
  role: Extracts per-target-gene regions from summary data via extract_vcf_region_with_tabix
  repo: http://www.htslib.org/doc/tabix.html
- name: pandas
  role: Loads and sorts annotation and eQTL summary statistic tables and restricts
    SNPs to gene cis-windows
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:ea43517654852e9c56d88f6f21bcd3d331834d6e5806cd856156478a0c428f45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - OTTERS
  - PLINK 1.9
  - TABIX
  - pandas
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cis-eQTL variant filtering

## Summary

Filter and LD-clump cis-eQTL summary statistics with PLINK 1.9 against an LD reference panel so that SNPs are restricted to each gene's cis-window and formatted as valid inputs for the OTTERS PRS training methods (P+T, lassosum, SDPR, PRS-CS) before Stage I eQTL weight estimation.

## When to use

Use when summary-level eQTL statistics must be converted into per-gene cis-window SNP sets and LD-clumped reference inputs for PRS/TWAS Stage I weight training from summary data.

## When NOT to use

- The analysis aims to include trans-eQTL effects — OTTERS only considers cis-eQTL effects and cannot incorporate trans signals
- Individual-level reference expression and genetic data are available — tools like PrediXcan or FUSION operate directly on individual-level data without this summary-based filtering step
- The eQTL summary statistics are already clumped/restricted to the target genes' cis-windows and formatted for the PRS tools

## Inputs

- eQTL summary statistics file with columns CHROM, POS, A1, A2, Zscore, TargetID, N (sorted by chromosome then ascending position)
- Gene annotation file with columns CHROM, GeneStart, GeneEnd, TargetID
- LD reference panel in PLINK 1.9 format (Exp_geno.bed/.bim/.fam)
- P+T significance threshold for marginal eQTL p values

## Outputs

- clumped_variants_per_gene.txt in the STAGE I output directory (TargetID-keyed retained SNP set per gene)
- Per-gene cis-window SNP lists formatted for P+T, lassosum, SDPR, and PRS-CS input

## How to apply

Load the annotation file (CHROM, GeneStart, GeneEnd, TargetID) and eQTL summary statistics (CHROM, POS, A1, A2, Zscore, TargetID, N) with pandas, first verifying the summary statistics are sorted by chromosome and then by SNP position in ascending order. Restrict the summary statistics to the cis-window SNPs of each gene using the gene start/end coordinates in the annotation file. Rank SNPs within each gene's cis-window by eQTL Zscore significance and run PLINK 1.9 LD clumping against the LD reference panel (Exp_geno.bed/.bim/.fam) with the OTTERS training.py STAGE I threshold --r2=0.99, excluding SNPs whose marginal eQTL p value exceeds the P+T threshold and any SNP with LD r² > 0.99 with a more significant SNP in the same region. OTTERS integrates TABIX and PLINK 1.9 (via functions like extract_vcf_region_with_tabix and call_PLINK_extract) to extract per-gene regions efficiently, and writes the retained clumped SNP set per gene, keyed by TargetID, to clumped_variants_per_gene.txt in the STAGE I output directory. The rationale is that eQTL effect sizes are essentially SNP effect sizes from a reference 'GWAS' of gene expression, so PRS methods can be adapted once variants are properly restricted and LD-pruned.

## Related tools

- **OTTERS** (Integrated framework that prepares LD reference and eQTL summary statistics in the required formats and defines the STAGE I clumping threshold (--r2=0.99) in training.py) — https://github.com/daiqile96/OTTERS
- **PLINK 1.9** (Performs LD clumping of cis-window SNPs against the Exp_geno LD reference panel) — https://www.cog-genomics.org/plink
- **TABIX** (Extracts per-target-gene regions from summary data via extract_vcf_region_with_tabix) — http://www.htslib.org/doc/tabix.html
- **pandas** (Loads and sorts annotation and eQTL summary statistic tables and restricts SNPs to gene cis-windows)

## Examples

```
plink --bfile Exp_geno --clump cis_eqtl_sumstats.txt --clump-r2 0.99 --out STAGE1/clumped_variants_per_gene
```

## Evaluation signals

- eQTL summary statistics are confirmed sorted by chromosome, then by SNP position in ascending order, before filtering
- Every retained SNP falls within the cis-window (GeneStart–GeneEnd) coordinates of its TargetID gene in the annotation file
- No retained SNP has LD r² > 0.99 with a more significant SNP in the same gene region (PLINK --clump-r2 0.99)
- SNPs with marginal eQTL p value above the P+T threshold are excluded
- clumped_variants_per_gene.txt is TargetID-keyed and present in the STAGE I output directory

## Limitations

- OTTERS currently considers only cis-eQTL effects; trans-eQTL effects are not incorporated and their availability in summary-level form remains an obstacle
- Currently available eQTL summary statistics are mainly derived from individuals of European descent, limiting cross-population transferability
- The LD reference panel must match the variant region; extraction per gene relies on TABIX/PLINK 1.9 compatibility with the input formats

## Evidence

- [readme] integrated tool capabilities: "prepare the required inputs (LD reference and eQTL summary statistics) in the required formats for P+T, lassosum, SDPR, and PRS-CS"
- [readme] per-gene extraction: [evidence span withheld — anchor status: not_found]
- [intro] PRS adaptation rationale: [evidence span withheld — anchor status: not_found]
- [methods] Stage I training methods: "# STAGE I
  # train eQTL weights using P+T, lassosum, SDPR and PRS-CS."
