---
name: benjamini-hochberg-fdr-correction
description: Use when a gene-list enrichment run (E.PAGE compute/compute_sets over
  the curated 522-module, 25,789-gene database) produces many raw hypergeometric p-values
  that must be adjusted for multiple hypothesis testing before filtering significant
  environmental modules.
when_to_use_negative:
- The analysis goal is joint or interaction odds ratios between gene–environment pairs
  — the article notes these statistical tests are not yet included in E.PAGE
- Input data are SNPs, copy number variants, or DNA methylations rather than a differentially
  expressed gene list — E.PAGE's BH-corrected hypergeometric test does not cover these
  genetic data types
- Only a single pre-specified gene set/module is tested, so there is no multiple-testing
  family to correct
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: E.PAGE
  role: R package whose compute()/compute_sets() run the hypergeometric enrichment
    and apply Benjamini-Hochberg FDR correction to produce p_adj in results$stats
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: R 4.0.3
  role: Implementation language for E.PAGE and the FDR correction workflow
- name: tidyverse
  role: Provides the tibble in which statistically enriched annotations and their
    adjusted p-values are returned and filtered
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:b84ed5088a37122fd2e7c74f451f5668b9566d8b8d4edf0bdeb0d69b26eec066
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - R
  - R 4.0.3
  - RStudio
  - scHiCDiff
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# benjamini-hochberg-fdr-correction

## Summary

Applies the Benjamini-Hochberg false discovery rate method to adjust raw Fisher exact (hypergeometric) p-values from E.PAGE environmental gene-module enrichment, yielding p_adj values used to call statistically enriched modules (e.g. infection, cigarette smoking, toxic-chemical categories).

## When to use

Use when a gene-list enrichment run (E.PAGE compute/compute_sets over the curated 522-module, 25,789-gene database) produces many raw hypergeometric p-values that must be adjusted for multiple hypothesis testing before filtering significant environmental modules.

## When NOT to use

- The analysis goal is joint or interaction odds ratios between gene–environment pairs — the article notes these statistical tests are not yet included in E.PAGE
- Input data are SNPs, copy number variants, or DNA methylations rather than a differentially expressed gene list — E.PAGE's BH-corrected hypergeometric test does not cover these genetic data types
- Only a single pre-specified gene set/module is tested, so there is no multiple-testing family to correct

## Inputs

- Vector of raw p-values from Fisher exact / hypergeometric tests of each E.PAGE gene module against the query
- Query gene list of interest (e.g. 229 RA DE genes, 71 SCLC DE genes, 27 cobalt-exposure DE genes) processed via process_input_text or process_input_seurat
- Curated E.PAGE environmental database (522 gene modules, 25,789 genes; four factors: cigarette smoking, diet, infections, toxic chemicals)

## Outputs

- BH-adjusted p-values (p_adj) per gene module, stored in the results$stats tidyverse tibble
- Enrichment table with gene module, number of DE genes, p_adj, fold enrichment (e.g. exported as sclc_enrichment_table.csv or smoking_enrichment_results.csv)

## How to apply

First compute per-module enrichment with E.PAGE's compute() or compute_sets(), which tests each of the 522 manually annotated gene modules against the query gene list via a Fisher exact test (hypergeometric distribution), e.g. the 229 RA DE genes from GSE12021/GSE55457/GSE55584/GSE55235 or the 27 cobalt-exposure DE genes. Then correct the family of module p-values with the Benjamini-Hochberg FDR method to obtain p_adj, since hundreds of modules are tested simultaneously and unadjusted p-values would be wildly optimistic. Fold enrichment is computed alongside as F.E = (N_gm/n(g))/(N_m^tot/w(U)). Filter or rank the results$stats tidyverse tibble on p_adj to report significant modules, and validate against paper-reported adjusted values such as 'Infection type: Acute' padj 8.49E-34 and 'Immune response' padj 1.03E-27 (RA), 'Cell cycle' padj 8.49E-07 (SCLC), or the cobalt Genotoxic/Carcinogen modules at p_adj 1.84E-05.

## Related tools

- **E.PAGE** (R package whose compute()/compute_sets() run the hypergeometric enrichment and apply Benjamini-Hochberg FDR correction to produce p_adj in results$stats) — https://github.com/AhmedMehdiLab/E.PAGE
- **R 4.0.3** (Implementation language for E.PAGE and the FDR correction workflow)
- **tidyverse** (Provides the tibble in which statistically enriched annotations and their adjusted p-values are returned and filtered)

## Examples

```
results <- compute(input); head(results$stats[order(results$stats$p_adj), ])
```

## Evaluation signals

- Reproduced p_adj values match the paper's reported tables within floating-point/rounding tolerance (e.g. RA 'Infection type: Acute' padj 8.49E-34, 'Immune response' padj 1.03E-27)
- For every module, p_adj is greater than or equal to the raw hypergeometric p-value (BH monotonicity invariant)
- Cobalt-exposure run recovers all six reported modules (Genotoxic, Non-genotoxic, Carcinogen, Hepatocarcinogens, Liver-based in vitro models, Chemical induced gene expression) with fold enrichment ~12 and p_adj = 1.84E-05
- Significant modules are selected only after filtering results$stats on the adjusted p-value column, not on raw p-values
- Result object is a tidyverse tibble exposing p_adj, fold enrichment, and odds ratio columns as returned by compute()

## Limitations

- The curated database covers only four environmental variable types, so BH correction is applied within a limited module universe; more environmental datasets would need to be integrated for wider use
- Possible dataset bias due to limited input data types, such as similarities between infection and tissue damage-associated immune responses
- Additional statistical tests such as joint odds ratio and interaction odds ratio are not yet included, limiting the statistical representation of the datasets

## Evidence

- [other] Correct p-values for multiple hypothesis testing with the Benjamini-Hochberg FDR method and compute fold enrichment: [evidence span withheld — anchor status: not_found]
- [other] Correct module p-values for multiple hypothesis testing using the Benjamini-Hochberg false discovery rate method to obtain adjusted p-values (p_adj): [evidence span withheld — anchor status: not_found]
- [methods] The statistical analysis package E.PAGE was written in R version 4.0.3: "The statistical analysis package E.PAGE (Environmental Pathways Affecting Gene Expression) (https://github.com/AhmedMehdiLab/E.PAGE) was written in R version 4.0.3"
- [readme] Statistically enriched annotations are stored in a tidyverse tibble (README): "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with:"
- [abstract] Database scale context: 522 modules across four environmental factors (abstract): "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
