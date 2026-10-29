---
name: fishers-exact-enrichment-testing
description: Use when testing whether a lead variant set (e.g. sQTLs at FDR < 0.05)
  is enriched in discrete annotation categories relative to matched non-sQTLs or intron
  sets, and you need odds ratios with Fisher's exact p-values.
when_to_use_negative:
- When variants need to be matched on more than location bin and MAF, or when a regression
  framework accounting for covariates/confounders is required (e.g. fgwas-style functional
  enrichment modeling) rather than simple 2x2 contingency tests
- When testing enrichment of continuous or non-categorical scores rather than discrete
  annotation membership
- When the variant sets cannot be validly matched/controlled — unbalanced MAF or LD
  structure between cases and controls can inflate Fisher's exact odds ratios spuriously
edam_operation: http://edamontology.org/operation_0280
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0622
tools:
- name: sQTLseekeR2
  role: Produces the sQTL catalog (nominal and permuted sqtlseeker2-nf outputs) that
    serves as the case variant set for Fisher's exact enrichment tests
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: Nextflow
  role: Executes the sqtlseeker2-nf pipeline whose sqtls-fdr and all-tests outputs
    feed the contingency tables
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:7679f94c3fd260051821c2da893358facfd19388d04c3de28490aaafc0297ced
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sqtlseeker2-nf
  tools:
  - Docker
  - Nextflow
  - sQTLseekeR2
  edam_operation: http://edamontology.org/operation_0280
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# fishers-exact-enrichment-testing

## Summary

Apply two-sided Fisher's exact tests on 2x2 contingency tables to test whether sQTL variants are over-represented in genomic/annotation categories (post-transcriptionally spliced introns, splice donor/acceptor sites, H3K36me3 marks, GWAS LD hits) relative to matched controls, reporting odds ratios and FDR-corrected p-values.

## When to use

Use when testing whether a lead variant set (e.g. sQTLs at FDR < 0.05) is enriched in discrete annotation categories relative to matched non-sQTLs or intron sets, and you need odds ratios with Fisher's exact p-values.

## When NOT to use

- When variants need to be matched on more than location bin and MAF, or when a regression framework accounting for covariates/confounders is required (e.g. fgwas-style functional enrichment modeling) rather than simple 2x2 contingency tests
- When testing enrichment of continuous or non-categorical scores rather than discrete annotation membership
- When the variant sets cannot be validly matched/controlled — unbalanced MAF or LD structure between cases and controls can inflate Fisher's exact odds ratios spuriously

## Inputs

- sQTL catalog from sqtlseeker2-nf nominal output (sqtls-fdr file, FDR threshold 0.05)
- All-tests nominal results file from the sqtlseeker2-nf result directory
- Matched non-sQTL null sets (1,000 sets matched on within-gene location bin and MAF +/- 0.02)
- Intron classification as co- vs post-transcriptionally spliced (ENCODE nuclear/cytosolic RNA-seq, 13 cell lines)
- Annotation tracks: Ensembl regulatory, GENCODE v19 exons, VEP variant consequences, H3K36me3 histone marks, GWAS Catalog variants in LD r2 >= 0.80

## Outputs

- Per-category odds ratio, two-sided Fisher's exact p-value, and FDR-adjusted p-value (e.g. fig3a_enrichment_table.tsv with metric_donor_enrichment, metric_acceptor_enrichment, metric_h3k36me3_enrichment, metric_gwas_enrichment columns)
- Ranked enrichment table with one row per annotation category, thresholded at FDR < 0.05
- Odds ratios for sQTL enrichment in post-transcriptionally spliced introns (all sQTLs and non-eQTL sQTLs)

## How to apply

First define the case and control variant sets: lead sQTLs from the sqtlseeker2-nf nominal output (sqtls-fdr at FDR 0.05) against 1,000 matched non-sQTL null sets matched on within-gene location bin and MAF within +/-0.02, or intersect lead sQTLs with co- vs post-transcriptionally spliced intron sets classified via ENCODE nuclear/cytosolic RNA-seq splicing kinetics across 13 cell lines. Intersect both sets with the annotations of interest (Ensembl regulatory, GENCODE v19 exons, VEP consequences, H3K36me3, GWAS Catalog variants in LD r2 >= 0.80). Build 2x2 contingency tables per category (case vs control x annotated vs non-annotated) and compute two-sided Fisher's exact tests and odds ratios in R. Correct p-values for multiple testing at FDR < 0.05 and rank categories by enrichment. Run separate tests for all sQTLs and for the subset that are not eQTLs, since enrichment strengthens for splicing-specific variants (OR 1.47 vs 1.67 in ps introns).

## Related tools

- **sQTLseekeR2** (Produces the sQTL catalog (nominal and permuted sqtlseeker2-nf outputs) that serves as the case variant set for Fisher's exact enrichment tests) — https://github.com/guigolab/sqtlseeker2-nf
- **Nextflow** (Executes the sqtlseeker2-nf pipeline whose sqtls-fdr and all-tests outputs feed the contingency tables) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
fisher.test(matrix(c(a,b,c,d), nrow=2), alternative="two.sided")
```

## Evaluation signals

- fig3a_enrichment_table.tsv exists with one row per annotation category containing odds_ratio, p_value, and fdr fields
- All reported FDR values fall below the 0.05 threshold used for significant enrichment calls
- Odds ratio for splice donor enrichment exceeds the reported OR = 12.98 (acceptor OR = 12.23, H3K36me3 OR = 2.85, GWAS LD OR = 2.08)
- sQTL enrichment odds ratio in post-transcriptionally spliced introns exceeds 1, reproducing OR = 1.47 (all sQTLs) and OR = 1.67 (non-eQTL sQTLs)
- Counts underlying each 2x2 table reconcile: sQTLs and matched nulls sum to the expected set sizes at MAF +/- 0.02 matching tolerance

## Limitations

- Fisher's exact tests on 2x2 tables cannot model covariates or LD structure; matching is limited to within-gene location bin and MAF within +/-0.02
- Unannotated or poorly annotated categories (e.g. incomplete GWAS Catalog coverage) can bias odds ratios
- Permutation-based nulls (max_perm=1000 in sqtlseeker2-nf) bound the resolution of empirical p-values feeding downstream enrichment tests

## Evidence

- [results] sQTLs are enriched in post-transcriptionally spliced (ps) introns: 9.2% of variants in ps introns are sQTLs versus 6.6% in co-transcriptionally spliced introns (OR = 1.47, Fisher's exact p < 10-16): [evidence span withheld — anchor status: not_found]
- [methods] Fisher's exact test workflow: [evidence span withheld — anchor status: not_found]
- [methods] Null matching strategy: [evidence span withheld — anchor status: not_found]
- [results] Reported enrichment magnitudes: [evidence span withheld — anchor status: not_found]
- [abstract] Paper's own terminology for splicing regulation hotspots: "sQTLs tend to be located in post-transcriptionally spliced introns, which would function as hotspots for splicing regulation."
