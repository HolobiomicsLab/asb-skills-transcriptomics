---
name: hypergeometric-enrichment-testing
description: Use when you have a list of differentially expressed genes (from text,
  GEO/MSigDB, or a clustered Seurat object) and want to find statistically enriched
  environmental gene modules with adjusted p-values and fold enrichment.
when_to_use_negative:
- Your question requires joint odds ratio or interaction odds ratio statistics — the
  article notes these are not yet included and could be added in future updates
- Your data are SNP, CNV, or DNA methylation data — the tool is limited to gene-list
  expression inputs and these genetic data types are not yet covered
- Your gene list contains no genes present in the E.PATH universe (25,789 genes) or
  you need environmental factors beyond the four annotated types (cigarette smoking,
  diet, infections, toxic chemicals)
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3174
tools:
- name: E.PAGE
  role: R statistical analysis package that performs the hypergeometric/Fisher exact
    enrichment of gene modules against the query and returns padj and fold enrichment
    in results$stats
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: Curated environmental pathways database (annotations tibble and gs_genes list)
    supplying the 522 gene modules tested against the query
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: Seurat
  role: Provides the clustered single-cell object input; E.PAGE performs differential
    expression analyses between Seurat clusters to build the query gene list
- name: tidyverse
  role: Dependency providing the tibble in which statistically enriched annotations
    are stored and manipulated
- name: R 4.0.3
  role: Language/runtime in which E.PAGE was written and executed
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_006
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:406154ac9b9a4cce52fefce92b55b7afc72da1209cad8e28a9a0917614dd78fb
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# hypergeometric-enrichment-testing

## Summary

Over-representation testing of curated environmental gene modules (E.PATH: 522 modules, 25,789 genes across cigarette smoking, diet, infections, toxic chemicals) against a DE gene list, using a hypergeometric/Fisher exact test with Benjamini-Hochberg FDR correction in the E.PAGE R package.

## When to use

Use when you have a list of differentially expressed genes (from text, GEO/MSigDB, or a clustered Seurat object) and want to find statistically enriched environmental gene modules with adjusted p-values and fold enrichment.

## When NOT to use

- Your question requires joint odds ratio or interaction odds ratio statistics — the article notes these are not yet included and could be added in future updates
- Your data are SNP, CNV, or DNA methylation data — the tool is limited to gene-list expression inputs and these genetic data types are not yet covered
- Your gene list contains no genes present in the E.PATH universe (25,789 genes) or you need environmental factors beyond the four annotated types (cigarette smoking, diet, infections, toxic chemicals)

## Inputs

- Gene list of interest as text (e.g. DE genes from a disease signature, such as 291 T1D PBMC microarray genes or 71 SCLC genes)
- Seurat object with graph-based clustering already performed (processed via process_input_seurat)
- Optional custom annotation (.csv) and database (.csv) files, or an org.db (e.g. org.Hs.eg.db) for Gene Ontology analysis
- E.PATH curated database (522 gene modules, 25,789 genes, four environmental factors)

## Outputs

- Enrichment tibble (results$stats) with gene module, number of modules, number of DE genes, adjusted p-value (padj), fold enrichment, and odds ratios
- Category-specific enrichment results from compute_sets() grouped as CSGE (cigarette smoking), IGE (infection), DGE (diet), TGE (toxin)
- CSV export of filtered enriched modules (e.g. t1d_enrichment_table.csv, covid19_severe_enriched_modules.csv)
- Optional visualisations of top modules and overlaps via plot_stats/plot_overlap

## How to apply

Install E.PAGE and E.PATH from GitHub (remotes::install_github) and load the query into R 4.0.3: gene lists from text via process_input_text(capitalize = TRUE), or Seurat objects via process_input_seurat(seurat_obj, cluster_id) which runs differential expression between Seurat clusters. Run compute(input), which tests each of the 522 annotated gene modules against the query using a hypergeometric distribution (Fisher exact test), then corrects p-values with Benjamini-Hochberg false discovery rate. Compute fold enrichment as F.E = (N_gm/n(g))/(N_m^tot/w(U)) — the ratio of module genes in the query to module genes in the universe background. Inspect results$stats (a tidyverse tibble), filter on adjusted p-value, and optionally examine category-specific results with compute_sets (CSGE, IGE, DGE, TGE headings). Validate against published tables, e.g. T1D Immune response padj 1.20E-04 (FE 1.59), COVID-19 severe-case Inflammation padj 1.18E-60 (FE 3.39), or cobalt-exposure Genotoxicity modules with all 27 query genes and p_adj 1.84E-05.

## Related tools

- **E.PAGE** (R statistical analysis package that performs the hypergeometric/Fisher exact enrichment of gene modules against the query and returns padj and fold enrichment in results$stats) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (Curated environmental pathways database (annotations tibble and gs_genes list) supplying the 522 gene modules tested against the query) — https://github.com/AhmedMehdiLab/E.PATH
- **Seurat** (Provides the clustered single-cell object input; E.PAGE performs differential expression analyses between Seurat clusters to build the query gene list)
- **tidyverse** (Dependency providing the tibble in which statistically enriched annotations are stored and manipulated)
- **R 4.0.3** (Language/runtime in which E.PAGE was written and executed)

## Examples

```
library(E.PAGE); input <- process_input_text(genes, capitalize = TRUE); results <- compute(input); results$stats
```

## Evaluation signals

- Reproduced entries match published tables: T1D Immune response padj 1.20E-04 (FE 1.59), Infection type: Acute padj 7.46E-07 (FE 1.61), Cigarette smoking padj 1.57E-04
- Cobalt-exposure query recovers the six reported modules (Genotoxic, Non-genotoxic, Carcinogen, Hepatocarcinogens, Liver-based in vitro models, Chemical induced gene expression) with fold enrichment ~12 and all 27 query genes per module (p_adj 1.84E-05)
- SCLC query recovers Cell cycle (FE 12.98, p_adj 8.49E-07), Cytoprotective mechanism (FE 5.33, p_adj 7.28E-08), and Lungs tissue gene expression (FE 12.62)
- results$stats is a tidyverse tibble containing columns for module name, number of DE genes, adjusted p-value, fold enrichment and odds ratios
- All reported padj values are BH-corrected; enrichment uses every annotated gene module in the E.PATH database (522 modules) as the multiple-testing family

## Limitations

- The database is limited to four types of environmental variables (cigarette smoking, diet, infections, toxic chemicals); wider community usage would require integrating more environmental datasets
- Possible dataset bias due to limited input data types, e.g. gene lists and similarities between infection and tissue damage-associated immune responses
- The study is restricted to MeSH-term queries of the GEO database for differential gene expression data
- Additional statistical tests such as joint odds ratio and interaction odds ratio are not yet included
- Genetic data types (SNPs, CNVs, DNA methylations) are not yet covered
- auto_anno() performs enrichment for all gene sets and may take a long time with large databases

## Evidence

- [methods] Fisher exact test with BH FDR: [evidence span withheld — anchor status: not_found]
- [abstract] Database scope: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [methods] Fold enrichment formula: [evidence span withheld — anchor status: not_found]
- [readme] results$stats tibble: "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with:

``` r
results$stats"
- [methods] Seurat input handling: "E.PAGE used a Seurat object (with clustering performed) as an input and performs differential expression analyses between the clusters"
- [readme] Package purpose: "This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database."
