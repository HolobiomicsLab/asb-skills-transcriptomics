---
name: colocalization-component-based-gene-set-definition
description: Use when you have a CAFEH results table (cafeh.{mode}.results) with pip/top_component/alpha/rank/p_active
  columns and want to convert top-component sharing across tissues into gene sets
  for constraint, OMIM, and GO enrichment testing.
when_to_use_negative:
- Loci whose trait effects are mediated via coding variants without eQTL effects —
  the article notes such loci cannot be explored with CAFEH, so no colocalization
  components exist to stratify
- Studies with large LD differences between them (e.g., ancestry differences) — CAFEH
  relies on a single LD matrix, so top_component sharing across such studies is unreliable
- 'When the goal is to claim disease-eQTL causality: colocalization does not imply
  mediation, and colocalizing signals can arise from horizontal pleiotropy'
edam_operation: http://edamontology.org/operation_3777
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CAFEH
  role: Joint fine-mapping/colocalization method that produces the results table whose
    top_component and p_active columns are used to quantify how many tissues share
    each gene's causal component
  repo: github.com/karltayeb/cafeh
- name: Python
  role: Used as a package (pandas) to load cafeh.{mode}.results, summarize top-component
    sharing per gene, define the three gene sets, and run Wilcoxon, Fisher, and Bonferroni-corrected
    GO enrichment tests
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:1a3ba3541b31240276d6ca2349a3833871fd4815021c9d298acc133452839651
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - Python
  edam_operation: http://edamontology.org/operation_3777
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# colocalization-component-based-gene-set-definition

## Summary

Defines biologically interpretable gene sets from CAFEH colocalization output (top_component, p_active, pip per SNP-study pair) by stratifying genes into highly colocalizing, poorly colocalizing, and background groups based on how many tissues share the CAFEH top component, then tests the sets for constraint (LOEUF, pLI), OMIM disease, and GO enrichment.

## When to use

Use when you have a CAFEH results table (cafeh.{mode}.results) with pip/top_component/alpha/rank/p_active columns and want to convert top-component sharing across tissues into gene sets for constraint, OMIM, and GO enrichment testing.

## When NOT to use

- Loci whose trait effects are mediated via coding variants without eQTL effects — the article notes such loci cannot be explored with CAFEH, so no colocalization components exist to stratify
- Studies with large LD differences between them (e.g., ancestry differences) — CAFEH relies on a single LD matrix, so top_component sharing across such studies is unreliable
- When the goal is to claim disease-eQTL causality: colocalization does not imply mediation, and colocalizing signals can arise from horizontal pleiotropy

## Inputs

- cafeh.{mode}.results table (tab-delimited; one row per SNP-study pair with pip, top_component, alpha, rank, p_active columns)
- cafeh.{mode}.model pickle (optional, for querying the fitted CAFEHSummary/CAFEHGenotype object)
- Gene-by-tissue cis-eQTL significance matrix (e.g., GTEx v7) to count significant tissues per gene
- Constraint scores per gene: LOEUF and pLI
- OMIM dominant and recessive disease gene lists
- GO term annotations per gene

## Outputs

- Gene set definitions: highly colocalizing (top_component shared in >=20 tissues), poorly colocalizing (<10 tissues), background (significant cis-eQTLs in >=20 tissues)
- Wilcoxon rank-sum p-values comparing LOEUF and pLI distributions of highly vs poorly colocalizing sets against background
- Fisher exact odds ratios and p-values for OMIM dominant/recessive enrichment per gene set
- Bonferroni-adjusted significant GO terms per gene set
- Combined results table of gene set definitions and all test statistics

## How to apply

Load the CAFEH results table (cafeh.{mode}.results, one row per SNP-study pair) into pandas and summarize, per gene, how many tissues share the CAFEH top_component (using p_active and pip to quantify component activity). Define three gene sets: background genes with significant cis-eQTLs in >=20 tissues, highly colocalizing genes whose top component is shared in >=20 tissues, and poorly colocalizing genes with top-component sharing in <10 tissues. Run two-sided Wilcoxon rank-sum tests comparing LOEUF and pLI constraint score distributions of each colocalization-defined set against the background. Perform Fisher exact (hypergeometric-style overrepresentation) tests for OMIM dominant and recessive disease genes in each set, and Fisher GO term enrichment with Bonferroni correction over tested GO terms. The rationale is that CAFEH's joint fine-mapping assigns each SNP-study pair to a shared vs tissue-specific causal component, so top-component sharing is a cleaner measure of tissue-specific genetic regulation than LD-confounded meta-analysis sharing.

## Related tools

- **CAFEH** (Joint fine-mapping/colocalization method that produces the results table whose top_component and p_active columns are used to quantify how many tissues share each gene's causal component) — github.com/karltayeb/cafeh
- **Python** (Used as a package (pandas) to load cafeh.{mode}.results, summarize top-component sharing per gene, define the three gene sets, and run Wilcoxon, Fisher, and Bonferroni-corrected GO enrichment tests)

## Examples

```
python cafeh.py --mode beta -R example/LD_example.tsv -B example/beta_example.tsv -S example/stderr_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- Gene sets reproduce the article's thresholds: highly colocalizing = top component shared in >=20 tissues, poorly colocalizing = <10 tissues, background = significant cis-eQTLs in >=20 tissues
- cafeh.{mode}.results loaded without schema errors, with pip, top_component, alpha, rank, and p_active columns present for every SNP-study pair
- Direction of constraint test matches the reported finding: tissue-specific genes are more variation intolerant (higher pLI / more extreme LOEUF) than background, which are more intolerant than shared-regulation genes
- Wilcoxon tests are two-sided rank-sum tests on constraint score distributions, and Fisher tests yield both odds ratios and p-values for OMIM dominant and recessive sets
- GO enrichment p-values are Bonferroni-corrected over the number of tested GO terms before significance is called

## Limitations

- It remains difficult to distinguish multiple causal signals among variants in nearly perfect or very high LD, which can blur top_component assignments
- CAFEH includes a simple prior on variant causality (equal prior probability across variants) and does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in all tested datasets
- Colocalization does not imply mediation; horizontal pleiotropic effects can produce colocalizing signals without a causal pathway

## Evidence

- [abstract] genes with highly tissue-specific genetic effects are under greater selection, enriched in differentiation and developmental processes, and more likely to be involved in human disease: "genes with highly tissue-specific genetic effects are under greater selection, enriched in differentiation and developmental processes, and more likely to be involved in human disease"
- [readme] top_component definition: "`top_component` the CAFEH component with the largest probability for that SNP"
- [readme] p_active definition: "`p_active` gives the probability that `top_component` is an active component in this study"
- [intro] tissue specificity linked to selection and disease: "We show that eQTL tissue specificity is associated with signals of selection and disease relevance."
