---
name: fdr-multiple-hypothesis-correction
description: Use when a gene-list query (e.g. 291 T1D or 229 RA DE genes) is tested
  against hundreds of gene modules simultaneously and raw Fisher/hypergeometric p-values
  must be FDR-adjusted before filtering significant modules.
when_to_use_negative:
- Only a single hypothesis test or a handful of a priori named modules is being tested,
  where no family-wise/FDR multiplicity burden applies
- The goal is comparing enrichment between two unadjusted raw p-value lists — padj
  values from different query sizes/universes are not directly comparable without
  care
- Input is already a table of adjusted p-values or the statistical framework is not
  a Fisher exact/hypergeometric gene-module test
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: E.PAGE
  role: Runs the Fisher exact (hypergeometric) enrichment of each gene module against
    the query and computes Benjamini–Hochberg adjusted p-values in results$stats
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: Curated database (four environmental factors, 25,789 genes, 522 gene modules)
    defining the family of hypotheses over which FDR is corrected
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: tidyverse
  role: Used as a dependency of E.PAGE for tibble output and dplyr filtering/joining
    of padj and fold enrichment values
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:7eb47c85531c4d8562450b78b30b29c35824eeb6ea240a34417bb0ebdb07c678
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - E.PATH
  - R
  - R 4.0.3
  - dplyr
  - tidyverse
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# fdr-multiple-hypothesis-correction

## Summary

Apply Benjamini–Hochberg false discovery rate correction to adjusted p-values from the many Fisher exact (hypergeometric) tests E.PAGE runs across the E.PATH curated database's 522 gene modules, so that enriched environmental pathways are called at a controlled FDR rather than raw p < 0.05.

## When to use

Use when a gene-list query (e.g. 291 T1D or 229 RA DE genes) is tested against hundreds of gene modules simultaneously and raw Fisher/hypergeometric p-values must be FDR-adjusted before filtering significant modules.

## When NOT to use

- Only a single hypothesis test or a handful of a priori named modules is being tested, where no family-wise/FDR multiplicity burden applies
- The goal is comparing enrichment between two unadjusted raw p-value lists — padj values from different query sizes/universes are not directly comparable without care
- Input is already a table of adjusted p-values or the statistical framework is not a Fisher exact/hypergeometric gene-module test

## Inputs

- Raw Fisher exact test p-values for 522 E.PATH gene modules against a query DE gene list (e.g. 291 T1D PBMC genes from GSE12021-style microarray study)
- Module membership counts (genes in module, module genes in universe, query size, universe size) for fold enrichment computation
- Enrichment stats tibble from E.PAGE results$stats

## Outputs

- FDR-adjusted p-values (padj, Benjamini–Hochberg) per gene module in a tidyverse tibble
- Filtered enrichment table (number of modules, number of DE genes, padj, fold enrichment), e.g. t1d_enrichment_table.csv
- Set of statistically significant enriched modules used for cross-disease comparison (e.g. shared_modules.csv)

## How to apply

Run E.PAGE's compute() with the E.PATH curated database (four environmental factors; 25,789 genes; 522 annotated gene modules) so each gene module is tested against the query list with the Fisher exact test (hypergeometric distribution), then correct the resulting p-values with the Benjamini–Hochberg false discovery rate to obtain padj. Because 522 modules are tested at once, significance decisions must be made on padj, not the raw p-value. Compute fold enrichment as (genes in module / query size) / (module genes / universe size) and filter on the adjusted p-value before exporting the enrichment table (e.g. to t1d_enrichment_table.csv). Correctness is judged by reproducing reported entries such as Immune response (padj 1.20E-04, FE 1.59), Infection type: Acute (padj 7.46E-07, FE 1.61), Insulin resistance (padj 1.00E-02) and Cigarette smoking (padj 1.57E-04) for the T1D query.

## Related tools

- **E.PAGE** (Runs the Fisher exact (hypergeometric) enrichment of each gene module against the query and computes Benjamini–Hochberg adjusted p-values in results$stats) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (Curated database (four environmental factors, 25,789 genes, 522 gene modules) defining the family of hypotheses over which FDR is corrected) — https://github.com/AhmedMehdiLab/E.PATH
- **tidyverse** (Used as a dependency of E.PAGE for tibble output and dplyr filtering/joining of padj and fold enrichment values)

## Examples

```
input <- process_input_text(genes, capitalize = TRUE); results <- compute(input); results$stats
```

## Evaluation signals

- Reproduced T1D entries match reported Table 2 values: Immune response padj 1.20E-04 (FE 1.59), Infection type: Acute padj 7.46E-07 (FE 1.61), Insulin resistance padj 1.00E-02, Cigarette smoking padj 1.57E-04
- Every exported module row carries a padj value ≤ its raw p-value (BH monotonicity)
- Filtering is applied on padj, not raw p; the number of significant modules is far fewer than 522 (multiplicity actually applied)
- Fold enrichment values equal (genes in module / query size) / (module genes / universe size) for the reproduced rows
- results$stats is a tidyverse tibble with padj and fold enrichment columns populated for both T1D (291 genes) and RA (229 genes) queries

## Limitations

- Only four environmental factor types are included; FDR correction across 522 modules may mask enrichments as more environmental datasets are integrated
- Possible dataset bias due to limited input data types (gene lists) and similarities between infection- and tissue damage-associated immune responses
- Additional statistical tests such as joint odds ratio and interaction odds ratio are not yet included, limiting statistical representation of the datasets

## Evidence

- [abstract] On testing, we uncovered statistically enriched biological processes, which revealed pathways associated with environmental factors and the genes: "On testing, we uncovered statistically enriched biological processes, which revealed pathways associated with environmental factors and the genes"
- [full_text] BH correction over 522 modules: [evidence span withheld — anchor status: not_found]
- [readme] README: enrichment output format: "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with:"
