---
name: hypergeometric-statistical-testing
description: Use when given a list of differentially expressed genes (e.g. 229 RA
  genes from GSE12021/GSE55457/GSE55584/GSE55235) and you need statistically enriched
  environmental or infection gene modules with FDR-adjusted p-values.
when_to_use_negative:
- Input is a full expression matrix or Seurat object without clustering/differential
  expression already performed — E.PAGE's enrichment path expects a gene list, not
  raw counts
- The goal is joint odds ratio or interaction odds ratio testing — the article notes
  these additional statistical tests are not yet included
- The query concerns SNPs, CNVs or DNA methylations — genetic data types are not covered
  by the current database
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: E.PAGE
  role: R package performing Fisher exact (hypergeometric) over-representation testing
    of the gene list against the curated environmental database with Benjamini-Hochberg
    FDR correction
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: Companion R package supplying the curated database (gs_genes gene sets and
    annotations) that E.PAGE tests against
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: R 4.0.3
  role: Runtime environment in which E.PAGE's statistical enrichment is written and
    executed
- name: tidyverse
  role: Dependency providing the tibble structures in which enriched annotations are
    stored and filtered
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:1729617c4ca5e8a4882a4265a65d07e44f86a4376363169f6fd3274b9b00979d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - E.PATH
  - R 4.0.3
  - tidyverse
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hypergeometric-statistical-testing

## Summary

Over-representation testing of a gene list against a curated environmental-pathway database (522 gene modules, 25,789 genes) using Fisher exact (hypergeometric) p-values corrected with Benjamini-Hochberg FDR, as implemented in the E.PAGE R package.

## When to use

Use when given a list of differentially expressed genes (e.g. 229 RA genes from GSE12021/GSE55457/GSE55584/GSE55235) and you need statistically enriched environmental or infection gene modules with FDR-adjusted p-values.

## When NOT to use

- Input is a full expression matrix or Seurat object without clustering/differential expression already performed — E.PAGE's enrichment path expects a gene list, not raw counts
- The goal is joint odds ratio or interaction odds ratio testing — the article notes these additional statistical tests are not yet included
- The query concerns SNPs, CNVs or DNA methylations — genetic data types are not covered by the current database

## Inputs

- List of genes of interest (text string or file), e.g. 229 differentially expressed genes from RA microarray datasets
- E.PAGE default curated database (522 gene modules; 25,789 genes) or user-imported .csv annotation/database files
- Optional org.Hs.eg.db for Gene Ontology term analysis

## Outputs

- Tidyverse tibble of statistically enriched annotations per heading (results$stats) with Fisher exact p-values, Benjamini-Hochberg FDR-adjusted p-values, and fold enrichment
- Per-category result lists under CSGE (cigarette smoking), IGE (infection), DGE (diet) and TGE (toxin) headings

## How to apply

Provide the gene list to E.PAGE's process_input_text with capitalize = TRUE (R 4.0.3), then call compute_sets against the default curated database of 522 manually annotated gene modules covering cigarette smoking, diet, infections and toxic chemicals. Enrichment is computed per heading (CSGE, IGE, DGE, TGE) using Fisher exact test (hypergeometric) p-values, which are corrected with the Benjamini-Hochberg FDR method. Filter the resulting tidyverse tibble (e.g. results$stats) to the heading of interest and report adjusted p-values together with fold enrichment. Validate by comparing reproduced rows to published values — for the RA test set, 'Infection type: Acute' padj 8.49E-34 (fold enrichment 2.63) and 'Immune response' padj 1.03E-27 (fold enrichment 2.87) — accepting matches within rounding tolerance.

## Related tools

- **E.PAGE** (R package performing Fisher exact (hypergeometric) over-representation testing of the gene list against the curated environmental database with Benjamini-Hochberg FDR correction) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (Companion R package supplying the curated database (gs_genes gene sets and annotations) that E.PAGE tests against) — https://github.com/AhmedMehdiLab/E.PATH
- **R 4.0.3** (Runtime environment in which E.PAGE's statistical enrichment is written and executed)
- **tidyverse** (Dependency providing the tibble structures in which enriched annotations are stored and filtered)

## Examples

```
library(E.PAGE); input <- process_input_text(genes, capitalize = TRUE); results <- compute_sets(input); results$stats
```

## Evaluation signals

- Reproduced 'Infection type: Acute' row matches padj 8.49E-34 with fold enrichment 2.63 within rounding tolerance
- Reproduced 'Immune response' row matches padj 1.03E-27 with fold enrichment 2.87 within rounding tolerance
- results$stats is a tidyverse tibble containing adjusted p-values and fold enrichments, and compute_sets returns results under all four headings (CSGE, IGE, DGE, TGE)
- Adjusted p-values are monotone with raw Fisher p-values under Benjamini-Hochberg correction and no enrichment rows appear outside the four environmental headings
- Input gene list resolves after process_input_text(capitalize = TRUE); unrecognised gene symbols should not silently inflate the module contingency tables

## Limitations

- Database is limited to four environmental variable types (cigarette smoking, diet, infections, toxic chemicals); more environmental datasets need integration for wider use
- Possible dataset bias due to limited input data types such as gene lists, e.g. similarities between infection and tissue damage-associated immune responses can confound interpretation
- Additional statistical tests such as joint odds ratio and interaction odds ratio are not yet included
- Genetic data types (SNPs, CNVs, DNA methylations) are not covered

## Evidence

- [methods] Fisher exact test with BH FDR: [evidence span withheld — anchor status: not_found]
- [abstract] Curated database size: "a total of 237 datasets were selected and 522 gene modules were manually annotated"
- [abstract] Four environmental factors and gene count: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [readme] README basic usage: "input <- process_input_text(genes, capitalize = TRUE)"
- [readme] README per-category results: "result will be a list with CSGE (cigarette smoking), IGE (infection), DGE (diet) and TGE (toxin) headings"
- [methods] Statistical analysis package implementation: "The statistical analysis package E.PAGE (Environmental Pathways Affecting Gene Expression) (https://github.com/AhmedMehdiLab/E.PAGE) was written in R version 4.0.3"
