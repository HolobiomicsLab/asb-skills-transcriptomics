---
name: per-factor-gene-count-aggregation
description: Use when a workflow must reproduce or audit the composition statistics
  of the E.PATH database — e.g. count unique gene sets, annotated modules, or genes
  assigned to each environmental factor from gs_genes/gs_info and annotations files.
when_to_use_negative:
- The goal is running enrichment on a user gene list (use E.PAGE's process_input_text/process_input_seurat
  and compute, not count aggregation)
- The database contains environmental factors beyond the four curated in this study
  (cigarette smoking, diet, infections, toxic chemicals) — the counts and match flags
  only apply to these categories
- Input is raw differential-expression data (e.g. GEO/MSigDB C7 downloads) that has
  not yet been curated into gene-set membership files
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: E.PATH
  role: R data package providing the curated database (gs_genes, gs_info) and annotations
    tibble whose membership rows are aggregated into per-factor gene counts
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: E.PAGE
  role: Supplies import_database and import_annotations functions to load the comma-delimited
    database and annotation CSV files with declared content column indices
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: tidyverse
  role: Dependency used for dplyr/tidyverse aggregation verbs to count unique gene
    sets, modules and per-factor genes
- name: R 4.0.3
  role: Runtime in which the aggregation and comparison script is executed (developed
    in RStudio)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:c30ea254e96a618d7e77b338c609b44eb799011f870dd78c358d0c7d375df551
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PATH
  tools:
  - E.PAGE
  - E.PATH
  - R
  - R 4.0.3
  - tidyverse
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# per-factor-gene-count-aggregation

## Summary

Aggregates gene-set membership data from the E.PATH curated database into per-environmental-factor gene counts (cigarette smoking, diet, infections, toxic chemicals) and validates them against the paper's reported totals (18,015 diet, 13,259 infection, 3,841 smoking, 644 toxic-chemical genes; 25,789 unique genes overall).

## When to use

Use when a workflow must reproduce or audit the composition statistics of the E.PATH database — e.g. count unique gene sets, annotated modules, or genes assigned to each environmental factor from gs_genes/gs_info and annotations files.

## When NOT to use

- The goal is running enrichment on a user gene list (use E.PAGE's process_input_text/process_input_seurat and compute, not count aggregation)
- The database contains environmental factors beyond the four curated in this study (cigarette smoking, diet, infections, toxic chemicals) — the counts and match flags only apply to these categories
- Input is raw differential-expression data (e.g. GEO/MSigDB C7 downloads) that has not yet been curated into gene-set membership files

## Inputs

- E.PATH::database list (gs_genes mapping gene set names to genes, gs_info tibble) from github.com/AhmedMehdiLab/E.PATH
- E.PATH::annotations tibble mapping gene set names to annotations and sources
- Comma-delimited database and annotation .csv files imported via import_database/import_annotations with declared column indices

## Outputs

- Per-environmental-factor gene count table (diet, infections, cigarette smoking, toxic chemicals)
- Computed-vs-reported summary table with per-row match flag (e.g. database_summary_stats.csv) covering 237 datasets, 522 gene modules, 25,789 genes

## How to apply

Import the curated database and annotation CSV files using E.PAGE's import_database and import_annotations functions, declaring the delimiter (comma), header flag, and content column indices (e.g. c(2,4) with info column 5/0 as in the README examples). The E.PATH package exposes the database as a list with gs_genes (gene-set-name to gene mapping) and gs_info, and annotations as a tibble mapping gene set names to annotations. Use tidyverse/dplyr aggregation verbs to count unique gene sets, unique manually annotated gene modules, and unique genes per environmental factor (diet, infections, cigarette smoking, toxic chemicals). Compare computed counts against reported values (237 datasets, 522 gene modules, 25,789 genes; per-factor counts above) and write a computed-vs-reported summary table with per-row match flags. A gene may belong to modules under multiple factors, so per-factor counts overlap and must not be summed to the unique-gene total.

## Related tools

- **E.PATH** (R data package providing the curated database (gs_genes, gs_info) and annotations tibble whose membership rows are aggregated into per-factor gene counts) — https://github.com/AhmedMehdiLab/E.PATH
- **E.PAGE** (Supplies import_database and import_annotations functions to load the comma-delimited database and annotation CSV files with declared content column indices) — https://github.com/AhmedMehdiLab/E.PAGE
- **tidyverse** (Dependency used for dplyr/tidyverse aggregation verbs to count unique gene sets, modules and per-factor genes)
- **R 4.0.3** (Runtime in which the aggregation and comparison script is executed (developed in RStudio))

## Examples

```
data <- import_database(data_path, ",", FALSE, c(2, 4), 0); length(data$gs_genes); length(unique(unlist(data$gs_genes)))
```

## Evaluation signals

- Unique gene-set count in gs_genes equals 237 datasets as reported
- Unique annotated gene-set names in the annotations tibble equal 522 gene modules
- Unique genes across all gs_genes mappings equal 25,789
- Per-factor counts match reported values: 18,015 diet, 13,259 infection, 3,841 cigarette smoking, 644 toxic chemicals
- Computed-vs-reported summary table has a match flag of TRUE on every row

## Limitations

- Only four environmental variable types are included, so per-factor counts cover only cigarette smoking, diet, infections and toxic chemicals
- Per-factor gene counts overlap (a gene can appear in modules of multiple factors); they must not be summed to reproduce the 25,789 unique-gene total
- Curated data were restricted to MeSH-term queries of GEO, so counts reflect only differential gene expression data of that scope

## Evidence

- [abstract] Curated database composition: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [abstract] Screening scale to reproduce: "After title and abstract screening against pre-set criteria, a total of 237 datasets were selected and 522 gene modules were manually annotated"
- [readme] Package data structures being aggregated: "`E.PATH::annotations` is a `tibble` which maps gene set names to annotations and
sources."
- [readme] Import path for database and annotation files: "anno <- import_annotations(anno_path, ",", TRUE, c(2, 4), 5)
data <- import_database(data_path, ",", FALSE, c(2, 4), 0)"
- [methods] tidyverse as the aggregation dependency: "Using publicly available packages tidyverse15 , Seurat16 as dependencies"
