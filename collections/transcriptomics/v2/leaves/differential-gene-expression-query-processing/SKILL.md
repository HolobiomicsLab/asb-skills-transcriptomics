---
name: differential-gene-expression-query-processing
description: Use when you have a DE gene list from a disease-vs-control contrast (e.g.,
  GEO microarray study) and need to identify statistically enriched environmental
  pathway gene modules via Fisher exact test with Benjamini-Hochberg FDR correction.
when_to_use_negative:
- Input is not a differential gene expression list — e.g., SNP, CNV, or DNA methylation
  data, which E.PAGE does not yet cover
- You need statistical tests beyond enrichment (joint odds ratio, interaction odds
  ratio), which are not yet included in E.PAGE
- Query environmental contexts outside the four curated factors (cigarette smoking,
  diet, infections, toxic chemicals)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: E.PAGE
  role: 'R 4.0.3 statistical analysis package: process_input_text() parses the DE
    gene list, compute() performs module enrichment against E.PATH'
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: Curated environmental pathway database (25,789 genes; 522 annotated gene modules
    across cigarette smoking, diet, infections, toxic chemicals) used as the enrichment
    reference
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: tidyverse
  role: Public dependency used to handle enrichment result tibbles and export the
    CSV table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:084fc150340c2a5d9293e72ea28e591b29dcb0bee116bbe19225c44f1a03add5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - E.PATH
  - R 4.0.3
  - tidyverse
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# differential-gene-expression-query-processing

## Summary

Process a differential gene expression list (e.g., 291 DE genes from a T1D PBMC microarray study) as text input and run enrichment against the E.PATH curated environmental pathway database (four environmental factors, 25,789 genes, 522 annotated gene modules) using E.PAGE in R 4.0.3.

## When to use

Use when you have a DE gene list from a disease-vs-control contrast (e.g., GEO microarray study) and need to identify statistically enriched environmental pathway gene modules via Fisher exact test with Benjamini-Hochberg FDR correction.

## When NOT to use

- Input is not a differential gene expression list — e.g., SNP, CNV, or DNA methylation data, which E.PAGE does not yet cover
- You need statistical tests beyond enrichment (joint odds ratio, interaction odds ratio), which are not yet included in E.PAGE
- Query environmental contexts outside the four curated factors (cigarette smoking, diet, infections, toxic chemicals)

## Inputs

- Text list of DE gene symbols (e.g., 291 DE genes from new-onset T1D PBMCs vs 24 healthy controls, GSE12021-style microarray study)
- Seurat object with clustering performed (alternative input via process_input_seurat)
- E.PATH curated database (gs_genes list + annotations tibble; 25,789 genes across 522 modules, 4 environmental factors)

## Outputs

- tidyverse tibble of statistically enriched gene modules (results$stats) with padj and fold enrichment
- Per-factor enrichment results via compute_sets() (CSGE, IGE, DGE, TGE headings)
- CSV of enriched gene modules with number of modules, number of DE genes, padj, and fold enrichment (e.g., t1d_enrichment_table.csv)

## How to apply

Load the DE gene list as text and process it with process_input_text() (with capitalize=TRUE if gene symbols need case normalization), then call compute() with the E.PATH database to test each of the 522 gene modules against the query. Compute module significance with the Fisher exact test (hypergeometric distribution) and correct for multiple hypothesis testing using Benjamini-Hochberg FDR to obtain adjusted p-values. Fold enrichment is calculated as (genes in module / query size) / (module genes / universe size); filter results on the adjusted p-value. Export enriched modules with module count, number of DE genes, padj, and fold enrichment to a CSV (e.g., t1d_enrichment_table.csv). Validate by confirming reproduced entries match published values, e.g., T1D Immune response (padj 1.20E-04, FE 1.59), Infection type: Acute (padj 7.46E-07, FE 1.61), Insulin resistance (padj 1.00E-02), Cigarette smoking (padj 1.57E-04). Alternatively, gene lists can be derived from a Seurat object via process_input_seurat() for single-cell cluster DE genes.

## Related tools

- **E.PAGE** (R 4.0.3 statistical analysis package: process_input_text() parses the DE gene list, compute() performs module enrichment against E.PATH) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (Curated environmental pathway database (25,789 genes; 522 annotated gene modules across cigarette smoking, diet, infections, toxic chemicals) used as the enrichment reference) — https://github.com/AhmedMehdiLab/E.PATH
- **tidyverse** (Public dependency used to handle enrichment result tibbles and export the CSV table)

## Examples

```
library(E.PAGE); input <- process_input_text(t1d_de_genes, capitalize=TRUE); results <- compute(input); write.csv(results$stats, "t1d_enrichment_table.csv")
```

## Evaluation signals

- Reproduced T1D key entries match published Table 2 values: Immune response padj 1.20E-04 (FE 1.59), Infection type: Acute padj 7.46E-07 (FE 1.61), Insulin resistance padj 1.00E-02, Cigarette smoking padj 1.57E-04
- All reported padj values are Benjamini-Hochberg FDR-corrected (multiple hypothesis testing over 522 gene modules)
- Output CSV contains the expected columns: number of modules, number of DE genes, padj, fold enrichment
- Fold enrichment invariant: (genes in module / query size) / (module genes / universe size) reproduces reported values (e.g., 1.59, 1.61)
- compute_sets() returns results under all four factor headings: CSGE (cigarette smoking), IGE (infection), DGE (diet), TGE (toxin)

## Limitations

- Database covers only four environmental factor types (cigarette smoking, diet, infections, toxic chemicals); wider use requires integrating more environmental datasets
- Possible dataset bias due to limited input data types (gene list) and similarities between infection and tissue damage-associated immune responses
- Curation restricted to MeSH-term queries of GEO for differential gene expression data
- Joint odds ratio and interaction odds ratio statistical tests not yet included
- Genetic data types (SNPs, CNVs, DNA methylations) not yet covered

## Evidence

- [methods] T1D query workflow: [evidence span withheld — anchor status: not_found]
- [abstract] database scale: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [intro] package purpose: "This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database."
- [readme] README usage: "genes <- "ftl ApoE CTSZ"
input <- process_input_text(genes, capitalize = TRUE)"
- [readme] README result format: "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with:

``` r
results$stats
```"
- [results] validation targets: [evidence span withheld — anchor status: not_found]
- [discussion] limitations: [evidence span withheld — anchor status: not_found]
