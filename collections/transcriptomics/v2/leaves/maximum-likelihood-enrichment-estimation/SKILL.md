---
name: maximum-likelihood-enrichment-estimation
description: Use when GWAS summary statistics for a trait are available together with
  binary functional annotations (e.g. sQTL vs expression-only variants) and you need
  to test whether associations are enriched in one annotation class via fgwas.
when_to_use_negative:
- Your GWAS has few independent loci — fgwas works best with at least ~20 independent
  loci with p-values less than 5e-8
- You lack a binary functional annotation distinguishing variant classes (e.g. no
  sQTL/eQTL classification), making the annotation-contrast model unidentifiable
- You need causal fine-mapping or co-localization rather than annotation enrichment
  — the article notes available co-localization/fine-mapping methods are not directly
  applicable within their multivariate framework, and enrichment estimates alone cannot
  distinguish whether paired sQTL/eQTL effects come from the same variant or two variants
  in LD
edam_operation: http://edamontology.org/operation_3715
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: fgwas
  role: Command-line tool used to fit genomic annotation enrichment models by maximum
    likelihood on GWAS summary statistics with binary sQTL/expression-only annotations
  repo: https://github.com/joepickrell/fgwas
- name: sQTLseekeR2
  role: R package used (via sqtlseeker2-nf) to map sQTLs that define the functional
    annotation track contrasted against expression-only variants
  repo: https://github.com/guigolab/sQTLseekeR2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:6df85735e003411e41a5cc1ce29b2522af9eda9e340f8ab67b4441519ea99dc0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joepickrell/fgwas
  tools:
  - fgwas
  - sQTLseekeR2
  edam_operation: http://edamontology.org/operation_3715
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# maximum-likelihood-enrichment-estimation

## Summary

Fit fgwas genomic annotation enrichment models by maximum likelihood to estimate whether GWAS associations are enriched in functional annotations (here, sQTL versus expression-only variants), extracting effect-size estimates and 95% CIs per trait.

## When to use

Use when GWAS summary statistics for a trait are available together with binary functional annotations (e.g. sQTL vs expression-only variants) and you need to test whether associations are enriched in one annotation class via fgwas.

## When NOT to use

- Your GWAS has few independent loci — fgwas works best with at least ~20 independent loci with p-values less than 5e-8
- You lack a binary functional annotation distinguishing variant classes (e.g. no sQTL/eQTL classification), making the annotation-contrast model unidentifiable
- You need causal fine-mapping or co-localization rather than annotation enrichment — the article notes available co-localization/fine-mapping methods are not directly applicable within their multivariate framework, and enrichment estimates alone cannot distinguish whether paired sQTL/eQTL effects come from the same variant or two variants in LD

## Inputs

- Genome-wide GWAS summary statistics (association P-values) for traits such as asthma, breast cancer, coronary artery disease, heart rate, height, LDL, rheumatoid arthritis, schizophrenia
- sQTL mapping results (e.g. from sQTLseekeR2 on GTEx data) used to label variants as sQTLs
- Expression-only variant assignments (variants affecting gene expression but not splicing)
- fgwas bed-format input files with chr, start, end, snpId and P-value columns, sorted by coordinate

## Outputs

- Per-trait maximum likelihood effect-size estimates and 95% CIs for the sQTL annotation (fgwas_sQTL_effect_sizes.csv)
- Forest plot of effect sizes across traits (Fig. 5b-style)

## How to apply

Compile genome-wide GWAS summary statistics for each trait into fgwas bed-format input files with chr, start, end, snpId and association P-value columns, sorted by coordinate. Classify variants into two binary annotation tracks derived from sQTL mapping results (sQTLs vs expression-only variants). Fit the fgwas genomic annotation enrichment model per trait by maximum likelihood, contrasting the sQTL annotation against the expression-only annotation. Extract the maximum likelihood effect-size estimates and their 95% confidence intervals per trait, save them to fgwas_sQTL_effect_sizes.csv, and render a Fig. 5b-style forest plot. Rationale: fgwas's joint model (Pickrell 2014) integrates functional genomic data with GWAS association P-values to quantify enrichment of associations in annotation classes.

## Related tools

- **fgwas** (Command-line tool used to fit genomic annotation enrichment models by maximum likelihood on GWAS summary statistics with binary sQTL/expression-only annotations) — https://github.com/joepickrell/fgwas
- **sQTLseekeR2** (R package used (via sqtlseeker2-nf) to map sQTLs that define the functional annotation track contrasted against expression-only variants) — https://github.com/guigolab/sQTLseekeR2

## Evaluation signals

- Fitted effect sizes for the sQTL annotation must be positive with 95% CIs excluding zero for asthma and the traits reported in Fig. 5b
- Estimates must match the published maximum likelihood values from the article for the eight compiled traits
- fgwas input bed files must contain chr, start, end, snpId and P-value columns and be sorted by coordinate
- Both sQTL and expression-only annotations should show effect sizes significantly different from zero across the compiled traits
- sQTL effect sizes should exceed expression-only effect sizes for asthma, breast cancer, heart rate and height

## Limitations

- fgwas works best if there are at least ~20 independent loci with p-values less than 5e-8 for the trait being analyzed
- Enrichment results cannot distinguish whether paired sQTL/eQTL effects are caused by the same variant or two different variants in LD
- Enrichment is a statistical association with annotations, not a causal fine-mapping result; co-localization methods are not directly applicable within the authors' multivariate framework

## Evidence

- [readme] fgwas tool definition: [evidence span withheld — anchor status: external_doc]
- [readme] fgwas loci requirement: [evidence span withheld — anchor status: external_doc]
- [abstract] phenotypic impact finding: "Genetic variants affecting splicing can have a stronger phenotypic impact than those affecting gene expression."
- [intro] sQTL pipeline: "we have developed a pipeline for efficient and reproducible sQTL mapping."
- [readme] enrichment question: [evidence span withheld — anchor status: external_doc]
