---
name: differential-gene-expression-analysis
description: Use when you have a Seurat object with graph-based clusters (e.g. severe
  vs healthy COVID-19 bronchoalveolar/peripheral cells) and need DE gene lists plus
  their enriched environmental/gene modules with padj and fold enrichment.
when_to_use_negative:
- Input is unclustered raw single-cell data — process_input_seurat() requires a Seurat
  object with clustering already performed
- The gene–environment categories of interest fall outside the four curated factors
  (cigarette smoking, diet, infections, toxic chemicals), since the database is limited
  to four types of environmental variables
- The data are genetic variant types (SNPs, CNVs, DNA methylations) — these are not
  covered by the current E.PAGE statistical tests
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: E.PAGE
  role: R package whose process_input_seurat() performs the differential expression
    analyses between Seurat clusters and whose compute() runs hypergeometric/Fisher
    enrichment with Benjamini-Hochberg FDR correction
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: Seurat
  role: Used to run the standard pipeline producing graph-based clusters; the clustered
    Seurat object is the required input to E.PAGE
- name: R
  role: Implementation language (E.PAGE written in R 4.0.3, developed in RStudio)
    in which the DE and enrichment steps are run
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: Companion database package providing E.PATH::database (gs_genes) and E.PATH::annotations
    used as the curated gene-module database
  repo: https://github.com/AhmedMehdiLab/E.PATH
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:168d7484cb36ec8a70a70c28bf3bacc3a5149836a3d3633e501ddd6778d3c658
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - E.PATH
  - R
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# differential-gene-expression-analysis

## Summary

Perform cluster-based differential expression on a clustered single-cell (Seurat) object and statistically enrich the resulting DE gene lists against E.PAGE's curated environmental gene-module database (cigarette smoking, diet, infections, toxic chemicals; 522 manually annotated modules, 25,789 genes) using hypergeometric testing with Benjamini-Hochberg correction.

## When to use

Use when you have a Seurat object with graph-based clusters (e.g. severe vs healthy COVID-19 bronchoalveolar/peripheral cells) and need DE gene lists plus their enriched environmental/gene modules with padj and fold enrichment.

## When NOT to use

- Input is unclustered raw single-cell data — process_input_seurat() requires a Seurat object with clustering already performed
- The gene–environment categories of interest fall outside the four curated factors (cigarette smoking, diet, infections, toxic chemicals), since the database is limited to four types of environmental variables
- The data are genetic variant types (SNPs, CNVs, DNA methylations) — these are not covered by the current E.PAGE statistical tests

## Inputs

- Seurat object with clustering already performed (e.g. .rds read via readRDS, such as E.PAGE's ex_seurat.rds)
- Single-cell RNA-seq dataset from disease vs healthy conditions (e.g. COVID-19 severe vs healthy)
- Alternatively, a space-separated list of genes of interest processed via process_input_text()
- Optional: org.Hs.eg.db object for Gene Ontology analysis; custom annotation/database .csv files via import_annotations()/import_database()

## Outputs

- DE gene lists between clusters/severe-vs-healthy groups
- Enriched gene-module table (tidyverse tibble, results$stats) with module name, module size, number of DE genes, padj, fold enrichment and odds ratio
- Category-split results from compute_sets() under CSGE, IGE, DGE and TGE headings
- Collated CSV such as covid19_severe_enriched_modules.csv

## How to apply

Load the single-cell RNA-seq dataset (e.g. COVID-19 severe vs healthy) and run a standard Seurat pipeline to determine graph-based clusters. Pass the clustered Seurat object to E.PAGE's process_input_seurat(), which performs differential expression analyses between the clusters and extracts the DE gene lists. Run E.PAGE's compute() in R (package built on R 4.0.3 with tidyverse/Seurat dependencies) to statistically enrich each gene module via a hypergeometric distribution (Fisher exact test) with Benjamini-Hochberg false discovery rate correction, returning adjusted p-values (padj), fold enrichment and odds ratios as a tidyverse tibble (inspectable via results$stats). Optionally call compute_sets() to obtain results split under CSGE (cigarette smoking), IGE (infection), DGE (diet) and TGE (toxin) headings. Filter on padj and collate module name, module size, number of DE genes, padj and fold enrichment; for GO analysis, supply an org_db (e.g. org.Hs.eg.db) to compute().

## Related tools

- **E.PAGE** (R package whose process_input_seurat() performs the differential expression analyses between Seurat clusters and whose compute() runs hypergeometric/Fisher enrichment with Benjamini-Hochberg FDR correction) — https://github.com/AhmedMehdiLab/E.PAGE
- **Seurat** (Used to run the standard pipeline producing graph-based clusters; the clustered Seurat object is the required input to E.PAGE)
- **R** (Implementation language (E.PAGE written in R 4.0.3, developed in RStudio) in which the DE and enrichment steps are run) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (Companion database package providing E.PATH::database (gs_genes) and E.PATH::annotations used as the curated gene-module database) — https://github.com/AhmedMehdiLab/E.PATH

## Examples

```
seurat_obj <- readRDS(seurat_path); input <- process_input_seurat(seurat_obj, 0); results <- compute(input); results$stats
```

## Evaluation signals

- Output table reproduces the paper's COVID-19 severe-case Table 6: Inflammation (padj 1.18E−60, fold enrichment 3.39), Infection type: Acute (padj 1.02E−56), Immune response (padj 7.61E−49)
- Enrichment statistics carry Benjamini-Hochberg adjusted p-values (padj) alongside fold enrichment and odds ratios
- results$stats is a tidyverse tibble; compute_sets() returns a list with exactly the CSGE, IGE, DGE and TGE headings
- process_input_seurat() was applied to a Seurat object with clustering already performed (clusters pre-computed), not a raw object
- Collated CSV includes module size, number of DE genes, padj and fold enrichment for each enriched module

## Limitations

- Database covers only four environmental variable types (cigarette smoking, diet, infections, toxic chemicals); more environmental datasets need to be integrated
- Curation restricted to MeSH-term queries of the GEO database for differential gene expression data, with possible dataset bias from limited input data types and similarity between infection- and tissue damage-associated immune responses
- Additional statistical tests such as joint odds ratio and interaction odds ratio are not yet included; genetic data (SNPs, CNVs, DNA methylations) not covered
- auto_anno() performs enrichment for all gene sets and may take a long time with large databases

## Evidence

- [methods] E.PAGE used a Seurat object (with clustering performed) as an input and performs differential expression analyses between the clusters: "E.PAGE used a Seurat object (with clustering performed) as an input and performs differential expression analyses between the clusters"
- [abstract] Abstract: testing revealed statistically enriched biological processes: "On testing, we uncovered statistically enriched biological processes, which revealed pathways associated with environmental factors and the genes"
- [readme] README: purpose of the E.PAGE package: "This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database."
- [discussion] Discussion: COVID-19 severe-case genes linked to shared immune modules: "we found that genes differentially expressed in severe cases were linked to gene modules common between bronchoalveolar and peripheral immune environments25,29."
- [methods] Methods: implementation language and dependencies: "The statistical analysis package E.PAGE (Environmental Pathways Affecting Gene Expression) (https://github.com/AhmedMehdiLab/E.PAGE) was written in R version 4.0.3"
