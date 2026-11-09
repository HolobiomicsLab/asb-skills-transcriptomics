---
name: enrichment-statistics-comparison-across-datasets
description: Use when you have two E.PAGE enrichment result objects (results$stats
  tibbles) from distinct gene lists and need to identify which enriched gene modules
  are common to both, comparing padj and fold enrichment per module.
when_to_use_negative:
- You have a single gene list and only need per-list enrichment — run E.PAGE compute()
  directly rather than intersecting two result sets
- Your input is a Seurat object requiring clustering/differential expression first
  — use E.PAGE's Seurat input path instead of comparing pre-existing enrichment tables
- The enrichment outputs come from different annotation/database versions, making
  module names non-comparable for an inner_join
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: E.PAGE
  role: Computes enrichment of environmental gene modules for each query gene list,
    returning significant modules with FDR-adjusted p-values in results$stats
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: R
  role: Runtime (version 4.0.3 in the paper) for filtering, joining and exporting
    the shared-module table
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: tidyverse
  role: Dependency used to manipulate the enrichment tibbles and assemble the tidy
    shared-module table
- name: dplyr
  role: inner_join on gene-module names to intersect the two significant-module lists
- name: E.PATH
  role: Companion database package (annotations, gs_genes gene-set lists) supplying
    the curated gene modules against which enrichment is tested
  repo: https://github.com/AhmedMehdiLab/E.PATH
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:8d55d61fbbfffc4a0cf0ae9c2222572d3eb409aed60e1736c39994216102af78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - E.PATH
  - R
  - dplyr
  - tidyverse
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# enrichment-statistics-comparison-across-datasets

## Summary

Intersects statistically significant enrichment results (FDR-adjusted p-values from E.PAGE) computed for two disease query signatures to identify shared gene modules, as demonstrated for T1D (291 DE genes) and RA (229 DE genes) queries that both enriched infection-associated modules.

## When to use

Use when you have two E.PAGE enrichment result objects (results$stats tibbles) from distinct gene lists and need to identify which enriched gene modules are common to both, comparing padj and fold enrichment per module.

## When NOT to use

- You have a single gene list and only need per-list enrichment — run E.PAGE compute() directly rather than intersecting two result sets
- Your input is a Seurat object requiring clustering/differential expression first — use E.PAGE's Seurat input path instead of comparing pre-existing enrichment tables
- The enrichment outputs come from different annotation/database versions, making module names non-comparable for an inner_join

## Inputs

- E.PAGE enrichment results object (results$stats tidyverse tibble) for query signature 1 (e.g. T1D, 291 DE genes)
- E.PAGE enrichment results object (results$stats tibble) for query signature 2 (e.g. RA, 229 DE genes)
- Gene lists of interest processed via process_input_text (e.g. DE genes from GEO/MSigDB C7 datasets such as GSE12021, GSE55457, GSE55584, GSE55235)

## Outputs

- Tidy table (CSV, e.g. shared_modules.csv) of shared significant gene modules with padj and fold enrichment from each disease query
- Expected shared modules for T1D/RA: Interferons, IL-12, Transcription regulation, Immune response (infection-associated)

## How to apply

Load the E.PAGE enrichment output tibbles (results$stats) computed for each query signature in R (the paper used R 4.0.3 with tidyverse/dplyr). Filter each tibble to statistically significant gene modules using the Benjamini–Hochberg FDR-adjusted p-value (padj) reported by E.PAGE. Use dplyr::inner_join on gene-module names to intersect the two significant-module lists, then assemble a tidy table with each shared module's padj and fold enrichment from both queries and export it (e.g. shared_modules.csv). Rationale: E.PAGE stores statistically enriched annotations in a tidyverse tibble accessible via results$stats, and computes enrichment per environmental category (CSGE, IGE, DGE, TGE), so the shared set reveals environmental factors — here infection-associated modules such as Interferons, IL-12, Transcription regulation and Immune response — common to both disease gene lists.

## Related tools

- **E.PAGE** (Computes enrichment of environmental gene modules for each query gene list, returning significant modules with FDR-adjusted p-values in results$stats) — https://github.com/AhmedMehdiLab/E.PAGE
- **R** (Runtime (version 4.0.3 in the paper) for filtering, joining and exporting the shared-module table) — https://github.com/AhmedMehdiLab/E.PAGE
- **tidyverse** (Dependency used to manipulate the enrichment tibbles and assemble the tidy shared-module table)
- **dplyr** (inner_join on gene-module names to intersect the two significant-module lists)
- **E.PATH** (Companion database package (annotations, gs_genes gene-set lists) supplying the curated gene modules against which enrichment is tested) — https://github.com/AhmedMehdiLab/E.PATH

## Examples

```
library(E.PAGE); input <- process_input_text(genes, capitalize=TRUE); results <- compute(input); shared <- dplyr::inner_join(filter(results$stats, padj<0.05), filter(results_ra$stats, padj<0.05), by="module"); write.csv(shared, "shared_modules.csv")
```

## Evaluation signals

- The shared-module table contains the modules reported as common in the paper (Interferons, IL-12, Transcription regulation, Immune response) with both T1D and RA padj and fold-enrichment values populated
- Every module in the output passes the Benjamini–Hochberg padj significance threshold in BOTH input enrichment tibbles
- Output is a tidy data frame with columns for module name, per-disease padj and per-disease fold enrichment, written successfully to shared_modules.csv
- Counts of significant modules before intersection match the sum after removing duplicates-only (inner_join row count ≤ min of the two filtered sets)

## Limitations

- The curated database covers only four environmental factors (cigarette smoking, diet, infections, toxic chemicals); shared modules reflect only these categories
- Possible dataset bias due to limited input data types, e.g. similarities between infection and tissue damage-associated immune responses may confound module assignment
- Statistical tests such as joint odds ratio and interaction odds ratio are not yet included, limiting cross-dataset statistical comparison
- Results depend on the query gene-list size (291 T1D and 229 RA DE genes in the reference case)

## Evidence

- [discussion] T1D/RA shared infection modules: "The T1D and RA associated DE genes were found to be primarily enriched in infection-associated gene modules and less in gene modules associated with the environmental factors diet, cigarette smoking"
- [readme] E.PAGE output structure: "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with: results$stats"
- [intro] E.PAGE purpose: "This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database."
- [methods] R version and dependencies: "The statistical analysis package E.PAGE (Environmental Pathways Affecting Gene Expression) (https://github.com/AhmedMehdiLab/E.PAGE) was written in R version 4.0.3"
- [methods] Input from published DE gene datasets: "Names of Differentially expressed genes were extracted from GEO and MSigDB C7 databases."
