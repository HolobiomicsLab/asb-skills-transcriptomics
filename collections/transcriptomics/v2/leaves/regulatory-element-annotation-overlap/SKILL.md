---
name: regulatory-element-annotation-overlap
description: Use when you have fine-mapped credible-set variants (e.g., CAFEH pip/alpha
  per SNP) and want to test whether they are enriched in active promoters/enhancers
  relative to a matched background, as in Figure 3D.
when_to_use_negative:
- Loci whose effects are mediated via coding variants may not show eQTL effects and
  cannot be assessed with this eQTL-based regulatory overlap approach
- When studies have large LD differences (e.g., ancestry differences), fine-mapped
  credible sets may be unreliable before annotation overlap is meaningful
- Colocalization does not necessarily suggest mediation — do not interpret regulatory
  enrichment alone as proof of a causal variant-to-gene pathway (horizontal pleiotropy
  is possible)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Produces the credible-set fine-mapping output (pip, alpha, top_component,
    pi) whose variants are intersected with regulatory annotations
  repo: github.com/karltayeb/cafeh
- name: Python
  role: Used to parse results tables, intersect variants with chromHMM annotations,
    and run scipy.stats.fisher_exact per tissue
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:d4924ff8b4508dda03d017886116381ff006292fd62f2f07f5a46a8d14884634
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - Python
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# regulatory-element-annotation-overlap

## Summary

Assess whether fine-mapped eQTL causal variants are functionally plausible by intersecting CAFEH credible-set variants with Roadmap Epigenomics chromHMM active regulatory elements and testing enrichment against a matched lead-eQTL background using Fisher's exact test per tissue.

## When to use

Use when you have fine-mapped credible-set variants (e.g., CAFEH pip/alpha per SNP) and want to test whether they are enriched in active promoters/enhancers relative to a matched background, as in Figure 3D.

## When NOT to use

- Loci whose effects are mediated via coding variants may not show eQTL effects and cannot be assessed with this eQTL-based regulatory overlap approach
- When studies have large LD differences (e.g., ancestry differences), fine-mapped credible sets may be unreliable before annotation overlap is meaningful
- Colocalization does not necessarily suggest mediation — do not interpret regulatory enrichment alone as proof of a causal variant-to-gene pathway (horizontal pleiotropy is possible)

## Inputs

- CAFEH results tables (cafeh.genotype.results / cafeh.beta.results / cafeh.z.results) with pip, alpha, rank, top_component, pi per (SNP, study) pair
- GTEx v7 eQTL summary statistics for protein-coding gene-tissue pairs (to pick minimum-p-value lead eQTL variants)
- Roadmap Epigenomics chromHMM chromatin state annotations per tissue (active promoters/enhancers)

## Outputs

- tissue_regulatory_enrichment.tsv with one odds ratio and Fisher's exact p-value per GTEx tissue
- Per-tissue test (credible-set) and background (lead-eQTL) variant sets flagged for active regulatory element overlap

## How to apply

Parse the CAFEH results tables (cafeh.{mode}.results) for each GTEx protein-coding gene-tissue pair, extracting per-(SNP, study) pip, alpha, rank, top_component, and pi fields. For each tissue, select the test variant as the maximum-pip SNP included in the credible set of its top_component at the alpha credible-set threshold; construct the background from each gene's minimum-p-value lead eQTL variant, balancing background size against the tissue-specific test set. Intersect both variant sets with Roadmap chromHMM chromatin state annotations to flag variants falling in active promoters/enhancers, then run scipy.stats.fisher_exact on the 2x2 table (variant set membership x active regulatory overlap) per tissue. Correct application yields per-tissue odds ratios and p-values showing credible-set variants are more enriched in active regulatory elements than the matched background.

## Related tools

- **CAFEH** (Produces the credible-set fine-mapping output (pip, alpha, top_component, pi) whose variants are intersected with regulatory annotations) — github.com/karltayeb/cafeh
- **Python** (Used to parse results tables, intersect variants with chromHMM annotations, and run scipy.stats.fisher_exact per tissue)

## Examples

```
from scipy.stats import fisher_exact; odds, p = fisher_exact([[n_test_active, n_test_inactive], [n_bg_active, n_bg_inactive]])
```

## Evaluation signals

- Output table must contain an odds ratio and Fisher's exact p-value for each GTEx tissue
- Tissue-specific credible-set variants should show greater enrichment of active regulatory elements than the matched lead-eQTL background, matching the paper's Figure 3D pattern
- Expected direction: enrichment is for promoter elements in the corresponding tissues as defined by Roadmap chromHMM
- Background size must be balanced against the tissue-specific test set before running the contingency test

## Limitations

- CAFEH relies on a single LD matrix; large LD differences between studies (e.g., ancestry differences) could impact fine-mapping and thus the credibility of the tested variant set
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD, which can blur which SNP enters the credible set
- Enrichment in active regulatory elements does not establish mediation; colocalizing/enriched signals can arise from horizontal pleiotropic effects

## Evidence

- [results] Causal variants fine-mapped by CAFEH in each tissue are enriched for promoter elements in the corresponding tissues, as defined by Roadmap chromHMM, compared to top eQTL variants.: [evidence span withheld — anchor status: not_found]
- [readme] alpha credible set definition: "`alpha` the smallest credible set level at which the SNP would be included in the credible set for `top_component`."
- [readme] results table contents: "The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair."
- [discussion] LD limitation: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
- [discussion] coding variant limitation: "loci whose effects are mediated via coding variants may not demonstrate eQTL effects and therefore cannot be explored with CAFEH."
