---
name: tissue-specificity-enrichment-analysis
description: Use when you have CAFEH results tables (cafeh.{mode}.results) with per-(SNP,
  study) pip/alpha/rank/pi fields and want to assess functional validity of fine-mapped
  variants against Roadmap chromatin states.
when_to_use_negative:
- Loci whose effects are mediated via coding variants may not show eQTL effects and
  cannot be explored with CAFEH, so this regulatory-enrichment validation is inappropriate
  for such loci
- When LD differs substantially between studies (e.g., ancestry differences), since
  CAFEH relies on a single LD matrix and colocalization/fine-mapping inference may
  be unreliable before enrichment testing
- 'When the goal is mediation inference: colocalization/enrichment of credible-set
  variants in regulatory elements does not establish a causal pathway due to possible
  horizontal pleiotropy'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: CAFEH
  role: Produces the fine-mapping results tables (credible sets, pip, alpha, top_component)
    whose variants are tested for regulatory enrichment
  repo: github.com/karltayeb/cafeh
- name: Python
  role: Used for parsing results tables, intersecting variants with Roadmap annotations,
    and running scipy.stats.fisher_exact per tissue
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:f644b2532a8b3f4ea4a2a45316ca9579ae4af39b1e76f01d95d0b76029430322
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - Python
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# tissue-specificity-enrichment-analysis

## Summary

Validates CAFEH fine-mapping on real data by testing whether tissue-specific credible-set variants are enriched for active regulatory elements (Roadmap chromHMM promoters/enhancers) relative to a matched lead-eQTL background, per GTEx tissue, using Fisher's exact tests.

## When to use

Use when you have CAFEH results tables (cafeh.{mode}.results) with per-(SNP, study) pip/alpha/rank/pi fields and want to assess functional validity of fine-mapped variants against Roadmap chromatin states.

## When NOT to use

- Loci whose effects are mediated via coding variants may not show eQTL effects and cannot be explored with CAFEH, so this regulatory-enrichment validation is inappropriate for such loci
- When LD differs substantially between studies (e.g., ancestry differences), since CAFEH relies on a single LD matrix and colocalization/fine-mapping inference may be unreliable before enrichment testing
- When the goal is mediation inference: colocalization/enrichment of credible-set variants in regulatory elements does not establish a causal pathway due to possible horizontal pleiotropy

## Inputs

- CAFEH results tables (cafeh.genotype.results / cafeh.beta.results / cafeh.z.results) with pip, top_component, alpha, rank, p_active, pi, effect fields per (SNP, study) pair
- GTEx v7 eQTL summary statistics for protein-coding gene-tissue pairs (to select minimum-p-value lead eQTL variants)
- Roadmap Epigenomics chromHMM chromatin state annotations (active promoters/enhancers) per tissue

## Outputs

- tissue_regulatory_enrichment.tsv with per-tissue odds ratios and Fisher's exact p-values
- Per-tissue test variant sets (max-pip credible-set SNPs) and matched lead-eQTL background variant sets with active-regulatory overlap flags

## How to apply

Parse the cafeh.{mode}.results tables for every GTEx protein-coding gene-tissue pair, extracting pip, alpha, rank, top_component, and pi per (SNP, study). For each tissue, define the test variant as the maximum-pip SNP included in the credible set of its top_component at the alpha credible-set threshold reported in the results table (alpha is the smallest credible-set level at which the SNP is included). Build a matched background per gene from the minimum-p-value lead eQTL variant, balancing background size against the tissue-specific test set. Intersect both variant sets with Roadmap Epigenomics chromatin state annotations to flag overlap with active promoters/enhancers, then run scipy.stats.fisher_exact on the 2x2 table (variant set membership x active regulatory overlap) per tissue. Tissue-specific credible-set variants should show greater enrichment than the matched lead-eQTL background, consistent with the paper's Figure 3D pattern.

## Related tools

- **CAFEH** (Produces the fine-mapping results tables (credible sets, pip, alpha, top_component) whose variants are tested for regulatory enrichment) — github.com/karltayeb/cafeh
- **Python** (Used for parsing results tables, intersecting variants with Roadmap annotations, and running scipy.stats.fisher_exact per tissue)

## Examples

```
python cafeh.py --mode beta -R example/LD_example.tsv -B example/beta_example.tsv -S example/stderr_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- tissue_regulatory_enrichment.tsv contains an odds ratio and Fisher's exact p-value for every GTEx tissue
- Tissue-specific credible-set variants show greater enrichment of active regulatory elements than the matched lead-eQTL background (Figure 3D pattern)
- Causal variants fine-mapped by CAFEH in each tissue are enriched for promoter elements in the corresponding tissues as defined by Roadmap chromHMM
- Test variants are the max-pip SNPs meeting the alpha credible-set threshold from top_component; background variants are per-gene minimum-p-value lead eQTLs with balanced set sizes

## Limitations

- CAFEH relies on a single LD matrix; large LD differences between studies could impact fine-mapping and thus the credible sets used for enrichment
- CAFEH includes a simple prior on variant causality (equal prior probability across all evaluated variants) with no informed priors
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD, which can blur credible-set membership
- Colocalization/enrichment does not necessarily suggest mediation; signals may arise without a linking causal pathway due to horizontal pleiotropy
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in the tested datasets

## Evidence

- [results] Causal variants fine-mapped by CAFEH in each tissue are enriched for promoter elements in the corresponding tissues, as defined by Roadmap chromHMM, compared to top eQTL variants.: [evidence span withheld — anchor status: not_found]
- [readme] alpha credible set definition: "`alpha` the smallest credible set level at which the SNP would be included in the credible set for `top_component`"
- [readme] results table content: "It contains a row for each (SNP, study) pair. It reports - `pip` The posterior inclusion probability for that SNP in that study"
- [full_text] Run Fisher's exact test per tissue: [evidence span withheld — anchor status: not_found]
- [discussion] LD limitation: "the current version of CAFEH relies on a single LD matrix, therefore large differences in LD between studies, due to ancestry differences or other factors, could impact inference"
