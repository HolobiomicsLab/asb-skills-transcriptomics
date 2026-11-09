---
name: eqtl-lead-variant-background-matching
description: Use when evaluating whether CAFEH fine-mapped credible-set variants are
  enriched in active regulatory elements (Roadmap chromHMM) and a size-balanced, gene-matched
  lead-eQTL background is needed for comparison.
when_to_use_negative:
- When LD matrices differ greatly between studies (e.g., ancestry differences) — CAFEH
  relies on a single LD matrix, so colocalization/fine-mapping inference feeding the
  credible sets may be biased
- When loci are likely mediated by coding variants, which may not demonstrate eQTL
  effects and therefore cannot be explored with CAFEH
- When the effect-size matrix has missing data — CAFEH does not handle missing data
  and implicitly assumes causal variants are present in all tested datasets
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Produces the per-(SNP, study) results table (pip, alpha, rank, top_component,
    pi) from which credible-set test variants are extracted; run as a command line
    tool or Python package
  repo: https://github.com/karltayeb/cafeh
- name: Python
  role: Parses cafeh.{mode}.results tables, constructs the matched background, intersects
    variants with Roadmap chromHMM states, and runs scipy.stats.fisher_exact per tissue
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:f81fcdfe742d7ac30ffe0cef2ef5e947e7f45d69169c52cd9853f6d5478e1713
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karltayeb/cafeh
  tools:
  - CAFEH
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# eqtl-lead-variant-background-matching

## Summary

Constructs a matched background set of minimum-p-value lead eQTL variants so that CAFEH credible-set variants can be tested (via Fisher's exact test) for enrichment in Roadmap active regulatory elements, distinguishing causal fine-mapping signal from ordinary top-eQTL association.

## When to use

Use when evaluating whether CAFEH fine-mapped credible-set variants are enriched in active regulatory elements (Roadmap chromHMM) and a size-balanced, gene-matched lead-eQTL background is needed for comparison.

## When NOT to use

- When LD matrices differ greatly between studies (e.g., ancestry differences) — CAFEH relies on a single LD matrix, so colocalization/fine-mapping inference feeding the credible sets may be biased
- When loci are likely mediated by coding variants, which may not demonstrate eQTL effects and therefore cannot be explored with CAFEH
- When the effect-size matrix has missing data — CAFEH does not handle missing data and implicitly assumes causal variants are present in all tested datasets

## Inputs

- CAFEH results tables (cafeh.genotype.results, cafeh.beta.results, or cafeh.z.results) with per-(SNP, study) pip, alpha, rank, top_component, p_active, pi fields
- GTEx v7 cis-eQTL summary statistics for protein-coding gene-tissue pairs (p-values, lead variants)
- Roadmap Epigenomics chromHMM chromatin state annotations (active promoters/enhancers)

## Outputs

- tissue_regulatory_enrichment.tsv with per-tissue odds ratios and Fisher's exact p-values
- Per-tissue test set (CAFEH credible-set variants) and matched lead-eQTL background set with active-regulatory overlap flags

## How to apply

Parse the CAFEH results tables (cafeh.{mode}.results), which contain one row per (SNP, study) pair with pip, alpha, rank, top_component, and pi fields. For each GTEx tissue, select the test variant as the highest-pip SNP included in the credible set of its top_component, using the alpha credible-set threshold reported in the results table. Build the matched background by taking, for each gene, the minimum-p-value lead eQTL variant, balancing background size against the tissue-specific test set. Intersect both sets with Roadmap chromHMM annotations to flag variants overlapping active promoters/enhancers, then run scipy.stats.fisher_exact on the 2x2 table (variant set membership x active regulatory overlap) per tissue. The rationale is that CAFEH's pip-based credible sets, which account for allelic heterogeneity, should show greater regulatory-element enrichment than raw top-eQTL variants if fine-mapping recovers causal regulatory variants.

## Related tools

- **CAFEH** (Produces the per-(SNP, study) results table (pip, alpha, rank, top_component, pi) from which credible-set test variants are extracted; run as a command line tool or Python package) — https://github.com/karltayeb/cafeh
- **Python** (Parses cafeh.{mode}.results tables, constructs the matched background, intersects variants with Roadmap chromHMM states, and runs scipy.stats.fisher_exact per tissue)

## Examples

```
from scipy.stats import fisher_exact; odds, p = fisher_exact([[a, b], [c, d]])  # credible-set variants vs matched lead-eQTL background, overlapped with Roadmap active states
```

## Evaluation signals

- tissue_regulatory_enrichment.tsv contains an odds ratio and Fisher's exact p-value for each GTEx tissue
- Tissue-specific CAFEH credible-set variants show greater enrichment of active regulatory elements than the matched minimum-p-value lead-eQTL background, consistent with the paper's Figure 3D pattern
- Test variant selection uses the alpha credible-set threshold from the results table (alpha = the smallest credible set level at which the SNP is included for top_component)
- Background size is balanced against the tissue-specific test set size
- Credible-set membership can be traced to CAFEH fields: pip (posterior inclusion probability), pi (posterior probability of being causal for top_component), and rank

## Limitations

- CAFEH relies on a single LD matrix; large LD differences between studies (e.g., ancestry differences) could impact colocalization and fine-mapping inference
- It is difficult to distinguish the presence of multiple signals among variants in nearly perfect or very high LD with each other
- Colocalization does not necessarily suggest mediation; colocalizing signals may arise without a linking causal pathway due to horizontal pleiotropic effects
- CAFEH includes a simple prior on variant causality, assuming equal prior probability across all evaluated variants at a locus (no informed priors)

## Evidence

- [readme] results table schema: "The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair."
- [readme] alpha credible-set definition: "`alpha` the smallest credible set level at which the SNP would be included in the credible set for `top_component`."
- [discussion] number of causal components: [evidence span withheld — anchor status: not_found]
- [discussion] colocalization vs mediation caveat: "colocalization does not necessarily suggest mediation and it remains possible that colocalizing signals can be observed without a linking causal pathway due to horizontal pleiotropic effects."
