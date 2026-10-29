---
name: qtl-variant-classification-sqtl-versus-eqtl
description: Use when GWAS summary statistics are available and you need to compare
  the phenotypic contribution of splicing-associated variants against variants affecting
  only gene expression via functional annotation enrichment.
when_to_use_negative:
- When the goal is to determine whether paired sQTL/eQTL effects are caused by the
  same variant or two different variants in LD — the article states this cannot be
  distinguished
- When fine-mapping or co-localization of causal variants is required — the authors
  note available methods are not directly applicable within their multivariate, non-parametric
  framework
- When the trait has fewer than ~20 independent loci with p-values < 5e-8, since fgwas
  works best on highly powered GWAS
tools:
- name: fgwas
  role: Fits genomic annotation enrichment models by maximum likelihood to test whether
    sQTL and expression-only annotations show effect sizes significantly different
    from zero in GWAS summary statistics
  repo: joepickrell/fgwas
- name: sQTLseekeR2
  role: R package producing the sQTL catalog used to define the sQTL annotation track
  repo: guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline used to map the GTEx sQTLs that populate the sQTL annotation
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:79f3fb6b7e8dd88f17e23f06f5ee200440495ed48f4941b8a779f2f9b575ce55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joepickrell/fgwas
  tools:
  - fgwas
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# qtl-variant-classification-sQTL-versus-eQTL

## Summary

Classifies GTEx variants into sQTL versus expression-only (eQTL) annotation tracks and tests their differential enrichment in GWAS disease/trait summary statistics using fgwas, to quantify how splicing variants contribute to phenotypes relative to expression-only variants.

## When to use

Use when GWAS summary statistics are available and you need to compare the phenotypic contribution of splicing-associated variants against variants affecting only gene expression via functional annotation enrichment.

## When NOT to use

- When the goal is to determine whether paired sQTL/eQTL effects are caused by the same variant or two different variants in LD — the article states this cannot be distinguished
- When fine-mapping or co-localization of causal variants is required — the authors note available methods are not directly applicable within their multivariate, non-parametric framework
- When the trait has fewer than ~20 independent loci with p-values < 5e-8, since fgwas works best on highly powered GWAS

## Inputs

- Genome-wide GWAS summary statistics for eight traits (asthma, breast cancer, coronary artery disease, heart rate, height, LDL, rheumatoid arthritis, schizophrenia)
- fgwas bed-format input files with chr, start, end, snpId and association P-value columns, sorted by coordinate
- sQTL mapping results from the GTEx sQTL catalog (sqtlseeker2-nf / sQTLseekeR2 output)
- Expression QTL (eQTL) results to define the expression-only variant class

## Outputs

- Binary functional annotation tracks encoding sQTLs versus expression-only variants for fgwas
- Per-trait maximum likelihood effect-size estimates and 95% confidence intervals (fgwas_sQTL_effect_sizes.csv)
- Fig. 5b-style forest plot of sQTL effect sizes across traits

## How to apply

Compile genome-wide GWAS summary statistics for the eight traits (asthma, breast cancer, coronary artery disease, heart rate, height, LDL, rheumatoid arthritis, schizophrenia) into fgwas bed-format input files with chr, start, end, snpId and association P-value columns, sorted by coordinate. Classify variants into two binary functional annotation tracks using the sQTL catalog mapped from GTEx data (via the sqtlseeker2-nf / sQTLseekeR2 pipeline): sQTLs versus variants affecting gene expression only. Fit fgwas genomic annotation enrichment models per trait by maximum likelihood, contrasting the sQTL annotation against the expression-only annotation. Extract maximum likelihood effect-size estimates and 95% confidence intervals for the sQTL annotation in each trait, save them to fgwas_sQTL_effect_sizes.csv, and render a Fig. 5b-style forest plot. fgwas requires at least ~20 independent loci with p-values below 5e-8 for adequate power.

## Related tools

- **fgwas** (Fits genomic annotation enrichment models by maximum likelihood to test whether sQTL and expression-only annotations show effect sizes significantly different from zero in GWAS summary statistics) — joepickrell/fgwas
- **sQTLseekeR2** (R package producing the sQTL catalog used to define the sQTL annotation track) — guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline used to map the GTEx sQTLs that populate the sQTL annotation) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Fitted fgwas effect sizes for the sQTL annotation must be positive with 95% confidence intervals excluding zero for asthma and the traits reported in Fig. 5b
- Estimates must match the published maximum likelihood values for the eight compiled GWAS traits
- Both sQTL and expression-only annotations should show effect sizes significantly different from zero in the fitted models
- fgwas_sQTL_effect_sizes.csv must contain per-trait effect-size estimates with CIs for all eight traits
- fgwas bed-format inputs must contain chr, start, end, snpId and association P-value columns sorted by coordinate

## Limitations

- Cannot distinguish whether paired sQTL/eQTL effects are caused by the same variant or two different variants in LD
- fgwas works best if there are at least ~20 independent loci with p-values less than 5e-8
- fgwas answers enrichment questions; it cannot identify novel causal variants beyond enrichments in the tested annotations

## Evidence

- [abstract] Genetic variants affecting splicing can have a stronger phenotypic impact than those affecting gene expression.: "Genetic variants affecting splicing can have a stronger phenotypic impact than those affecting gene expression."
- [readme] fgwas integrates functional genomic information into GWAS: [evidence span withheld — anchor status: external_doc]
- [readme] sQTLseekeR2 detects splicing QTLs: [evidence span withheld — anchor status: external_doc]
- [abstract] Many sQTLs also affect gene expression: "Many also affect the expression of the same or other genes, uncovering regulatory loci that act through different mechanisms."
- [readme] fgwas requires ~20 independent significant loci: [evidence span withheld — anchor status: external_doc]
