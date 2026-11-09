---
name: seurat-object-cluster-processing
description: Use when you have a Seurat object with graph-based clustering already
  performed and need to extract a cluster-specific differential expression gene list
  for enrichment analysis with E.PAGE.
when_to_use_negative:
- You already have a plain gene list from text/abstract sources — use process_input_text(genes,
  capitalize=TRUE) instead of the Seurat path.
- The Seurat object has no clustering performed — E.PAGE requires graph-based clusters
  to compute differential expression between clusters.
- You need joint odds ratio, interaction odds ratio, or SNP/CNV/DNA methylation data
  types — these are not yet supported per the article's discussion.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: E.PAGE
  role: Provides process_input_seurat() to compute differential expression between
    Seurat clusters and derive the enrichment query gene list; compute() then runs
    Fisher exact/hypergeometric testing with Benjamini-Hochberg correction.
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: Seurat
  role: Dependency; supplies the clustered single-cell object format (Seurat object
    with clustering performed) accepted by process_input_seurat.
- name: tidyverse
  role: Dependency; results$stats enriched-annotation table is a tidyverse tibble.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:610676d2a2ba77b463c5032e73c165788fe3c157e023c07669622961df55e7ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - R 4.0.3
  - Seurat
  - tidyverse
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# seurat-object-cluster-processing

## Summary

Process a clustered Seurat single-cell object through E.PAGE's process_input_seurat, which computes differential expression between Seurat clusters to derive the query gene list for environmental pathway enrichment. This converts a scRNA-seq atlas (e.g. smokers vs non-smokers airway epithelium) into input for the E.PAGE enrichment engine.

## When to use

Use when you have a Seurat object with graph-based clustering already performed and need to extract a cluster-specific differential expression gene list for enrichment analysis with E.PAGE.

## When NOT to use

- You already have a plain gene list from text/abstract sources — use process_input_text(genes, capitalize=TRUE) instead of the Seurat path.
- The Seurat object has no clustering performed — E.PAGE requires graph-based clusters to compute differential expression between clusters.
- You need joint odds ratio, interaction odds ratio, or SNP/CNV/DNA methylation data types — these are not yet supported per the article's discussion.

## Inputs

- Seurat object (.rds, e.g. readRDS of an extdata exemplar) with graph-based clustering already performed
- Integer cluster identifier (Seurat cluster ID) for the cell population of interest
- Optionally an org.*.eg.db annotation package for Gene Ontology analysis

## Outputs

- Processed input object consumed by E.PAGE compute()
- results$stats tidyverse tibble of enriched annotations (gene module, DE gene count, adjusted p-value, fold enrichment, odds ratio)
- results from compute_sets() split into CSGE (cigarette smoking), IGE (infection), DGE (diet), TGE (toxin) headings

## How to apply

Load the scRNA-seq dataset as a Seurat object with graph-based clustering already performed (E.PAGE requires clustering as a prerequisite and does not re-cluster). Call process_input_seurat(seurat_obj, cluster_id), supplying the object and the integer cluster identifier of interest; the function performs differential expression analyses between Seurat clusters to obtain the query gene list. Feed the resulting input object to compute(), which enriches each annotated gene module against the curated database using a Fisher exact/hypergeometric test with Benjamini-Hochberg FDR correction. Inspect results$stats — a tidyverse tibble filtered on adjusted p-value — and optionally visualise rankings with plot_stats/plot_overlap. E.PAGE depends on tidyverse and Seurat and was developed in R 4.0.3 / RStudio; Gene Ontology analysis can be layered on by supplying org_db = org.Hs.eg.db to compute().

## Related tools

- **E.PAGE** (Provides process_input_seurat() to compute differential expression between Seurat clusters and derive the enrichment query gene list; compute() then runs Fisher exact/hypergeometric testing with Benjamini-Hochberg correction.) — https://github.com/AhmedMehdiLab/E.PAGE
- **Seurat** (Dependency; supplies the clustered single-cell object format (Seurat object with clustering performed) accepted by process_input_seurat.)
- **tidyverse** (Dependency; results$stats enriched-annotation table is a tidyverse tibble.)

## Examples

```
library(E.PAGE); seurat_obj <- readRDS(system.file("extdata", "ex_seurat.rds", package="E.PAGE")); input <- process_input_seurat(seurat_obj, 0); results <- compute(input); results$stats
```

## Evaluation signals

- results$stats is a tidyverse tibble and contains statistically enriched annotations filtered on adjusted p-value.
- For the smoking airway epithelium single-cell query, top-ranked modules must include Epithelial gene expression, Cigarette smoking, Airway epithelium, and Chronic obstructive pulmonary disease, matching Table 7 (e.g. Epithelial gene expression padj 3.07E−87).
- compute_sets() returns a list with exactly the CSGE (cigarette smoking), IGE (infection), DGE (diet) and TGE (toxin) headings.
- Benjamini-Hochberg-adjusted p-values, fold enrichment, and odds ratios are present in the enrichment table before export to CSV.
- R CMD check badge workflows pass for the installed E.PAGE package (R 4.0.3 with Seurat and tidyverse dependencies).

## Limitations

- E.PAGE used a Seurat object as input and requires that clustering already be performed; it performs differential expression between existing clusters rather than re-clustering.
- The curated database covers only four environmental factors (cigarette smoking, diet, infections, toxic chemicals); more environmental datasets need to be integrated.
- Possible dataset bias due to limited input data types, e.g. similarities between infection and tissue damage-associated immune responses.
- Additional statistical tests such as joint odds ratio and interaction odds ratio are not yet included; genetic data (SNPs, CNVs, DNA methylations) are not covered.

## Evidence

- [methods] E.PAGE used a Seurat object (with clustering performed) as an input and performs differential expression analyses between the clusters: "E.PAGE used a Seurat object (with clustering performed) as an input and performs differential expression analyses between the clusters"
- [readme] input from Seurat in README: "seurat_obj <- readRDS(seurat_path)
input <- process_input_seurat(seurat_obj, 0)"
- [readme] results storage in README: "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with:

``` r
results$stats"
- [methods] dependencies in methods: "Using publicly available packages tidyverse15 , Seurat16 as dependencies"
- [discussion] smoking scRNA-seq finding: "we found gene modules for Cigarette smoking, Airway epithelium, Epithelial gene expression, and Chronic obstructive pulmonary disease."
