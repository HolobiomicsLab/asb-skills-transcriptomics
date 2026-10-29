---
name: gene-set-enrichment-analysis
description: Use when you have a list of differentially expressed genes (from a text
  list, microarray study, or clustered Seurat single-cell object) and want to find
  statistically enriched gene modules linked to environmental factors such as cigarette
  smoking, diet, infections, or toxic chemicals.
when_to_use_negative:
- When the query is not a gene list but raw sequencing reads, SNPs, CNVs or DNA methylation
  data — E.PAGE does not yet cover these genetic data types
- When you need environmental associations outside the four curated factors (cigarette
  smoking, diet, infections, toxic chemicals), unless supplying custom annotations/database
- When only differential expression between conditions (not enrichment against a curated
  module set) is the goal — use process_input_seurat/DE tools directly instead
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: E.PAGE
  role: R package performing Fisher exact/hypergeometric enrichment of query genes
    against the curated database with BH FDR correction
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: Curated environmental pathways database (annotations tibble and gs_genes list)
    used as the enrichment background
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: Seurat
  role: Provides clustered single-cell objects from which process_input_seurat extracts
    differential expression gene lists
- name: tidyverse
  role: Dependency used to manipulate and filter the enrichment tibble (results$stats)
    and export result tables
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_005
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:790325a966236b2291302f137af2e10ab819fd0db39d10463ae8aaa690f5f0f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - E.PATH
  - Gene Expression Omnibus (GEO)
  - R
  - R 4.0.3
  - RStudio
  - Seurat
  - dplyr
  - tidyverse
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# gene-set-enrichment-analysis

## Summary

Over-representation enrichment of a query gene list against a curated environmental gene-module database (E.PATH: 4 environmental factors, 522 modules, 25,789 genes), using Fisher exact/hypergeometric tests with Benjamini-Hochberg FDR correction, implemented in the E.PAGE R package.

## When to use

Use when you have a list of differentially expressed genes (from a text list, microarray study, or clustered Seurat single-cell object) and want to find statistically enriched gene modules linked to environmental factors such as cigarette smoking, diet, infections, or toxic chemicals.

## When NOT to use

- When the query is not a gene list but raw sequencing reads, SNPs, CNVs or DNA methylation data — E.PAGE does not yet cover these genetic data types
- When you need environmental associations outside the four curated factors (cigarette smoking, diet, infections, toxic chemicals), unless supplying custom annotations/database
- When only differential expression between conditions (not enrichment against a curated module set) is the goal — use process_input_seurat/DE tools directly instead

## Inputs

- Text gene list of DE genes (e.g. 'ftl ApoE CTSZ'), processed with process_input_text(capitalize = TRUE)
- Clustered Seurat object (single-cell RNA-seq), processed with process_input_seurat(seurat_obj, cluster_id)
- E.PATH curated database: 522 gene modules, 25,789 genes across 4 environmental factors (E.PATH::database and E.PATH::annotations)
- Optional custom annotation (.csv) and database (.csv) files via import_annotations/import_database
- Optional org.Hs.eg.db for Gene Ontology term enrichment

## Outputs

- results$stats: tidyverse tibble of statistically enriched gene modules with adjusted p-value (padj), fold enrichment, and odds ratios
- Per-heading enrichment lists from compute_sets() (CSGE, IGE, DGE, TGE)
- Exportable enrichment tables (e.g. t1d_enrichment_table.csv, sclc_enrichment_table.csv) with gene module, module size, number of DE genes, padj, fold enrichment

## How to apply

Install E.PAGE from GitHub (remotes::install_github("AhmedMehdiLab/E.PAGE")) in R 4.0.3 with tidyverse/Seurat dependencies. Prepare the query via process_input_text(genes, capitalize = TRUE) for a text gene list (e.g. 291 T1D or 229 RA DE genes) or process_input_seurat(seurat_obj, cluster_id) for a clustered Seurat object. Run compute(input), which tests each of the 522 manually annotated gene modules against the query with a Fisher exact test (hypergeometric distribution), corrects p-values with Benjamini-Hochberg FDR, and reports padj, fold enrichment (F.E = (N_gm/n(g))/(N_m^tot/w(U))) and odds ratios in results$stats. Use compute_sets(input) to restrict analysis to the CSGE (cigarette smoking), IGE (infection), DGE (diet) and TGE (toxin) headings, filter results on adjusted p-value, and optionally pass org_db = org.Hs.eg.db for Gene Ontology enrichment or import custom annotations via import_annotations/import_database.

## Related tools

- **E.PAGE** (R package performing Fisher exact/hypergeometric enrichment of query genes against the curated database with BH FDR correction) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (Curated environmental pathways database (annotations tibble and gs_genes list) used as the enrichment background) — https://github.com/AhmedMehdiLab/E.PATH
- **Seurat** (Provides clustered single-cell objects from which process_input_seurat extracts differential expression gene lists)
- **tidyverse** (Dependency used to manipulate and filter the enrichment tibble (results$stats) and export result tables)

## Examples

```
library(E.PAGE); input <- process_input_text("ftl ApoE CTSZ", capitalize = TRUE); results <- compute(input); results$stats
```

## Evaluation signals

- T1D query (291 DE genes): Immune response padj = 1.20E-04 with fold enrichment 1.59; Infection type: Acute padj 7.46E-07, FE 1.61; Insulin resistance padj 1.00E-02
- RA query (229 DE genes): Infection type: Acute padj 8.49E-34 (FE 2.63) and Immune response padj 1.03E-27 (FE 2.87) within rounding tolerance
- SCLC query (71 DE genes): Cell cycle FE 12.98 padj 8.49E-07; Cytoprotective mechanism FE 5.33 padj 7.28E-08; Cigarette smoking-related modules enriched
- Cobalt-exposure query (27 DE genes): Genotoxicity and Carcinogen modules recovered with FE ~12 and padj = 1.84E-05
- results$stats is a tidyverse tibble with padj, fold enrichment and odds ratio columns; top modules match expected Table 6/7 rankings (e.g. Epithelial gene expression padj 3.07E-87)

## Limitations

- The curated database covers only four environmental variable types; wider use requires integration of more environmental datasets
- Study restricted to MeSH-term queries of the GEO database, with possible dataset bias due to limited input data types (e.g. gene lists) and similarities between infection and tissue damage-associated immune responses
- Additional statistical tests (joint odds ratio, interaction odds ratio) and genetic data types (SNPs, CNVs, DNA methylations) are not yet included

## Evidence

- [abstract] The curated database and enrichment tool are available as R packages: "The curated database and enrichment tool are available as R packages at https://github.com/AhmedMehdiLab/E.PATH"
- [abstract] 522 gene modules, 25,789 genes, four environmental factors: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [methods] E.PAGE written in R 4.0.3: "The statistical analysis package E.PAGE (Environmental Pathways Affecting Gene Expression) (https://github.com/AhmedMehdiLab/E.PAGE) was written in R version 4.0.3"
- [methods] Seurat object input and differential expression between clusters: "E.PAGE used a Seurat object (with clustering performed) as an input and performs differential expression analyses between the clusters"
- [readme] README basic usage: "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with:

``` r
results$stats
```"
- [readme] README compute_sets categories: "# result will be a list with CSGE (cigarette smoking), IGE (infection), DGE (diet) and TGE (toxin) headings"
- [discussion] Discussion limitation: [evidence span withheld — anchor status: not_found]
