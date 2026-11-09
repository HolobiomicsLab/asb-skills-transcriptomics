---
name: go-term-overrepresentation-analysis
description: Use when you have a CAFEH results table (cafeh.{mode}.results with pip,
  top_component, alpha, rank, p_active per SNP-study pair) and want to test whether
  tissue-specific vs shared eQTL regulation differs in constraint, disease association,
  and functional pathways.
when_to_use_negative:
- No CAFEH results table exists yet — the gene sets depend on top_component sharing
  across tissues, so colocalization/fine-mapping must be run first
- Loci whose disease effects are mediated via coding variants without eQTL effects
  — per the paper these 'may not demonstrate eQTL effects and therefore cannot be
  explored with CAFEH', so no colocalization-based gene sets can be defined
- You need ranked-list enrichment (e.g., GSEA) rather than binary gene-set overrepresentation
  — this skill uses Fisher exact tests on dichotomized gene sets
edam_operation: http://edamontology.org/operation_3662
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: CAFEH
  role: Produces the cafeh.{mode}.results table whose top_component and p_active columns
    define the highly/poorly colocalizing gene sets that feed the enrichment analyses
  repo: github.com/karltayeb/cafeh
- name: Python
  role: Used with pandas to load the CAFEH results table, define gene sets, and run
    Wilcoxon and Fisher (GO/OMIM) tests
- name: pip
  role: Installs the CAFEH package into the cafeh conda environment before generating
    the results table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ajhg.2022.01.002
    title: Redefining tissue specificity of genetic regulation of gene expression
      in the presence of allelic heterogeneity
  source_content_hash: sha256:94789aebcbbf2fa9f37164a3fa7a634499de8a5a18db0ba4e775c46d36a705d9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAFEH
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3662
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# go-term-overrepresentation-analysis

## Summary

Defines gene sets from CAFEH colocalization results (highly vs poorly colocalizing genes against a background of genes with significant cis-eQTLs in >=20 tissues) and tests them for selective constraint (LOEUF, pLI Wilcoxon tests), OMIM disease-gene enrichment (Fisher exact tests), and GO term overrepresentation with Bonferroni correction.

## When to use

Use when you have a CAFEH results table (cafeh.{mode}.results with pip, top_component, alpha, rank, p_active per SNP-study pair) and want to test whether tissue-specific vs shared eQTL regulation differs in constraint, disease association, and functional pathways.

## When NOT to use

- No CAFEH results table exists yet — the gene sets depend on top_component sharing across tissues, so colocalization/fine-mapping must be run first
- Loci whose disease effects are mediated via coding variants without eQTL effects — per the paper these 'may not demonstrate eQTL effects and therefore cannot be explored with CAFEH', so no colocalization-based gene sets can be defined
- You need ranked-list enrichment (e.g., GSEA) rather than binary gene-set overrepresentation — this skill uses Fisher exact tests on dichotomized gene sets

## Inputs

- CAFEH results table (cafeh.{mode}.results, tab-delimited, one row per SNP-study pair with pip, top_component, alpha, rank, p_active, pi, effect, effect_var columns)
- Gene sets defined by CAFEH top-component sharing thresholds (>=20 tissues shared, <10 tissues shared, background with significant cis-eQTLs in >=20 tissues)
- LOEUF and pLI constraint scores per gene
- OMIM dominant and recessive disease gene annotations
- GO term-to-gene associations

## Outputs

- Gene set definitions table (highly colocalizing, poorly colocalizing, background)
- Wilcoxon rank-sum p-values for LOEUF and pLI comparisons against background
- Fisher exact odds ratios and p-values for OMIM dominant/recessive disease gene enrichment
- Bonferroni-adjusted significant GO terms per gene set

## How to apply

Load the CAFEH results table with pandas and summarize top-component sharing per gene across tissues: count, for each gene, the number of studies/tissues in which the top_component is shared. Define three gene sets: background genes with significant cis-eQTLs in >=20 tissues, highly colocalizing genes with top-component sharing in >=20 tissues, and poorly colocalizing genes with sharing in <10 tissues. Run two-sided Wilcoxon rank-sum tests comparing LOEUF and pLI constraint score distributions of highly and poorly colocalizing sets against the background. Perform Fisher exact tests (hypergeometric-style overrepresentation) for OMIM dominant and recessive disease genes, and run GO term Fisher enrichment per gene set, applying Bonferroni correction over tested GO terms to call significant categories. Rationale: the paper found genes with highly tissue-specific genetic effects are under greater selection, enriched in differentiation and developmental processes, and more likely to be involved in human disease, so this downstream analysis quantifies those associations.

## Related tools

- **CAFEH** (Produces the cafeh.{mode}.results table whose top_component and p_active columns define the highly/poorly colocalizing gene sets that feed the enrichment analyses) — github.com/karltayeb/cafeh
- **Python** (Used with pandas to load the CAFEH results table, define gene sets, and run Wilcoxon and Fisher (GO/OMIM) tests)
- **pip** (Installs the CAFEH package into the cafeh conda environment before generating the results table)

## Examples

```
python cafeh.py --mode z -R example/LD_example.tsv -z example/z_example.tsv -n example/n_example.tsv --out output --save-model
```

## Evaluation signals

- The results table contains, for each gene set, both a Wilcoxon p-value for LOEUF/pLI and Fisher odds ratios/p-values for OMIM dominant and recessive enrichment
- Gene set definitions are reproducible from the cafeh.{mode}.results table using the stated thresholds: top-component sharing >=20 tissues (highly colocalizing), <10 tissues (poorly colocalizing), and significant cis-eQTLs in >=20 tissues (background)
- GO enrichment reports Bonferroni-adjusted p-values computed over the tested GO terms, not raw Fisher p-values
- The expected directional finding is that highly tissue-specific genes are more variation intolerant (higher constraint: LOEUF/pLI) than poorly colocalizing genes, which are in turn more intolerant than shared-regulation genes
- The results file row granularity matches the README: one row per (SNP, study) pair before gene-level aggregation

## Limitations

- CAFEH relies on a single LD matrix; large LD differences between studies (e.g., ancestry differences) could impact colocalization and hence gene set assignment
- CAFEH does not handle missing data in the effect size matrix, implicitly assuming causal variants are present in all tested datasets
- It is difficult to distinguish multiple signals among variants in nearly perfect or very high LD, which can blur top_component-based tissue-sharing counts
- Colocalization does not necessarily suggest mediation — colocalizing signals may arise through horizontal pleiotropy, limiting causal interpretation of disease-gene enrichment

## Evidence

- [abstract] genes with highly tissue-specific genetic effects are under greater selection, enriched in differentiation and developmental processes, and more likely to be involved in human disease: "genes with highly tissue-specific genetic effects are under greater selection, enriched in differentiation and developmental processes, and more likely to be involved in human disease"
- [intro] We show that eQTL tissue specificity is associated with signals of selection and disease relevance: "We show that eQTL tissue specificity is associated with signals of selection and disease relevance."
- [readme] The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair.: "The `.results` model provides a useful summary of CAFEH's output. It contains a row for each (SNP, study) pair."
- [readme] pip The posterior inclusion probability for that SNP in that study: "`pip` The posterior inclusion probability for that SNP in that study"
- [methods] git clone https://github.com/karltayeb/cafeh.git: "git clone https://github.com/karltayeb/cafeh.git"
