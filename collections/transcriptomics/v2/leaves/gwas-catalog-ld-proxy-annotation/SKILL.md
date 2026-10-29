---
name: gwas-catalog-ld-proxy-annotation
description: Use when testing whether a variant catalog (e.g., GTEx sQTLs at FDR <
  0.05) is enriched for GWAS Catalog hits via LD proxies, alongside annotation-based
  enrichments like splice donor/acceptor or H3K36me3.
when_to_use_negative:
- When the goal is fine-mapping or identifying which functional annotation explains
  a GWAS signal (fgwas-style joint modeling) rather than simple overlap enrichment
  of sQTLs against LD proxies — the paper's multivariate framework does not support
  co-localization/fine-mapping methods
- When variants cannot be matched to suitable null sets on MAF and within-gene location,
  since unmatched LD overlap is confounded by MAF and position
- When LD structure is unknown for the study population, as r2 >= 0.80 proxy assignment
  requires population-appropriate LD references
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: sQTLseekeR2
  role: Produces the sQTL catalog (sqtls-fdr, FDR < 0.05) that is annotated against
    GWAS Catalog LD proxies
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline whose result directory provides the sQTL catalog and all-tests
    nominal results used as enrichment inputs
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: fgwas
  role: Alternative framework for testing whether GWAS associations are enriched in
    functional genomic regions; contextualizes the GWAS-LD enrichment analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:bdae808a2ff51ddfed0dac91bec0a91c48f9fc76f3a4392dca5204e72b0e4f49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - fgwas
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# GWAS Catalog LD proxy annotation

## Summary

Annotates sQTLs (and matched non-sQTL null sets) with GWAS Catalog variants in linkage disequilibrium (LD r2 >= 0.80) as one functional-enrichment category, so that the overlap of splicing variants with trait-associated loci can be tested via 2x2 contingency tables and Fisher's exact tests.

## When to use

Use when testing whether a variant catalog (e.g., GTEx sQTLs at FDR < 0.05) is enriched for GWAS Catalog hits via LD proxies, alongside annotation-based enrichments like splice donor/acceptor or H3K36me3.

## When NOT to use

- When the goal is fine-mapping or identifying which functional annotation explains a GWAS signal (fgwas-style joint modeling) rather than simple overlap enrichment of sQTLs against LD proxies — the paper's multivariate framework does not support co-localization/fine-mapping methods
- When variants cannot be matched to suitable null sets on MAF and within-gene location, since unmatched LD overlap is confounded by MAF and position
- When LD structure is unknown for the study population, as r2 >= 0.80 proxy assignment requires population-appropriate LD references

## Inputs

- sQTL catalog from sqtlseeker2-nf (sqtls-fdr nominal output, FDR < 0.05)
- All-tests nominal results file from sqtlseeker2-nf
- 1,000 matched non-sQTL null sets (matched on within-gene location bin and MAF +/-0.02)
- GWAS Catalog variant list with LD information (r2 >= 0.80 proxies)
- Optional parallel annotation tracks: Ensembl regulatory annotations, GENCODE v19 exons, VEP consequence calls, H3K36me3 histone marks

## Outputs

- fig3a_enrichment_table.tsv with one row per annotation category, including a metric_gwas_enrichment (GWAS LD proxy) odds ratio, alongside metric_donor_enrichment, metric_acceptor_enrichment, and metric_h3k36me3_enrichment
- Per-category odds_ratio, p_value, and FDR fields from two-sided Fisher's exact tests
- Ranked top-enrichment table (Fig. 3a style) with GWAS LD OR ~ 2.08 among significant categories

## How to apply

Start from the sqtlseeker2-nf result directory: load the sQTL catalog (sqtls-fdr nominal output at FDR threshold 0.05) and the all-tests nominal results file. Generate 1,000 matched non-sQTL null sets, matching each sQTL on within-gene location bin and MAF within +/-0.02, so that enrichment is not driven by MAF or genomic-position confounders. Intersect both sQTLs and each null set with GWAS Catalog variants in LD r2 >= 0.80 (as proxies for trait-associated loci), in parallel with other annotation tracks (Ensembl regulatory annotations, GENCODE v19 exons, VEP variant consequences, H3K36me3 histone marks). For each annotation category, including the GWAS-LD category, build a 2x2 contingency table (sQTL vs null x annotated vs non-annotated), compute odds ratios, and apply two-sided Fisher's exact tests in R. Correct P-values at FDR < 0.05 and rank annotations by enrichment; the GWAS-LD proxy category is expected to show a positive but moderate enrichment (OR ~ 2.08) compared with splice donor variants (OR ~ 12.98), reflecting the paper's finding that splicing variants overlap both splice-site sequences and trait-associated loci.

## Related tools

- **sQTLseekeR2** (Produces the sQTL catalog (sqtls-fdr, FDR < 0.05) that is annotated against GWAS Catalog LD proxies) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline whose result directory provides the sQTL catalog and all-tests nominal results used as enrichment inputs) — https://github.com/guigolab/sqtlseeker2-nf
- **fgwas** (Alternative framework for testing whether GWAS associations are enriched in functional genomic regions; contextualizes the GWAS-LD enrichment analysis)

## Examples

```
fisher.test(matrix(c(n_sqtl_annot, n_sqtl_not, n_null_annot, n_null_not), nrow=2), alternative="two.sided")
```

## Evaluation signals

- fig3a_enrichment_table.tsv exists and contains one row per annotation category with odds_ratio, p_value and fdr fields
- All reported FDR values are below the 0.05 threshold used for significant enrichment calls
- The GWAS-LD proxy enrichment odds ratio is in a moderate range (~2.08), clearly below splice donor (OR = 12.98) and acceptor (OR = 12.23) enrichments
- Matched null sets replicate on within-gene location bin and MAF within +/-0.02 of each sQTL, and each of the 1,000 nulls yields its own OR before aggregation
- Each category's 2x2 contingency table (sQTL vs null x annotated vs non-annotated) reconstructs the reported OR under a two-sided Fisher's exact test

## Limitations

- LD-based proxy annotation cannot establish that a GWAS hit and an sQTL share a causal variant: the article notes 'we cannot distinguish the cases in which the two effects are indeed caused by the same variant or by two different variants in LD'
- Enrichment significance depends on the matched-null construction; without MAF/location matching, overlap statistics are confounded
- The related fgwas-style joint functional modeling 'works best if there are at least ~20 independent loci with p-values less than 5e-8', which may not hold for the sQTL catalog being tested

## Evidence

- [other] GWAS LD proxy annotation step: [evidence span withheld — anchor status: not_found]
- [other] matched null construction: [evidence span withheld — anchor status: not_found]
- [other] statistical testing procedure: [evidence span withheld — anchor status: not_found]
- [discussion] LD causal-variant ambiguity caveat: "we cannot distinguish the cases in which the two effects are indeed caused by the same variant or by two different variants in LD"
- [readme] pipeline that produces the sQTL catalog: "A pipeline for splicing quantitative trait loci (sQTL) mapping."
