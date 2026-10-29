---
name: omim-disease-gene-fisher-enrichment
description: Use when you have gene sets stratified by a regulatory metric (e.g.,
  CAFEH top-component sharing across tissues) and want to test overrepresentation
  of curated disease genes from OMIM, separately for dominant and recessive inheritance
  modes.
when_to_use_negative:
- When the regulatory stratification itself is unreliable — CAFEH relies on a single
  LD matrix, so large LD differences between studies (e.g., ancestry differences)
  could invalidate the gene set definitions feeding the enrichment test
- 'When the question concerns coding-variant-mediated loci: loci whose effects are
  mediated via coding variants may not show eQTL effects and cannot be explored with
  CAFEH, so they would be absent from the gene sets'
- When colocalization is being interpreted as mediation — colocalizing signals may
  arise without a linking causal pathway due to horizontal pleiotropic effects, so
  disease-gene enrichment does not prove causal mediation
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3517
tools:
- name: CAFEH
  role: Produces the colocalization results table (pip, top_component, alpha, rank,
    p_active) from which highly vs poorly colocalizing gene sets are defined before
    OMIM Fisher enrichment
  repo: github.com/karltayeb/cafeh
- name: Python
  role: Runs the Fisher exact (hypergeometric-style) overrepresentation tests on the
    colocalization-defined gene sets and writes the statistics to a results table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:5ad34311bcfbb97f0ed6e944090463a336257ee93fdf2cd047ff2b155ee8bacb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - Python
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# omim-disease-gene-fisher-enrichment

## Summary

Tests whether gene sets defined by CAFEH eQTL colocalization sharing (highly vs poorly colocalizing vs background) are overrepresented for OMIM dominant and recessive disease genes using hypergeometric-style Fisher exact tests. This connects tissue specificity of genetic regulation to human disease relevance.

## When to use

Use when you have gene sets stratified by a regulatory metric (e.g., CAFEH top-component sharing across tissues) and want to test overrepresentation of curated disease genes from OMIM, separately for dominant and recessive inheritance modes.

## When NOT to use

- When the regulatory stratification itself is unreliable — CAFEH relies on a single LD matrix, so large LD differences between studies (e.g., ancestry differences) could invalidate the gene set definitions feeding the enrichment test
- When the question concerns coding-variant-mediated loci: loci whose effects are mediated via coding variants may not show eQTL effects and cannot be explored with CAFEH, so they would be absent from the gene sets
- When colocalization is being interpreted as mediation — colocalizing signals may arise without a linking causal pathway due to horizontal pleiotropic effects, so disease-gene enrichment does not prove causal mediation

## Inputs

- CAFEH results table (cafeh.{mode}.results) with per-SNP-study columns pip, top_component, alpha, rank, p_active
- Gene set assignments: background (significant cis-eQTLs in >=20 tissues), highly colocalizing (top-component sharing >=20 tissues), poorly colocalizing (top-component sharing <10 tissues)
- OMIM disease gene annotations stratified by inheritance mode (dominant and recessive)

## Outputs

- Per gene set, per inheritance mode: Fisher exact test odds ratios and p-values for OMIM dominant and recessive disease gene enrichment
- Results table combining gene set definitions with all test statistics (Wilcoxon p-values, Fisher odds ratios and p-values, Bonferroni-adjusted GO terms)

## How to apply

First define the gene sets from the CAFEH results table (cafeh.{mode}.results, using pip, top_component, alpha, rank, p_active columns per SNP-study pair): highly colocalizing genes whose top component is shared in >=20 tissues, poorly colocalizing genes with top-component sharing in <10 tissues, and background genes with significant cis-eQTLs in >=20 tissues. For each gene set and each OMIM inheritance mode (dominant, recessive), construct a 2x2 contingency table (in-set vs out-of-set by OMIM disease gene vs non-disease gene) and run a Fisher exact test as a hypergeometric-style overrepresentation test, reporting the odds ratio and p-value. Interpret alongside the companion Wilcoxon constraint tests (LOEUF, pLI) and GO term Fisher enrichment with Bonferroni correction; the article's expectation is that tissue-specific genes are 'more likely to be involved in human disease'.

## Related tools

- **CAFEH** (Produces the colocalization results table (pip, top_component, alpha, rank, p_active) from which highly vs poorly colocalizing gene sets are defined before OMIM Fisher enrichment) — github.com/karltayeb/cafeh
- **Python** (Runs the Fisher exact (hypergeometric-style) overrepresentation tests on the colocalization-defined gene sets and writes the statistics to a results table)

## Examples

```
from scipy.stats import fisher_exact; odds, p = fisher_exact([[n_set_disease, n_set_nodisease], [n_out_disease, n_out_nodisease]])  # OMIM dominant enrichment per gene set
```

## Evaluation signals

- Each gene set definition matches the stated thresholds: highly colocalizing = top-component shared in >=20 tissues; poorly colocalizing = sharing <10 tissues; background = significant cis-eQTLs in >=20 tissues
- Fisher tests are run separately for OMIM dominant and recessive modes, each yielding both an odds ratio and a p-value in the results table
- Contingency tables are constructed hypergeometric-style over the full tested gene universe (in-set vs out-of-set), not just within the gene set
- Enrichment direction is consistent with the article's finding that highly tissue-specific genes are 'more likely to be involved in human disease'
- Results table contains all statistics together: Wilcoxon p-values for LOEUF/pLI, Fisher odds ratios and p-values, and Bonferroni-adjusted GO terms

## Limitations

- Colocalization does not necessarily suggest mediation; enriched disease-gene overlap in colocalizing sets can arise from horizontal pleiotropic effects rather than a causal pathway
- CAFEH relies on a single LD matrix, so ancestry-driven LD differences between studies could distort the colocalization-based gene set definitions underlying the enrichment
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in all tested datasets; absent genes/loci will be missing from the enrichment universe
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD with each other, which can misassign top-component sharing and thus gene set membership

## Evidence

- [abstract] Genes with highly tissue-specific genetic effects are under greater selection, enriched in differentiation and developmental processes, and more likely to be involved in human disease: "genes with highly tissue-specific genetic effects are under greater selection, enriched in differentiation and developmental processes, and more likely to be involved in human disease"
- [intro] eQTL tissue specificity associated with selection and disease relevance: "We show that eQTL tissue specificity is associated with signals of selection and disease relevance. That is, tissue-specific genes are under greater selective pressure, and tissue-specific eQTLs are"
- [other] Fisher exact tests for OMIM dominant and recessive disease genes per colocalization-defined gene set: [evidence span withheld — anchor status: not_found]
- [other] Gene set thresholds defined from CAFEH top-component sharing: [evidence span withheld — anchor status: not_found]
- [readme] CAFEH results table columns used for gene set construction: "`top_component` the CAFEH component with the largest probability for that SNP"
