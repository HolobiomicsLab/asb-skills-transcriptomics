---
name: differential-expression-signature-querying
description: Use when you have a short DE gene signature (e.g. 27–71 genes from GEO/MSigDB
  C7 studies of disease, smoking, or toxic-chemical exposure) and want to identify
  statistically enriched environmental gene modules.
when_to_use_negative:
- Your query is not a gene list — the database is limited to MeSH-term-queried GEO
  differential gene expression data, so SNP, CNV, or DNA methylation inputs are unsupported
- Your question requires joint odds ratio or interaction odds ratio statistics between
  environmental factors — these tests are not yet implemented in E.PAGE
- You need environmental factors beyond cigarette smoking, diet, infections, and toxic
  chemicals — the curated database covers only these four
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: E.PAGE
  role: 'Core R package: processes gene-list input, runs hypergeometric/Fisher enrichment
    against the curated environmental database, and returns FDR-corrected fold enrichment
    statistics'
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: Companion data package supplying the annotations tibble and gene-module database
    (gs_genes, gs_info) used by E.PAGE
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: R 4.0.3
  role: Implementation language for E.PAGE; workflows in the source tasks run the
    package in R 4.0.3 / RStudio with tidyverse
- name: Seurat
  role: E.PAGE accepts a Seurat object (with clustering performed) as input and performs
    differential expression analyses between the clusters
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:3a2122a7da6b760c03eae8740443e3ebb6c9e3900705098cd3088be54b6ceeec
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
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# differential-expression-signature-querying

## Summary

Query a list of differentially expressed genes against the E.PAGE curated environmental-pathways database (237 datasets, 522 gene modules, 25,789 genes across cigarette smoking, diet, infections, and toxic chemicals) using hypergeometric/Fisher exact enrichment with Benjamini-Hochberg FDR correction to predict gene-environment associations.

## When to use

Use when you have a short DE gene signature (e.g. 27–71 genes from GEO/MSigDB C7 studies of disease, smoking, or toxic-chemical exposure) and want to identify statistically enriched environmental gene modules.

## When NOT to use

- Your query is not a gene list — the database is limited to MeSH-term-queried GEO differential gene expression data, so SNP, CNV, or DNA methylation inputs are unsupported
- Your question requires joint odds ratio or interaction odds ratio statistics between environmental factors — these tests are not yet implemented in E.PAGE
- You need environmental factors beyond cigarette smoking, diet, infections, and toxic chemicals — the curated database covers only these four

## Inputs

- List of differentially expressed genes of interest (text string or gene list), e.g. a 71-gene SCLC signature from 23 tumor / 42 healthy control tissues or a 27-gene cobalt-exposure signature from rat liver cell lines
- Seurat object with clustering performed (alternative input via process_input_seurat)
- Optional custom annotation and database .csv files (ex_anno.csv, ex_data.csv format)

## Outputs

- results$stats: tidyverse tibble of statistically enriched gene modules with gene module, number of modules, number of DE genes, p_adj, and fold enrichment (F.E)
- results from compute_sets() as a list with CSGE, IGE, DGE and TGE headings
- Enrichment table exportable to CSV (e.g. sclc_enrichment_table.csv)

## How to apply

Preprocess the gene signature into E.PAGE input with process_input_text (capitalize = TRUE), or extract genes from a Seurat object with process_input_seurat if starting from clustered single-cell data. Run compute() against the default curated database of 237 datasets / 25,789 genes, which tests each gene module against the query using a hypergeometric distribution / Fisher exact test; use compute_sets() if you want results split into CSGE (cigarette smoking), IGE (infection), DGE (diet), and TGE (toxin) headings. Correct module p-values with the Benjamini-Hochberg FDR method and compute fold enrichment as F.E = (N_gm/n(g))/(N_m^tot/w(U)). Filter results on p_adj and inspect the enriched modules in the tidyverse tibble returned in results$stats. For GO-based enrichment, pass an organism database (e.g. org.Hs.eg.db) via the org_db argument; for custom module collections, load annotation and database .csv files with import_annotations() and import_database().

## Related tools

- **E.PAGE** (Core R package: processes gene-list input, runs hypergeometric/Fisher enrichment against the curated environmental database, and returns FDR-corrected fold enrichment statistics) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (Companion data package supplying the annotations tibble and gene-module database (gs_genes, gs_info) used by E.PAGE) — https://github.com/AhmedMehdiLab/E.PATH
- **R 4.0.3** (Implementation language for E.PAGE; workflows in the source tasks run the package in R 4.0.3 / RStudio with tidyverse)
- **Seurat** (E.PAGE accepts a Seurat object (with clustering performed) as input and performs differential expression analyses between the clusters)

## Examples

```
library(E.PAGE); input <- process_input_text("ftl ApoE CTSZ", capitalize=TRUE); results <- compute(input); results$stats
```

## Evaluation signals

- SCLC query (71 DE genes) reproduces Cell cycle (fold enrichment 12.98, p_adj = 8.49E-07), Cytoprotective mechanism (5.33, 7.28E-08), Lungs tissue gene expression (12.62), and Cigarette smoking-related modules including Adenocarcinoma/Tumor vs Non-tumor in Non-smoker vs Smoker (5.82)
- Cobalt-exposure query (27 DE genes) recovers six modules — Genotoxic, Non-genotoxic, Carcinogen, Hepatocarcinogens, Liver-based in vitro models, Chemical induced gene expression — each with fold enrichment ~12 and p_adj = 1.84E-05 with all 27 query genes per module
- p_adj values are present in results$stats, confirming Benjamini-Hochberg multiple-testing correction was applied
- Output is a tidyverse tibble with required columns: gene module, number of modules, number of DE genes, p_adj, fold enrichment
- compute_sets() returns results under exactly four headings: CSGE, IGE, DGE, TGE

## Limitations

- Limited to four types of environmental variables; wider use requires integrating more environmental datasets
- Possibility of dataset bias due to limited input data types (gene list) and similarities between infection- and tissue damage-associated immune responses
- Restricted to MeSH-term queries of the GEO database for differential gene expression data
- Additional statistical tests (joint odds ratio, interaction odds ratio) and genetic data types (SNPs, CNVs, DNA methylations) are not yet covered

## Evidence

- [abstract] curated database scope: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [abstract] testing with DE genes: "The database and statistical analysis package was then tested with the differentially expressed genes obtained from the published literature"
- [methods] implementation: "The statistical analysis package E.PAGE (Environmental Pathways Affecting Gene Expression) (https://github.com/AhmedMehdiLab/E.PAGE) was written in R version 4.0.3"
- [readme] package purpose: "This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database."
- [readme] output format: "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with:"
- [discussion] validation case: "On testing, we found the modules Genotoxicity and Carcinogen to be enriched."
