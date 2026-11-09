---
name: tidyverse-tabular-data-wrangling
description: Use when a task requires counting, filtering, or joining the E.PATH curated
  database tables (gs_genes, annotations tibbles) or intersecting E.PAGE enrichment
  results (results$stats) across query gene lists in R.
when_to_use_negative:
- The analysis requires gene-set enrichment statistics (odds ratios, padj) rather
  than reshaping/joining tables — use E.PAGE's compute() instead of manual tidyverse
  wrangling
- Input is single-cell expression data needing clustering and differential expression
  — feed a Seurat object to process_input_seurat() rather than deconstructing it into
  tables manually
- The goal is building new gene-environment databases or GO/KEGG/MeSH annotation generation
  — use auto_anno() rather than hand-rolled tidyverse aggregation
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3307
tools:
- name: tidyverse
  role: Dependency of E.PAGE providing dplyr/tibble/readr verbs for importing, aggregating,
    joining, and exporting the database and enrichment tibbles
- name: E.PAGE
  role: R package whose import_annotations/import_database functions load comma-delimited
    database files and whose compute() results$stats tibbles are the objects wrangled
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: R package supplying the curated database (gs_genes list and annotations tibble)
    whose tables are counted and aggregated
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: R
  role: Runtime environment (E.PAGE written in R version 4.0.3) in which all dplyr/tidyverse
    wrangling is executed
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:813817e1801b97e702faf45628cf3716aa0d8a901c964e4c423e11c236a669d0
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tidyverse-tabular-data-wrangling

## Summary

Wrangle E.PATH/E.PAGE tabular data (gene set-gene memberships, annotations, enrichment tibbles) in R using tidyverse/dplyr verbs — importing comma-delimited database files, aggregating counts, and joining per-disease enrichment results.

## When to use

Use when a task requires counting, filtering, or joining the E.PATH curated database tables (gs_genes, annotations tibbles) or intersecting E.PAGE enrichment results (results$stats) across query gene lists in R.

## When NOT to use

- The analysis requires gene-set enrichment statistics (odds ratios, padj) rather than reshaping/joining tables — use E.PAGE's compute() instead of manual tidyverse wrangling
- Input is single-cell expression data needing clustering and differential expression — feed a Seurat object to process_input_seurat() rather than deconstructing it into tables manually
- The goal is building new gene-environment databases or GO/KEGG/MeSH annotation generation — use auto_anno() rather than hand-rolled tidyverse aggregation

## Inputs

- E.PATH::database (list of gs_genes and gs_info) and E.PATH::annotations tibble from the GitHub-deposited package
- Comma-delimited annotation file (.csv) with declared content column indices and info column (e.g. c(2,4) content, column 5 info)
- Comma-delimited database file (.csv) mapping gene set names to genes (e.g. c(2,4) content, no header)
- E.PAGE results$stats enrichment tibbles computed per query gene list (e.g. 291 T1D DE genes, 229 RA DE genes) with padj and fold-enrichment columns

## Outputs

- database_summary_stats.csv: computed-vs-reported count table (237 datasets, 522 modules, 25,789 genes; 18,015 diet, 13,259 infection, 3,841 cigarette-smoking, 644 toxic-chemical genes) with per-row match flags
- shared_modules.csv: tidy table of gene modules significant in both queries (expected: Interferons, IL-12, Transcription regulation, Immune response) with padj and fold enrichment per disease
- Per-environmental-factor gene count tables derived from the gene-module membership list

## How to apply

Install E.PAGE and E.PATH from GitHub and load the bundled data: E.PATH::annotations (a tibble mapping gene set names to annotations/sources) and E.PATH::database (a list with gs_genes and gs_info). For custom files, import comma-delimited data with E.PAGE's import_annotations(path, ',', TRUE, c(2,4), 5) and import_database(path, ',', FALSE, c(2,4), 0), specifying the separator, header flag, content column indices, and info column. Aggregate with dplyr verbs: n_distinct on dataset entries and gene modules, group-by counts of genes per environmental factor (cigarette smoking, diet, infections, toxic chemicals). To find modules shared between two queries (e.g. T1D's 291 DE genes and RA's 229 DE genes), filter each results$stats tibble on the Benjamini-Hochberg padj and inner_join on gene-module names before writing CSV outputs with readr.

## Related tools

- **tidyverse** (Dependency of E.PAGE providing dplyr/tibble/readr verbs for importing, aggregating, joining, and exporting the database and enrichment tibbles)
- **E.PAGE** (R package whose import_annotations/import_database functions load comma-delimited database files and whose compute() results$stats tibbles are the objects wrangled) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (R package supplying the curated database (gs_genes list and annotations tibble) whose tables are counted and aggregated) — https://github.com/AhmedMehdiLab/E.PATH
- **R** (Runtime environment (E.PAGE written in R version 4.0.3) in which all dplyr/tidyverse wrangling is executed)

## Examples

```
anno <- import_annotations(anno_path, ",", TRUE, c(2, 4), 5); data <- import_database(data_path, ",", FALSE, c(2, 4), 0); results <- compute(input, anno, data); results$stats
```

## Evaluation signals

- Unique-gene count across gs_genes must equal 25,789 genes, and unique gene modules must equal 522; any mismatch indicates a bad import or wrong column indices
- Per-factor gene counts must reproduce the reported values: 18,015 diet, 13,259 infection, 3,841 cigarette smoking, and 644 toxic chemical genes
- Unique dataset entries must total 237, matching the number of datasets selected after title and abstract screening
- The shared-modules inner_join output must contain Interferons, IL-12, Transcription regulation, and Immune response, each with non-missing padj and fold-enrichment values from both the T1D and RA queries
- Only gene modules with significant Benjamini-Hochberg FDR-adjusted p-values (padj) should enter the shared set; insignificant modules must be excluded before joining

## Limitations

- The curated E.PATH database covers only four environmental factors (cigarette smoking, diet, infections, toxic chemicals), so wrangled counts cannot extend beyond these categories
- The gs_info tibble is included only for compatibility with other packages and does not contain any information — do not rely on it for joins or metadata
- auto_anno()-generated annotations perform enrichment for all gene sets and may take a long time with large databases
- E.PAGE queries are restricted to the MeSH-term-derived GEO data used to build the database, limiting the data types the wrangled tables can represent

## Evidence

- [readme] Statistically enriched annotations are stored in a tidyverse tibble: "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with"
- [readme] import_annotations example call: "anno <- import_annotations(anno_path, ",", TRUE, c(2, 4), 5)
data <- import_database(data_path, ",", FALSE, c(2, 4), 0)"
- [methods] tidyverse as declared dependency: "Using publicly available packages tidyverse15 , Seurat16 as dependencies"
- [abstract] database composition statistics: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [abstract] dataset and module counts: "a total of 237 datasets were selected and 522 gene modules were manually annotated"
