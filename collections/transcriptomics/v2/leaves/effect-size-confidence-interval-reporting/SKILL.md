---
name: effect-size-confidence-interval-reporting
description: Use when per-trait fgwas annotation model fits have been obtained and
  you must report the annotation effect sizes (with 95% CIs) to test whether sQTLs
  show stronger GWAS association than expression-only variants.
when_to_use_negative:
- fgwas fits have not converged on the trait summary statistics — there is no effect-size
  estimate or CI to report yet
- The analysis targets identification of novel loci or colocalization rather than
  quantifying annotation enrichment effects
- Only one annotation category exists (e.g. no expression-only contrast), making effect-size
  comparison against an expression-only baseline undefined
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: fgwas
  role: Fits genomic annotation enrichment models by maximum likelihood and yields
    the effect-size estimates and 95% confidence intervals reported per trait
  repo: https://github.com/joepickrell/fgwas
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:c41102512ebed6f94fef02889423ce8aef3f656b7758628fbb00a675c3f58512
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joepickrell/fgwas
  tools:
  - fgwas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# effect-size-confidence-interval-reporting

## Summary

Extract and report maximum-likelihood effect-size estimates with 95% confidence intervals for genomic annotations (sQTL vs expression-only variants) fitted with fgwas per GWAS trait, enabling cross-trait comparison of annotation enrichment.

## When to use

Use when per-trait fgwas annotation model fits have been obtained and you must report the annotation effect sizes (with 95% CIs) to test whether sQTLs show stronger GWAS association than expression-only variants.

## When NOT to use

- fgwas fits have not converged on the trait summary statistics — there is no effect-size estimate or CI to report yet
- The analysis targets identification of novel loci or colocalization rather than quantifying annotation enrichment effects
- Only one annotation category exists (e.g. no expression-only contrast), making effect-size comparison against an expression-only baseline undefined

## Inputs

- Fitted fgwas genomic annotation model files per trait (maximum likelihood fit contrasting sQTL annotation vs expression-only annotation)
- GWAS summary statistics in fgwas bed-format (chr, start, end, snpId, association P-value), sorted by coordinate
- Binary functional annotation tracks classifying variants as sQTLs versus expression-only variants from sQTL mapping results

## Outputs

- fgwas_sQTL_effect_sizes.csv with per-trait maximum likelihood effect-size estimates and 95% confidence intervals
- Fig. 5b-style forest plot of sQTL annotation effect sizes with 95% CIs across traits

## How to apply

After fitting fgwas genomic annotation enrichment models by maximum likelihood for each of the eight GWAS traits (asthma, breast cancer, coronary artery disease, heart rate, height, LDL, rheumatoid arthritis, schizophrenia), extract the maximum likelihood effect-size estimate for the sQTL annotation and its 95% confidence interval from the fitted model. Compile the per-trait estimates and CIs into a table (fgwas_sQTL_effect_sizes.csv). Judge correctness by requiring fitted effect sizes to be positive with confidence intervals excluding zero for asthma and the traits reported in Fig. 5b, and by comparing estimates across traits to confirm sQTLs show stronger GWAS association enrichment than expression-only variants. Render the results as a forest plot (Fig. 5b style) so CI overlap with zero is visually explicit.

## Related tools

- **fgwas** (Fits genomic annotation enrichment models by maximum likelihood and yields the effect-size estimates and 95% confidence intervals reported per trait) — https://github.com/joepickrell/fgwas

## Evaluation signals

- Each fitted effect size for the sQTL annotation is positive in asthma, breast cancer, heart rate and height, matching the published maximum likelihood estimates
- The 95% confidence intervals for these effect sizes exclude zero (effect sizes significantly different from zero)
- fgwas_sQTL_effect_sizes.csv contains one estimate-and-CI pair for each of the eight compiled traits
- Forest plot ordering reproduces the Fig. 5b pattern with sQTLs showing stronger association enrichment than expression-only variants for the reported traits

## Limitations

- fgwas works best when there are at least ~20 independent loci with p-values less than 5e-8; traits with fewer loci may yield unstable effect-size estimates
- The analysis cannot distinguish whether paired sQTL/eQTL effects are caused by the same variant or two different variants in LD, which confounds annotation attribution
- Effect-size estimates are contrast-dependent: they quantify enrichment of the sQTL annotation relative to expression-only variants and should not be read as per-variant effect sizes

## Evidence

- [other] sQTLs and expression-only variants show effect sizes significantly different from zero: [evidence span withheld — anchor status: not_found]
- [readme] fgwas is a command line tool for integrating functional genomic information into GWAS: [evidence span withheld — anchor status: external_doc]
- [abstract] splicing variants phenotypic impact: "Genetic variants affecting splicing can have a stronger phenotypic impact than those affecting gene expression."
