---
name: snp-null-set-matching-maf-and-location-bin
description: Use when you have a significant sQTL catalog (sqtlseeker2-nf FDR < 0.05)
  and need a variant control set to test whether sQTLs are enriched in splice sites,
  chromatin marks, or GWAS LD hits.
when_to_use_negative:
- When testing enrichment of eQTLs or other QTL classes without an equivalent matched
  null scheme — the matching bins are defined on the sQTL within-gene location distribution
- When variants cannot be intersected with the annotation tracks in a common genome
  build, since GENCODE v19 and the enrichment tracks assume consistent coordinates
- When a model-based functional enrichment framework (e.g. fgwas on GWAS summary statistics
  with >= ~20 independent loci) is already appropriate — the matched-null contingency
  approach is a simpler catalog-level alternative
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: sQTLseekeR2
  role: R package whose nominal and permuted passes produce the sQTL catalog and all-tests
    results that are sampled and matched to build the null sets
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: sqtlseeker2-nf
  role: Nextflow pipeline that generates the sqtls-fdr and all-tests output files
    consumed as enrichment-analysis inputs
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:99d006df7511ad78267753ad5449945f445591a71c69e8f178fb06e3e199d634
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sqtlseeker2-nf
  tools:
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# snp-null-set-matching-maf-and-location-bin

## Summary

Generates MAF- and within-gene-location-bin-matched non-sQTL null variant sets so functional enrichment of sQTLs (splice sites, H3K36me3, GWAS LD hits) can be tested with Fisher's exact tests rather than against unmatched genome-wide SNPs.

## When to use

Use when you have a significant sQTL catalog (sqtlseeker2-nf FDR < 0.05) and need a variant control set to test whether sQTLs are enriched in splice sites, chromatin marks, or GWAS LD hits.

## When NOT to use

- When testing enrichment of eQTLs or other QTL classes without an equivalent matched null scheme — the matching bins are defined on the sQTL within-gene location distribution
- When variants cannot be intersected with the annotation tracks in a common genome build, since GENCODE v19 and the enrichment tracks assume consistent coordinates
- When a model-based functional enrichment framework (e.g. fgwas on GWAS summary statistics with >= ~20 independent loci) is already appropriate — the matched-null contingency approach is a simpler catalog-level alternative

## Inputs

- sQTL catalog file (sqtls-fdr nominal output, FDR threshold 0.05) from sqtlseeker2-nf
- all-tests nominal sQTL results file from sqtlseeker2-nf
- Ensembl regulatory annotations (BED/GTF)
- GENCODE v19 exon annotations
- VEP variant consequence annotations (splice donor/acceptor)
- H3K36me3 histone mark peaks
- GWAS Catalog variants in LD r2 >= 0.80 with trait associations

## Outputs

- 1,000 matched non-sQTL null variant sets (per-sQTL MAF +/-0.02, within-gene location-bin matched)
- Per-annotation 2x2 contingency tables (sQTL vs null x annotated vs non-annotated)
- fig3a_enrichment_table.tsv with one row per annotation category and odds_ratio, p_value, and fdr fields

## How to apply

Load the sQTL catalog (sqtls-fdr nominal output, FDR threshold 0.05) and the all-tests nominal results file from the sqtlseeker2-nf result directory. Generate 1,000 matched non-sQTL null sets by matching each sQTL to non-sQTL variants on within-gene location bin and MAF within +/-0.02, which controls for the variant-property confounders that would otherwise inflate enrichment statistics. Intersect both sQTLs and matched nulls with Ensembl regulatory annotations, GENCODE v19 exons, VEP variant consequences (donor/acceptor), H3K36me3 histone marks, and GWAS Catalog variants in LD r2 >= 0.80. Build a 2x2 contingency table per annotation category (sQTL vs null x annotated vs non-annotated), compute odds ratios with two-sided Fisher's exact tests in R, and rank annotations by enrichment after FDR < 0.05 correction.

## Related tools

- **sQTLseekeR2** (R package whose nominal and permuted passes produce the sQTL catalog and all-tests results that are sampled and matched to build the null sets) — https://github.com/guigolab/sqtlseeker2-nf
- **sqtlseeker2-nf** (Nextflow pipeline that generates the sqtls-fdr and all-tests output files consumed as enrichment-analysis inputs) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
fisher.test(matrix(c(n_sqtl_annot, n_sqtl_nonannot, n_null_annot, n_null_nonannot), nrow=2), alternative="two.sided")
```

## Evaluation signals

- fig3a_enrichment_table.tsv exists and contains one row per annotation category with odds_ratio, p_value, and fdr fields
- All reported FDR values for significant enrichment calls are below the 0.05 threshold used for the calls
- Reproduced point estimates match the published enrichments: donor OR = 12.98, acceptor OR = 12.23, H3K36me3 OR = 2.85, GWAS LD r2 >= 0.80 OR = 2.08
- Each null set contains 1,000 matched non-sQTL variants, with each null variant within +/-0.02 MAF of its sQTL and in the same within-gene location bin
- Empirical p-values from the matched nulls are consistent with the two-sided Fisher's exact test p-values computed on the 2x2 contingency tables

## Limitations

- Null matching is limited to MAF and within-gene location bin; other confounders (LD structure, GC content, variant density) are not balanced
- The pipeline cannot distinguish whether co-located sQTL and eQTL effects are caused by the same variant or by two different variants in LD
- sQTLseekeR2's multivariate, non-parametric framework is not directly compatible with standard co-localization and fine-mapping methods, limiting downstream attribution of enriched signals

## Evidence

- [methods] Matched null construction for enrichment testing: [evidence span withheld — anchor status: not_found]
- [methods] Fisher's exact test contingency analysis: [evidence span withheld — anchor status: not_found]
- [results] Published enrichment magnitudes: [evidence span withheld — anchor status: not_found]
- [readme] Source pipeline of the sQTL catalog: "A pipeline for splicing quantitative trait loci (sQTL) mapping."
- [full_text] Multiple testing control rationale: [evidence span withheld — anchor status: not_found]
