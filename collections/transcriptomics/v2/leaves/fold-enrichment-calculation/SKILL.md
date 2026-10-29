---
name: fold-enrichment-calculation
description: Use when a set of differentially expressed genes (e.g. from SCLC samples,
  cobalt-exposed rat liver cells, or a Seurat object) must be tested for over-representation
  in curated gene-environment modules, and fold enrichment with FDR-corrected significance
  is the required output.
when_to_use_negative:
- The input is raw sequencing or microarray data that has not yet been reduced to
  a list of differentially expressed genes — E.PAGE requires a gene list (or a Seurat
  object with clustering already performed)
- The question requires joint odds ratio or interaction odds ratio tests, or coverage
  of SNPs, CNVs, or DNA methylations — these statistical tests and genetic data types
  are not included in E.PAGE
- The environmental factor of interest is outside the four covered factors (cigarette
  smoking, diet, infections, toxic chemicals) — the article notes more environmental
  datasets need to be integrated
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: E.PAGE
  role: Statistical analysis package that runs the Fisher exact / hypergeometric test
    per module, applies Benjamini-Hochberg FDR, and returns the enriched tibble in
    results$stats
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: Supplies the curated Environmental Pathways Database (E.PATH::database with
    gs_genes and E.PATH::annotations) that E.PAGE tests the query against
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: R 4.0.3
  role: Language in which E.PAGE was written and in which the enrichment computation
    and F.E formula are executed
- name: tidyverse
  role: Dependency providing the tibble structure of the enrichment results and used
    to filter on p_adj and assemble the enrichment table
- name: Seurat
  role: 'Alternative query source: E.PAGE accepts a Seurat object (with clustering
    performed) as input and performs differential expression analyses between the
    clusters'
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:38aa051c76a69ea651d8b8e358ef581776634828f41cea1fa0d5660bd900478b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - E.PATH
  - R
  - R 4.0.3
  - RStudio
  - Seurat
  - tidyverse
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# fold-enrichment-calculation

## Summary

Computes fold enrichment (F.E) and Benjamini-Hochberg adjusted p-values for environmental gene modules tested against a query gene list with the E.PAGE R package, which tests each of 522 manually annotated modules via hypergeometric/Fisher exact test against a curated database of 237 datasets / 25,789 genes.

## When to use

Use when a set of differentially expressed genes (e.g. from SCLC samples, cobalt-exposed rat liver cells, or a Seurat object) must be tested for over-representation in curated gene-environment modules, and fold enrichment with FDR-corrected significance is the required output.

## When NOT to use

- The input is raw sequencing or microarray data that has not yet been reduced to a list of differentially expressed genes — E.PAGE requires a gene list (or a Seurat object with clustering already performed)
- The question requires joint odds ratio or interaction odds ratio tests, or coverage of SNPs, CNVs, or DNA methylations — these statistical tests and genetic data types are not included in E.PAGE
- The environmental factor of interest is outside the four covered factors (cigarette smoking, diet, infections, toxic chemicals) — the article notes more environmental datasets need to be integrated

## Inputs

- Query gene list as text (e.g. a 71-DE-gene SCLC signature or 27-DE-gene cobalt-exposure signature)
- E.PAGE input object from process_input_text() or process_input_seurat()
- Curated E.PAGE database (237 datasets, 25,789 genes, 522 annotated modules); optional custom annotation (.csv) and database (.csv) via import_annotations()/import_database(); optional org.Hs.eg.db for GO terms

## Outputs

- Enrichment tibble in results$stats with per-module p_adj and fold enrichment (F.E)
- Category-specific result list with CSGE, IGE, DGE, TGE headings from compute_sets()
- Exportable enrichment table (e.g. sclc_enrichment_table.csv) with columns: gene module, number of modules, number of DE genes, p_adj, fold enrichment

## How to apply

Preprocess the query with process_input_text(genes, capitalize = TRUE) (or process_input_seurat for a clustered Seurat object), then call compute() against the curated E.PAGE database, which tests each gene module against the query using a hypergeometric distribution / Fisher exact test. Correct module p-values with the Benjamini-Hochberg FDR method and compute fold enrichment as F.E = (N_gm/n(g))/(N_m^tot/w(U)) — the ratio of module genes in the query to module genes in the universe background. Retrieve the enriched module table (a tidyverse tibble) via results$stats; per-category results (CSGE cigarette smoking, IGE infection, DGE diet, TGE toxin) come from compute_sets(). Filter on p_adj and report gene module, number of modules, number of DE genes, p_adj, and fold enrichment. As validation, a 71-DE-gene SCLC query should recover Cell cycle (F.E 12.98, p_adj 8.49E-07) and Cytoprotective mechanism (F.E 5.33, p_adj 7.28E-08); a 27-gene cobalt query should recover six Genotoxicity/Carcinogen-type modules with F.E ~12, p_adj = 1.84E-05, all 27 query genes per module.

## Related tools

- **E.PAGE** (Statistical analysis package that runs the Fisher exact / hypergeometric test per module, applies Benjamini-Hochberg FDR, and returns the enriched tibble in results$stats) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (Supplies the curated Environmental Pathways Database (E.PATH::database with gs_genes and E.PATH::annotations) that E.PAGE tests the query against) — https://github.com/AhmedMehdiLab/E.PATH
- **R 4.0.3** (Language in which E.PAGE was written and in which the enrichment computation and F.E formula are executed)
- **tidyverse** (Dependency providing the tibble structure of the enrichment results and used to filter on p_adj and assemble the enrichment table)
- **Seurat** (Alternative query source: E.PAGE accepts a Seurat object (with clustering performed) as input and performs differential expression analyses between the clusters)

## Examples

```
library(E.PAGE); input <- process_input_text("ftl ApoE CTSZ", capitalize = TRUE); results <- compute(input); results$stats
```

## Evaluation signals

- 71-DE-gene SCLC query reproduces Cell cycle (fold enrichment 12.98, p_adj 8.49E-07), Cytoprotective mechanism (fold enrichment 5.33, p_adj 7.28E-07→7.28E-08), and Lungs tissue (fold enrichment 12.62) modules in results$stats
- 27-gene cobalt-exposure query recovers the six reported modules (Genotoxic, Non-genotoxic, Carcinogen, Hepatocarcinogens, Liver-based in vitro models, Chemical induced gene expression) each with fold enrichment ~12, p_adj = 1.84E-05, and all 27 query genes per module
- Fold enrichment matches the F.E = (N_gm/n(g))/(N_m^tot/w(U)) ratio of module genes in the query to module genes in the universe background
- All reported p-values are Benjamini-Hochberg FDR-adjusted (p_adj), not raw hypergeometric p-values
- results$stats is a tidyverse tibble; per-category lists split cleanly into CSGE, IGE, DGE and TGE headings

## Limitations

- The curated database covers only four environmental factors (cigarette smoking, diet, infections, toxic chemicals), so queries outside these factors cannot be enriched
- Dataset bias is possible due to limited input data types (gene lists) and similarities between infection and tissue damage-associated immune responses
- Queries of GEO for differential gene expression data were restricted to MeSH terms; joint odds ratio and interaction odds ratio tests are not yet included
- auto_anno() performs enrichment for all gene sets and may take a long time with large databases

## Evidence

- [abstract] Database scale and environmental factors: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [readme] Tool purpose: "This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database."
- [methods] Implementation language: "was written in R version 4.0.313 and developed using RStudio14"
- [methods] Input format for single-cell data: "E.PAGE used a Seurat object (with clustering performed) as an input and performs differential expression analyses between the clusters"
