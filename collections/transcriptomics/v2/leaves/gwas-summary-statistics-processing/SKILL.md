---
name: gwas-summary-statistics-processing
description: Use when per-trait GWAS summary statistics (SNP coordinates and association
  P-values) must be annotated with sQTL vs expression-only labels and tested for annotation
  enrichment of splicing effects.
when_to_use_negative:
- The GWAS does not provide genome-wide summary statistics (only a list of significant
  loci) — fgwas enrichment works best when there are at least ~20 independent loci
  with p-values less than 5e-8
- The goal is to fine-map causal variants within a locus — fgwas tests annotation
  enrichment across the genome, not variant-level causal inference
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: fgwas
  role: Fits maximum likelihood genomic annotation enrichment models contrasting the
    sQTL binary annotation against expression-only variants using the GWAS summary
    statistics
  repo: https://github.com/joepickrell/fgwas
- name: sQTLseekeR2
  role: Provides the GTEx sQTL catalog used to classify variants into sQTL versus
    expression-only annotation tracks
  repo: https://github.com/guigolab/sQTLseekeR2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:6b4b4857cb8eec2520ea12337c95167577705dee880e6e1640d2645a14e6a1f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joepickrell/fgwas
  tools:
  - fgwas
  - sQTLseekeR2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gwas-summary-statistics-processing

## Summary

Compile genome-wide GWAS summary statistics into fgwas bed-format inputs and fit fgwas annotation-enrichment models to test whether sQTLs show stronger GWAS association signal than expression-only variants across traits such as asthma, breast cancer, heart rate and height.

## When to use

Use when per-trait GWAS summary statistics (SNP coordinates and association P-values) must be annotated with sQTL vs expression-only labels and tested for annotation enrichment of splicing effects.

## When NOT to use

- The GWAS does not provide genome-wide summary statistics (only a list of significant loci) — fgwas enrichment works best when there are at least ~20 independent loci with p-values less than 5e-8
- The goal is to fine-map causal variants within a locus — fgwas tests annotation enrichment across the genome, not variant-level causal inference

## Inputs

- Genome-wide GWAS summary statistics per trait (SNP ID, coordinates, association P-value) for eight traits (asthma, breast cancer, coronary artery disease, heart rate, height, LDL, rheumatoid arthritis, schizophrenia)
- sQTL mapping results from the GTEx sQTLseekeR2 catalog used to label splicing-affecting variants
- Gene expression QTL results used to define variants affecting expression only

## Outputs

- fgwas bed-format input files per trait (chr, start, end, snpId, association P-value), coordinate-sorted, with binary sQTL/expression-only annotation tracks
- fgwas_sQTL_effect_sizes.csv containing per-trait maximum likelihood effect-size estimates and 95% confidence intervals
- Fig. 5b-style forest plot comparing sQTL annotation enrichment across traits

## How to apply

Compile genome-wide GWAS summary statistics for each trait (e.g. asthma, breast cancer, coronary artery disease, heart rate, height, LDL, rheumatoid arthritis, schizophrenia) into fgwas bed-format files with chr, start, end, snpId and association P-value columns, sorted by coordinate. Classify variants into two binary functional annotation tracks derived from the GTEx sQTL mapping results (sQTLseekeR2 catalog): sQTLs versus variants affecting gene expression only. Fit fgwas genomic annotation enrichment models per trait by maximum likelihood, contrasting the sQTL annotation against the expression-only annotation, then extract the maximum likelihood effect-size estimates and their 95% confidence intervals for the sQTL annotation in each trait. Save the per-trait estimates and CIs to fgwas_sQTL_effect_sizes.csv and render a Fig. 5b-style forest plot for cross-trait comparison.

## Related tools

- **fgwas** (Fits maximum likelihood genomic annotation enrichment models contrasting the sQTL binary annotation against expression-only variants using the GWAS summary statistics) — https://github.com/joepickrell/fgwas
- **sQTLseekeR2** (Provides the GTEx sQTL catalog used to classify variants into sQTL versus expression-only annotation tracks) — https://github.com/guigolab/sQTLseekeR2

## Evaluation signals

- The fitted sQTL annotation effect sizes must be positive with 95% confidence intervals excluding zero for asthma and the traits reported in Fig. 5b
- Estimates should match the published maximum likelihood effect sizes for the sQTL annotation across the eight traits
- Input bed-format files must be sorted by coordinate and contain chr, start, end, snpId and association P-value columns
- The sQTL and expression-only annotation tracks must be encoded as disjoint binary functional annotations derived from the sQTL mapping results
- The output CSV contains one effect-size estimate with 95% CI per trait

## Limitations

- fgwas works best if there are at least ~20 independent loci with p-values less than 5e-8, so traits with fewer loci may yield underpowered enrichment tests
- sQTL and eQTL signals cannot be unambiguously separated when effects may be caused by two different variants in LD rather than the same variant
- The analysis depends on the GTEx sQTL catalog; different preprocessing and statistical methodologies could confound comparisons between sGene and eGene sets

## Evidence

- [readme] fgwas README quote on purpose: [evidence span withheld — anchor status: external_doc]
- [readme] fgwas README quote on requirements: [evidence span withheld — anchor status: external_doc]
- [abstract] Abstract finding on phenotypic impact: "Genetic variants affecting splicing can have a stronger phenotypic impact than those affecting gene expression."
- [abstract] Article finding on shared splicing/expression effects: "Many also affect the expression of the same or other genes, uncovering regulatory loci that act through different mechanisms."
