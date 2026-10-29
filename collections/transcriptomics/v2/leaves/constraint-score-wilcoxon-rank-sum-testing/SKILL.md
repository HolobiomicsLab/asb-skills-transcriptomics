---
name: constraint-score-wilcoxon-rank-sum-testing
description: Use when you have gene sets defined by a genomic property (e.g., tissue-specific
  vs shared CAFEH colocalization) and continuous constraint scores (LOEUF, pLI) per
  gene, and you need to test whether the sets differ in selective pressure.
when_to_use_negative:
- The input is variant-level or SNP-level data rather than per-gene constraint scores
  — CAFEH-style fine-mapping, not constraint testing, is the right tool
- You need to test tissue specificity of eQTL effects themselves rather than downstream
  functional consequences of defined gene sets
- Colocalizing signals must be interpreted as mediation — the article warns that colocalization
  does not necessarily suggest mediation and can arise from horizontal pleiotropy
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CAFEH
  role: Produces the colocalization results table (cafeh.{mode}.results) whose top_component
    sharing across tissues defines the gene sets being compared for constraint
  repo: https://github.com/karltayeb/cafeh
- name: Python
  role: Runs the Wilcoxon rank-sum tests, Fisher exact tests, GO enrichment with Bonferroni
    correction, and writes the results table (e.g., via pandas and scipy)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:3ae5c7b8124ffbec5f6413fbd41ada46caae9dd320dd9547f685d7b50a8af3d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/karltayeb/cafeh
  tools:
  - CAFEH
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# constraint-score-wilcoxon-rank-sum-testing

## Summary

A statistical procedure for testing whether gene sets defined by eQTL colocalization structure (e.g., CAFEH top-component sharing across tissues) differ in selective constraint, by comparing LOEUF and pLI score distributions with two-sided Wilcoxon rank-sum tests against a background gene set.

## When to use

Use when you have gene sets defined by a genomic property (e.g., tissue-specific vs shared CAFEH colocalization) and continuous constraint scores (LOEUF, pLI) per gene, and you need to test whether the sets differ in selective pressure.

## When NOT to use

- The input is variant-level or SNP-level data rather than per-gene constraint scores — CAFEH-style fine-mapping, not constraint testing, is the right tool
- You need to test tissue specificity of eQTL effects themselves rather than downstream functional consequences of defined gene sets
- Colocalizing signals must be interpreted as mediation — the article warns that colocalization does not necessarily suggest mediation and can arise from horizontal pleiotropy

## Inputs

- CAFEH results table (cafeh.{mode}.results) with pip, top_component, alpha, rank, p_active columns per SNP-study pair
- LOEUF constraint scores per gene
- pLI constraint scores per gene
- OMIM dominant and recessive disease gene lists
- GO term annotations for Fisher enrichment

## Outputs

- Gene set definitions (background, highly colocalizing, poorly colocalizing)
- Wilcoxon rank-sum test statistics and p-values for LOEUF and pLI comparisons
- Fisher exact odds ratios and p-values for OMIM disease-gene enrichment
- Bonferroni-adjusted significant GO terms per gene set
- Combined results table of gene set definitions and test statistics

## How to apply

Load the CAFEH results table (cafeh.{mode}.results, with per-SNP-study columns pip, top_component, alpha, rank, and p_active) and summarize top-component sharing per gene across tissues. Define the comparison gene sets with the paper's thresholds: background genes with significant cis-eQTLs in >=20 tissues, highly colocalizing genes whose top component is shared in >=20 tissues, and poorly colocalizing genes with top-component sharing in <10 tissues. For each colocalization-defined set, run two-sided Wilcoxon rank-sum tests (scipy.stats.ranksums) comparing the LOEUF and pLI distributions against the background set. Complement the constraint tests with Fisher exact (hypergeometric-style) tests for OMIM dominant/recessive disease-gene enrichment and Bonferroni-corrected GO term Fisher enrichment over the tested GO terms. Write the gene set definitions, Wilcoxon p-values, Fisher odds ratios and p-values, and Bonferroni-adjusted GO terms to a results table; interpret lower LOEUF (stronger variation intolerance) as greater selective constraint.

## Related tools

- **CAFEH** (Produces the colocalization results table (cafeh.{mode}.results) whose top_component sharing across tissues defines the gene sets being compared for constraint) — https://github.com/karltayeb/cafeh
- **Python** (Runs the Wilcoxon rank-sum tests, Fisher exact tests, GO enrichment with Bonferroni correction, and writes the results table (e.g., via pandas and scipy))

## Examples

```
from scipy.stats import ranksums; stat, p = ranksums(hi_coloc_loeuf, background_loeuf, alternative='two-sided')
```

## Evaluation signals

- The three gene sets are non-empty and obey the threshold definitions: background genes with significant cis-eQTLs in >=20 tissues, highly colocalizing genes with top-component sharing >=20 tissues, poorly colocalizing genes with sharing <10 tissues
- Each constraint test is two-sided and paired with its gene-set comparison (highly vs background, poorly vs background), with a reported Wilcoxon p-value per score (LOEUF and pLI)
- Expected directional result from the paper: tissue-specific genes are more variation intolerant (lower LOEUF, higher pLI) than background genes, which are in turn more intolerant than shared-regulation genes
- GO enrichment results are Bonferroni-corrected over the number of tested GO terms, and OMIM enrichment reports odds ratios alongside p-values
- The results table contains gene set definitions plus all test statistics (Wilcoxon p-values, Fisher odds ratios and p-values, adjusted GO terms)

## Limitations

- CAFEH relies on a single LD matrix, so large LD differences between studies (e.g., ancestry differences) could impact colocalization inference and thus gene set membership
- It is difficult to distinguish multiple causal signals among variants in nearly perfect or very high LD, which can misassign top-component sharing
- Loci whose effects are mediated via coding variants may not demonstrate eQTL effects and therefore cannot be explored with CAFEH, excluding them from these gene sets
- Colocalization does not necessarily suggest mediation; colocalizing signals can arise without a linking causal pathway due to horizontal pleiotropic effects

## Evidence

- [abstract] Genes with highly tissue-specific genetic effects are under greater selection, enriched in differentiation and developmental processes, and more likely to be involved in human disease: "genes with highly tissue-specific genetic effects are under greater selection, enriched in differentiation and developmental processes, and more likely to be involved in human disease"
- [other] Run two-sided Wilcoxon rank-sum tests in Python comparing LOEUF and pLI constraint score distributions between the highly colocalizing and poorly colocalizing gene sets against the background.: [evidence span withheld — anchor status: not_found]
- [other] Perform Fisher exact tests (hypergeometric-style overrepresentation) for enrichment of OMIM dominant and recessive disease genes in each colocalization-defined gene set.: [evidence span withheld — anchor status: not_found]
- [readme] The .results model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair.: "The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair."
- [other] Tissue-specific genes are more variation intolerant as measured by LOEUF and pLI compared to background genes, which are in turn more intolerant than genes with shared genetic regulation.: [evidence span withheld — anchor status: not_found]
