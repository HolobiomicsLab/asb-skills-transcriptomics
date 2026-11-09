---
name: curated-database-summary-validation
description: Use when a curated database's reported scale statistics (dataset counts,
  module counts, per-factor gene tallies) must be independently reproduced from the
  deposited data files before the database is used downstream.
when_to_use_negative:
- The goal is enrichment analysis on a gene list of interest (E.PAGE compute()) rather
  than auditing the database's own summary statistics
- The database files are custom annotations with a different schema — import may succeed
  but the 237/522/25,789 reference values will not apply
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: E.PATH
  role: Supplies the curated database (database and annotations tibbles) whose summary
    statistics are recomputed and validated
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: E.PAGE
  role: Provides import_database and import_annotations functions to load comma-delimited
    database and annotation .csv files with declared column indices
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: tidyverse
  role: dplyr/tidyverse aggregation verbs used to count unique datasets, gene modules,
    and per-factor gene memberships
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:2a242c89043663fe8d8a7b7ec1e4c9dcae3d867449bc039201b4c1fbd37484e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PATH
  tools:
  - E.PAGE
  - E.PATH
  - R
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# curated-database-summary-validation

## Summary

Validates a curated gene-environment database (E.PATH) by recomputing its headline summary statistics — 237 datasets, 522 manually annotated gene modules, 25,789 associated genes, and per-factor gene counts across cigarette smoking, diet, infections, and toxic chemicals — directly from the deposited data files and comparing them against the published figures.

## When to use

Use when a curated database's reported scale statistics (dataset counts, module counts, per-factor gene tallies) must be independently reproduced from the deposited data files before the database is used downstream.

## When NOT to use

- The goal is enrichment analysis on a gene list of interest (E.PAGE compute()) rather than auditing the database's own summary statistics
- The database files are custom annotations with a different schema — import may succeed but the 237/522/25,789 reference values will not apply

## Inputs

- E.PATH::database list (gs_genes and gs_info) from the installed R package
- E.PATH::annotations tibble mapping gene set names to annotations and sources
- Comma-delimited database .csv file with gene sets and associated genes (imported via import_database)
- Comma-delimited annotation .csv file (imported via import_annotations with content column indices and info column)

## Outputs

- database_summary_stats.csv with one row per reported statistic, the recomputed value, and a per-row match flag
- Per-environmental-factor gene counts for cigarette smoking, diet, infections, and toxic chemicals

## How to apply

Import the E.PATH curated database and gene-module annotation files using E.PAGE's import_database and import_annotations functions, supplying the comma delimiter, header flag, declared content column indices (e.g. c(2, 4)) and the info column index exactly as the file layout dictates. Load E.PATH's database list (gs_genes, gs_info) and annotations tibble directly from the installed package. Use tidyverse/dplyr aggregation verbs to count unique dataset entries and unique gene modules, then compute per-environmental-factor gene counts for cigarette smoking, diet, infections and toxic chemicals from the gene-module membership table. Compare each computed tally against the reported values — 237 datasets, 522 gene modules, 25,789 total genes; 18,015 diet genes, 13,259 infection genes, 3,841 cigarette-smoking genes, 644 toxic-chemical genes — flagging rows that mismatch. Rationale: reproducing these counts from the GitHub-deposited packages (written in R 4.0.3 with tidyverse as a dependency) confirms the curated release matches the published description before enrichment runs.

## Related tools

- **E.PATH** (Supplies the curated database (database and annotations tibbles) whose summary statistics are recomputed and validated) — https://github.com/AhmedMehdiLab/E.PATH
- **E.PAGE** (Provides import_database and import_annotations functions to load comma-delimited database and annotation .csv files with declared column indices) — https://github.com/AhmedMehdiLab/E.PAGE
- **tidyverse** (dplyr/tidyverse aggregation verbs used to count unique datasets, gene modules, and per-factor gene memberships)

## Examples

```
library(E.PAGE); library(E.PATH); data <- import_database(data_path, ",", FALSE, c(2, 4), 0); anno <- import_annotations(anno_path, ",", TRUE, c(2, 4), 5); length(data$gs_genes); length(unique(anno$gs_name))
```

## Evaluation signals

- Unique dataset count in the database equals 237
- Unique manually annotated gene modules equal 522
- Total genes associated with one or more gene modules equals 25,789
- Per-factor gene counts equal 18,015 (diet), 13,259 (infection), 3,841 (cigarette smoking), and 644 (toxic chemicals)
- database_summary_stats.csv contains a match flag per row and every row is TRUE when the deposited data is the published release

## Limitations

- The curated database covers only four environmental variable types; statistics for other environmental factors cannot be validated
- The search was limited to MeSH-term queries of GEO, so dataset counts may not generalize beyond that retrieval strategy
- The gs_info tibble is included only for compatibility and does not contain information, so info-based counts must come from the annotation file

## Evidence

- [abstract] 237 datasets and 522 gene modules: "a total of 237 datasets were selected and 522 gene modules were manually annotated"
- [abstract] four factors and gene total: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [methods] tidyverse dependency: "Using publicly available packages tidyverse15 , Seurat16 as dependencies"
- [readme] import_database signature: "data <- import_database(data_path, ",", FALSE, c(2, 4), 0)"
- [readme] gs_info compatibility note: "It is included 
for compatibility with other packages, and does not contain any information."
