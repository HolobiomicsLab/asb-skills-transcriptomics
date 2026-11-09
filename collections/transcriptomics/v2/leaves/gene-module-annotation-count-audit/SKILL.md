---
name: gene-module-annotation-count-audit
description: Use when you have the E.PATH GitHub data files (database and annotations)
  and need to verify or recompute the reported dataset, gene-module, gene-count, and
  per-environmental-factor statistics before downstream enrichment work.
when_to_use_negative:
- You need to run enrichment on a gene list of interest (use E.PAGE's process_input_text/process_input_seurat
  + compute instead of auditing counts)
- Your goal is annotating new databases or auto-generating GO/KEGG/MeSH annotations
  via auto_anno, rather than verifying existing counts
- The environmental factor of interest is outside the four curated categories (cigarette
  smoking, diet, infections, toxic chemicals) — the article notes the study is limited
  to four types of environmental variables
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: E.PAGE
  role: R package supplying import_annotations and import_database functions to load
    comma-delimited database and annotation files for counting
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: R package providing the curated database object (gs_genes, gs_info) and annotations
    tibble whose contents are counted and audited
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: R
  role: Runtime for the audit script (article used R version 4.0.3 developed in RStudio)
- name: tidyverse
  role: Dependency providing dplyr/tidyverse aggregation verbs used to count unique
    datasets, gene modules, and per-factor genes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:0056fcfc62cd6c268f2b639981b2237b0b3889b79d2f07e4a309144b49564378
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - E.PATH
  - R
  - tidyverse
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gene-module-annotation-count-audit

## Summary

Reproduces the curated E.PATH database's reported scale and composition (237 datasets, 522 gene modules, 25,789 genes across cigarette smoking, diet, infections and toxic chemicals) by importing the GitHub-deposited data files with E.PAGE's import functions and auditing computed counts against the paper's reported figures.

## When to use

Use when you have the E.PATH GitHub data files (database and annotations) and need to verify or recompute the reported dataset, gene-module, gene-count, and per-environmental-factor statistics before downstream enrichment work.

## When NOT to use

- You need to run enrichment on a gene list of interest (use E.PAGE's process_input_text/process_input_seurat + compute instead of auditing counts)
- Your goal is annotating new databases or auto-generating GO/KEGG/MeSH annotations via auto_anno, rather than verifying existing counts
- The environmental factor of interest is outside the four curated categories (cigarette smoking, diet, infections, toxic chemicals) — the article notes the study is limited to four types of environmental variables

## Inputs

- E.PATH::annotations tibble (gene set names mapped to annotations and sources) from the GitHub-deposited package
- E.PATH::database list (gs_genes gene-set-to-gene mapping and gs_info tibble) from the GitHub-deposited package
- Custom comma-delimited database .csv and annotations .csv files (imported via import_database / import_annotations with declared content column indices and info column)

## Outputs

- database_summary_stats.csv — computed-vs-reported summary table with one row per statistic (datasets, gene modules, total genes, per-factor gene counts) and a per-row match flag

## How to apply

Install E.PAGE and E.PATH from GitHub (remotes::install_github) and load the deposited data: E.PATH::annotations (a tibble mapping gene set names to annotations/sources) and E.PATH::database (a list with gs_genes and gs_info). For custom comma-delimited files, call import_annotations(anno_path, ',', TRUE, c(2,4), 5) and import_database(data_path, ',', FALSE, c(2,4), 0), matching declared content column indices and info columns. Use dplyr/tidyverse aggregation verbs to count unique dataset entries in the database and unique manually annotated gene modules, then compute per-factor gene counts from the gene-module membership table for cigarette smoking, diet, infections and toxic chemicals. Compare computed counts to the reported values (237 datasets, 522 gene modules, 25,789 genes; 18,015 diet genes, 13,259 infection genes, 3,841 cigarette-smoking genes, 644 toxic-chemical genes) and emit a computed-vs-reported table with a per-row match flag.

## Related tools

- **E.PAGE** (R package supplying import_annotations and import_database functions to load comma-delimited database and annotation files for counting) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (R package providing the curated database object (gs_genes, gs_info) and annotations tibble whose contents are counted and audited) — https://github.com/AhmedMehdiLab/E.PATH
- **R** (Runtime for the audit script (article used R version 4.0.3 developed in RStudio))
- **tidyverse** (Dependency providing dplyr/tidyverse aggregation verbs used to count unique datasets, gene modules, and per-factor genes)

## Examples

```
library(E.PATH); data <- E.PATH::database$gs_genes; anno <- E.PATH::annotations; total_genes <- length(unique(unlist(data))); n_modules <- length(unique(names(data)))
```

## Evaluation signals

- Unique dataset count in the database equals 237, matching the number of datasets selected after title/abstract screening
- Unique manually annotated gene module count equals 522
- Total genes associated with one or more gene modules equals 25,789
- Per-factor gene counts match reported values: diet 18,015, infections 13,259, cigarette smoking 3,841, toxic chemicals 644
- database_summary_stats.csv contains exactly one row per reported statistic, each with a computed value, reported value, and TRUE match flag

## Limitations

- The curated database covers only four environmental variable types (cigarette smoking, diet, infections, toxic chemicals); more environmental datasets would be needed for wider use
- Possible dataset bias due to limited input data types such as gene lists, and similarities between infection- and tissue damage-associated immune responses
- The source study was restricted to MeSH-term queries of the GEO database for differential gene expression data
- gs_info in E.PATH::database 'is included for compatibility with other packages, and does not contain any information' — do not rely on it for counts

## Evidence

- [abstract] Database scale and environmental factors: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [abstract] Screening yields dataset and module counts: "After title and abstract screening against pre-set criteria, a total of 237 datasets were selected and 522 gene modules were manually annotated"
- [readme] Import function parameters for custom files: "anno <- import_annotations(anno_path, ",", TRUE, c(2, 4), 5)
data <- import_database(data_path, ",", FALSE, c(2, 4), 0)"
- [readme] Package data structure to audit: "`E.PATH::database` is a `list` containing `gs_genes` and `gs_info`.
* `gs_genes` is a `list` mapping gene set names to individual genes."
- [methods] R environment used for the package: "was written in R version 4.0.313 and developed using RStudio14"
