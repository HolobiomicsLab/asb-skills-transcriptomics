---
name: fgwas-genomic-annotation-modeling
description: Use when genome-wide GWAS summary statistics (P-values per SNP) are available
  together with a catalog of sQTLs and eQTLs, and the goal is to compare their GWAS
  enrichment per trait.
when_to_use_negative:
- The GWAS has few independent loci (fgwas works best with at least ~20 independent
  loci with p-values less than 5e-8)
- The question is fine-mapping or causal-variant resolution within a locus — the article
  notes that co-localization and fine-mapping methods are not applicable within the
  authors' multivariate framework, and paired sQTL/eQTL effects cannot be attributed
  to the same variant versus two variants in LD
- Input is single-variant association testing rather than annotation-set enrichment;
  use the sqtlseeker2-nf pipeline for sQTL discovery instead
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: fgwas
  role: Fits genomic annotation enrichment models by maximum likelihood to test GWAS
    association enrichment in the sQTL vs expression-only annotation tracks
  repo: joepickrell/fgwas
- name: sQTLseekeR2
  role: Provides the sQTL calls (via sqtlseeker2-nf on GTEx) that define the splicing-variant
    annotation track
  repo: https://github.com/guigolab/sQTLseekeR2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:8f3e066e447ed69b08de9cb222e4f4fbfaa9e8f2d01e296c6b37b6be0776e7fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joepickrell/fgwas
  tools:
  - fgwas
  - sQTLseekeR2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# fgwas-genomic-annotation-modeling

## Summary

Use the fgwas command line tool to test whether GWAS associations for a trait are enriched in sQTL versus expression-only annotation tracks, via maximum-likelihood genomic annotation models. This quantifies the phenotypic contribution of splicing variants relative to variants affecting gene expression only.

## When to use

Use when genome-wide GWAS summary statistics (P-values per SNP) are available together with a catalog of sQTLs and eQTLs, and the goal is to compare their GWAS enrichment per trait.

## When NOT to use

- The GWAS has few independent loci (fgwas works best with at least ~20 independent loci with p-values less than 5e-8)
- The question is fine-mapping or causal-variant resolution within a locus — the article notes that co-localization and fine-mapping methods are not applicable within the authors' multivariate framework, and paired sQTL/eQTL effects cannot be attributed to the same variant versus two variants in LD
- Input is single-variant association testing rather than annotation-set enrichment; use the sqtlseeker2-nf pipeline for sQTL discovery instead

## Inputs

- Genome-wide GWAS summary statistics for a trait (SNP coordinates and association P-values), as fgwas bed-format files with chr, start, end, snpId and P-value columns, sorted by coordinate
- sQTL catalog from sqtlseeker2-nf / sQTLseekeR2 mapping of GTEx data
- eQTL/expression association results used to classify variants affecting gene expression only

## Outputs

- Per-trait maximum-likelihood effect-size estimates with 95% confidence intervals for the sQTL annotation
- Per-trait fgwas annotation enrichment model fits
- fgwas_sQTL_effect_sizes.csv and a Fig. 5b-style forest plot comparing sQTL vs expression-only enrichment

## How to apply

Compile each trait's GWAS summary statistics (e.g. asthma, breast cancer, coronary artery disease, heart rate, height, LDL, rheumatoid arthritis, schizophrenia) into fgwas bed-format input with chr, start, end, snpId and association P-value, sorted by coordinate. Classify variants into two binary functional annotations derived from the GTEx sQTL mapping: sQTLs versus variants that affect gene expression only. Fit fgwas genomic annotation enrichment models per trait by maximum likelihood, contrasting the sQTL annotation against the expression-only annotation, then extract the maximum-likelihood effect-size estimate and 95% confidence interval for the sQTL annotation. Note the fgwas README caveat that the approach works best with at least ~20 independent loci with P-values less than 5e-8. Finally compare effect sizes across traits and export them (fgwas_sQTL_effect_sizes.csv) for a Fig. 5b-style forest plot.

## Related tools

- **fgwas** (Fits genomic annotation enrichment models by maximum likelihood to test GWAS association enrichment in the sQTL vs expression-only annotation tracks) — joepickrell/fgwas
- **sQTLseekeR2** (Provides the sQTL calls (via sqtlseeker2-nf on GTEx) that define the splicing-variant annotation track) — https://github.com/guigolab/sQTLseekeR2

## Evaluation signals

- Fitted sQTL annotation effect sizes must be positive with 95% confidence intervals excluding zero for asthma and the traits reported in Fig. 5b
- Estimates should reproduce the published maximum-likelihood values showing sQTLs have stronger GWAS association enrichment than expression-only variants for asthma, breast cancer, heart rate and height
- fgwas input files must have the chr, start, end, snpId and P-value columns, sorted by coordinate, with valid binary annotation tracks
- Both annotation tracks should show effect sizes significantly different from zero before interpreting the contrast

## Limitations

- fgwas works best if there are at least ~20 independent loci with p-values less than 5e-8 for the trait
- A substantial fraction of sQTLs also affects gene expression, so the sQTL vs expression-only tracks are not fully independent; paired effects cannot be resolved as same variant vs two variants in LD
- Heteropleiotropic (multi-mechanism) regulatory effects cannot be formally tested and results should be considered preliminary

## Evidence

- [readme] fgwas integrates functional annotations into GWAS: [evidence span withheld — anchor status: external_doc]
- [readme] fgwas power requirement: [evidence span withheld — anchor status: external_doc]
- [abstract] Splicing variants have stronger phenotypic impact: "Genetic variants affecting splicing can have a stronger phenotypic impact than those affecting gene expression."
- [discussion] Overlap between sQTLs and eQTLs complicates attribution: "we cannot distinguish the cases in which the two effects are indeed caused by the same variant or by two different variants in LD"
- [abstract] Many sQTLs also affect gene expression: "Many also affect the expression of the same or other genes, uncovering regulatory loci that act through different mechanisms."
