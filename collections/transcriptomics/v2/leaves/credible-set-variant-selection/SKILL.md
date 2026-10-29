---
name: credible-set-variant-selection
description: Use when you have CAFEH fine-mapping output tables for GTEx gene-tissue
  pairs and need per-tissue credible-set test variants plus a matched lead-eQTL background
  for enrichment tests against Roadmap regulatory annotations.
when_to_use_negative:
- Input loci are mediated by coding variants rather than eQTL effects — the paper
  notes such loci 'may not demonstrate eQTL effects and therefore cannot be explored
  with CAFEH'.
- LD matrices differ substantially between studies (e.g., ancestry differences) —
  CAFEH relies on a single LD matrix and such differences could impact inference.
- The goal is single-signal colocalization with no expectation of allelic heterogeneity;
  CAFEH's multi-component credible sets are designed for the presence of multiple
  causal alleles.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: CAFEH
  role: Produces the fine-mapping results tables from which credible-set variants
    are selected; fit with an arbitrarily high number of causal components (20 per
    locus in this study).
  repo: github.com/karltayeb/cafeh
- name: Python
  role: Used to parse cafeh.{mode}.results tables, intersect variant sets with chromHMM
    annotations, and run scipy.stats.fisher_exact per tissue.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:5365115a37491ca6f2163aab0d8521a0f5726935b45ae1ca4a8bc7b7d23bcd03
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - Python
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# credible-set-variant-selection

## Summary

Selects test variants for regulatory-enrichment analysis by parsing CAFEH fine-mapping results (cafeh.{mode}.results) to extract the maximum-pip SNP in the credible set of each locus's top_component, and builds a matched background of minimum-p-value lead eQTL variants per gene.

## When to use

Use when you have CAFEH fine-mapping output tables for GTEx gene-tissue pairs and need per-tissue credible-set test variants plus a matched lead-eQTL background for enrichment tests against Roadmap regulatory annotations.

## When NOT to use

- Input loci are mediated by coding variants rather than eQTL effects — the paper notes such loci 'may not demonstrate eQTL effects and therefore cannot be explored with CAFEH'.
- LD matrices differ substantially between studies (e.g., ancestry differences) — CAFEH relies on a single LD matrix and such differences could impact inference.
- The goal is single-signal colocalization with no expectation of allelic heterogeneity; CAFEH's multi-component credible sets are designed for the presence of multiple causal alleles.

## Inputs

- CAFEH results tables (cafeh.{mode}.results) with per-(SNP, study) pip, alpha, rank, top_component, p_active, pi fields
- GTEx v7 eQTL summary statistics (lead eQTL variants and p-values per protein-coding gene-tissue pair)
- Roadmap Epigenomics chromHMM chromatin-state annotations (active promoters/enhancers)

## Outputs

- Per-tissue credible-set test variant sets and matched lead-eQTL background sets
- tissue_regulatory_enrichment.tsv containing per-tissue odds ratios and Fisher's exact p-values

## How to apply

Parse the CAFEH results table (cafeh.{mode}.results), which reports one row per (SNP, study) pair with fields pip, top_component, alpha, rank, p_active, pi, effect, and effect_var. For each tissue, select the test variant as the maximum-weight (highest pip) SNP whose alpha value indicates membership in the credible set of its top_component; alpha gives the smallest credible-set level at which the SNP is included (e.g., alpha = 0.3 means inclusion in every credible set at 30% or greater). Construct the matched background by taking, per gene, the minimum-p-value lead eQTL variant, balancing its size against the tissue-specific test set. Intersect both variant sets with Roadmap Epigenomics chromHMM active promoter/enhancer states and run a 2x2 Fisher's exact test (scipy.stats.fisher_exact) per tissue to obtain odds ratios and p-values, compiled into tissue_regulatory_enrichment.tsv. The rationale is that CAFEH's component-aware credible sets, unlike top-eQTL picks, account for allelic heterogeneity across tissues.

## Related tools

- **CAFEH** (Produces the fine-mapping results tables from which credible-set variants are selected; fit with an arbitrarily high number of causal components (20 per locus in this study).) — github.com/karltayeb/cafeh
- **Python** (Used to parse cafeh.{mode}.results tables, intersect variant sets with chromHMM annotations, and run scipy.stats.fisher_exact per tissue.)

## Examples

```
python cafeh.py --mode beta -R example/LD_example.tsv -B example/beta_example.tsv -S example/stderr_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- Every parsed row retains pip, alpha, rank, top_component, and pi fields as defined in the CAFEH results-table schema.
- Each selected test variant satisfies the credible-set condition: its reported alpha equals the smallest credible-set level at which it is included in top_component.
- The output tissue_regulatory_enrichment.tsv contains an odds ratio and Fisher's exact p-value for each GTEx tissue.
- Tissue-specific credible-set variants show greater enrichment of active Roadmap regulatory elements than the matched lead-eQTL background, reproducing the paper's Figure 3D pattern.
- Background set size is balanced against the tissue-specific test set size.

## Limitations

- CAFEH includes a simple prior on variant causality, assuming equal prior probability across all evaluated variants at a locus (no informed priors).
- CAFEH does not handle missing data in the effect size matrix; it implicitly assumes causal variants are present in the tested datasets.
- It is difficult to distinguish the presence of multiple signals among variants in nearly perfect or very high LD with each other.
- The current version of CAFEH relies on a single LD matrix; large LD differences between studies (e.g., ancestry differences) could impact colocalization and fine-mapping inference.
- Colocalization/credible-set membership does not necessarily suggest mediation; horizontal pleiotropic effects can produce colocalizing signals without a linking causal pathway.

## Evidence

- [readme] alpha as credible-set membership criterion: "`alpha` the smallest credible set level at which the SNP would be included in the credible set for `top_component`."
- [readme] results table schema: "It contains a row for each (SNP, study) pair. It reports - `pip` The posterior inclusion probability for that SNP in that study"
- [discussion] number of causal components: [evidence span withheld — anchor status: not_found]
- [methods] running CAFEH to produce results tables: "To run CAFEH with effect sizes and standard errors, include `--mode beta` in the arguments. You must provide input `--ld (-R)`, `--betas (-B)`, `--standard-errors (-S)`"
