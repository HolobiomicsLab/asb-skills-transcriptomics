---
name: sqtl-catalog-functional-enrichment-analysis
description: Use when a catalog of sQTLs (e.g. sqtlseeker2-nf nominal/permuted output
  at FDR < 0.05) exists and the goal is to identify which functional annotations —
  splice sites, histone marks, GWAS hits — are over-represented among sQTLs versus
  matched non-sQTLs.
when_to_use_negative:
- No matched non-sQTL null sets can be generated (e.g. the sQTL catalog lacks MAF
  and within-gene location metadata needed for +/-0.02 MAF matching).
- The goal is fine-mapping or co-localization of sQTL and eQTL effects — the article
  notes currently available co-localization methods are not directly applicable within
  the multivariate non-parametric sQTL framework.
- Input is a raw expression/genotype dataset rather than a mapped sQTL catalog; sQTL
  mapping via sqtlseeker2-nf must be run first.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0625
tools:
- name: sQTLseekeR2
  role: R package producing the sQTL catalog (nominal and permuted/FDR passes) that
    is the input variant set for the enrichment analysis
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline wrapping sQTLseekeR2 that generates the sqtls-fdr and all-tests
    output files consumed by the enrichment workflow
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: LOFTEE
  role: VEP plugin for variant effect annotation (splice-site disruption calls) used
    to categorize sQTL functional consequences
  repo: https://github.com/konradjk/loftee
- name: fgwas
  role: Alternative framework for testing enrichment of functional genomic regions
    among genetic associations
  repo: https://github.com/joepickrell/fgwas
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:ff4215051152f77107768316ee6dcd2de448d5cb8552f7cb92bfc3d504f3a546
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - LOFTEE
  - fgwas
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# sqtl-catalog-functional-enrichment-analysis

## Summary

Tests whether sQTLs mapped with sQTLseekeR2/sqtlseeker2-nf are enriched in functional genome annotations (splice donor/acceptor sites, H3K36me3 marks, GWAS LD hits) relative to matched non-sQTL null variants, using 2x2 contingency tables and Fisher's exact tests with FDR correction.

## When to use

Use when a catalog of sQTLs (e.g. sqtlseeker2-nf nominal/permuted output at FDR < 0.05) exists and the goal is to identify which functional annotations — splice sites, histone marks, GWAS hits — are over-represented among sQTLs versus matched non-sQTLs.

## When NOT to use

- No matched non-sQTL null sets can be generated (e.g. the sQTL catalog lacks MAF and within-gene location metadata needed for +/-0.02 MAF matching).
- The goal is fine-mapping or co-localization of sQTL and eQTL effects — the article notes currently available co-localization methods are not directly applicable within the multivariate non-parametric sQTL framework.
- Input is a raw expression/genotype dataset rather than a mapped sQTL catalog; sQTL mapping via sqtlseeker2-nf must be run first.

## Inputs

- sQTL catalog from sqtlseeker2-nf (sqtls-fdr file, FDR threshold 0.05)
- all-tests nominal sQTL results file from sqtlseeker2-nf
- Ensembl regulatory annotations
- GENCODE v19 exon annotations
- VEP variant consequence annotations
- H3K36me3 histone mark intervals
- GWAS Catalog variants in LD r2 >= 0.80 with index variants

## Outputs

- fig3a_enrichment_table.tsv with one row per annotation category containing odds_ratio, p_value and fdr fields
- Metric columns metric_donor_enrichment, metric_acceptor_enrichment, metric_h3k36me3_enrichment, metric_gwas_enrichment
- Ranked top-enrichment annotation table (Fig. 3a style) with significant calls at FDR < 0.05

## How to apply

Load the sQTL catalog (sqtls-fdr output, FDR 0.05) and the all-tests nominal results from the sqtlseeker2-nf result directory. Generate 1,000 matched non-sQTL null sets, matching each sQTL on within-gene location bin and MAF within +/-0.02 to control for confounders. Intersect sQTLs and matched nulls with Ensembl regulatory annotations, GENCODE v19 exons, VEP variant consequences, H3K36me3 histone marks, and GWAS Catalog variants in LD r2 >= 0.80. Build 2x2 contingency tables per annotation category (sQTL vs null x annotated vs non-annotated), compute odds ratios with two-sided Fisher's exact tests in R, then correct p-values at FDR < 0.05 and rank annotations by enrichment to reproduce the Fig. 3a top-enrichment table. Rationale: splice-site enrichments (donor OR = 12.98, acceptor OR = 12.23) support the finding that many variants affect splicing by altering splice site sequences, while enrichments in RNA-binding protein contexts reflect other regulatory mechanisms.

## Related tools

- **sQTLseekeR2** (R package producing the sQTL catalog (nominal and permuted/FDR passes) that is the input variant set for the enrichment analysis) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline wrapping sQTLseekeR2 that generates the sqtls-fdr and all-tests output files consumed by the enrichment workflow) — https://github.com/guigolab/sqtlseeker2-nf
- **LOFTEE** (VEP plugin for variant effect annotation (splice-site disruption calls) used to categorize sQTL functional consequences) — https://github.com/konradjk/loftee
- **fgwas** (Alternative framework for testing enrichment of functional genomic regions among genetic associations) — https://github.com/joepickrell/fgwas

## Examples

```
./nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- fig3a_enrichment_table.tsv exists and contains one row per annotation category with odds_ratio, p_value and fdr fields
- All reported FDR values are below the 0.05 threshold used for significant enrichment calls
- Splice donor odds ratio is the largest enrichment (expected OR = 12.98) with splice acceptor OR = 12.23
- H3K36me3 enrichment reproduces at OR = 2.85 and GWAS LD enrichment at OR = 2.08
- Matched null sets are validated: each non-sQTL matches its sQTL on within-gene location bin and MAF within +/-0.02 across all 1,000 replicates

## Limitations

- Cannot distinguish whether paired sQTL/eQTL effects at the same locus are caused by the same variant or two different variants in LD, which complicates attribution of enrichment signals.
- Enrichment statistics depend on the choice of matching scheme (within-gene location bin, MAF +/-0.02) and the 1,000-null-set design; different matching schemes may shift odds ratios.
- LD-based variant clustering is disabled by default in sqtlseeker2-nf (--ld 0), so overlapping LD-linked sQTLs can inflate annotation overlaps unless clustered first.

## Evidence

- [abstract] While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins.: "While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins."
- [intro] we have developed a pipeline for efficient and reproducible sQTL mapping: "we have developed a pipeline for efficient and reproducible sQTL mapping"
- [readme] A pipeline for splicing quantitative trait loci (sQTL) mapping.: "A pipeline for splicing quantitative trait loci (sQTL) mapping."
- [discussion] currently available methods are not directly applicable within our multivariate, non-parametric framework: "currently available methods are not directly applicable within our multivariate, non-parametric framework"
