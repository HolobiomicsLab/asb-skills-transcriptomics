---
name: single-cell-rnaseq-clustering
description: Use when a single-cell RNA-seq dataset (e.g. severe COVID-19 vs healthy
  bronchoalveolar/peripheral cells, smokers vs non-smokers airway epithelium atlas)
  needs Seurat graph-based clustering before DE gene extraction for E.PAGE enrichment.
when_to_use_negative:
- Input is already a plain gene list from text or bulk data — use E.PAGE's process_input_text()
  instead of Seurat clustering
- Data are bulk microarray datasets (e.g. GSE12021, GSE55457, GSE55584, GSE55235)
  with no single-cell clusters to define
- The query is for genetic data types (SNPs, CNVs, DNA methylations), which E.PAGE
  does not yet cover
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: E.PAGE
  role: Consumes the clustered Seurat object via process_input_seurat() to run differential
    expression between clusters and hypergeometric/Fisher enrichment with BH FDR correction
    against the curated database
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: Seurat
  role: Performs the graph-based clustering of single-cell RNA-seq data and provides
    the Seurat object input format required by E.PAGE
- name: R 4.0.3
  role: Language in which E.PAGE was written and in which the clustering-to-enrichment
    workflow is executed
- name: tidyverse
  role: Dependency providing the tibble in which statistically enriched annotations
    (results$stats) are returned
merged_aliases:
- single-cell-rna-seq-clustering
merged_alias_records:
- alias: single-cell-rnaseq-differential-expression
  slug: single-cell-rnaseq-differential-expression
  cosine_score: 0.8043
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:a5c6b31e797282bef5cdb79763c1484e7a0ef1970f6de42ebfc941f280d8d1fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - R
  - R 4.0.3
  - Seurat
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# single-cell-rnaseq-clustering

## Summary

Graph-based clustering of single-cell RNA-seq data with Seurat to define cell clusters whose differential expression gene lists (e.g. severe COVID-19 vs healthy, smokers vs non-smokers airway epithelium) serve as input to E.PAGE environmental pathway enrichment.

## When to use

Use when a single-cell RNA-seq dataset (e.g. severe COVID-19 vs healthy bronchoalveolar/peripheral cells, smokers vs non-smokers airway epithelium atlas) needs Seurat graph-based clustering before DE gene extraction for E.PAGE enrichment.

## When NOT to use

- Input is already a plain gene list from text or bulk data — use E.PAGE's process_input_text() instead of Seurat clustering
- Data are bulk microarray datasets (e.g. GSE12021, GSE55457, GSE55584, GSE55235) with no single-cell clusters to define
- The query is for genetic data types (SNPs, CNVs, DNA methylations), which E.PAGE does not yet cover

## Inputs

- Single-cell RNA-seq dataset (e.g. severe vs healthy COVID-19; smokers vs non-smokers airway epithelium atlas) as a Seurat object (.rds, e.g. ex_seurat.rds) with graph-based clustering performed
- Cluster identifier supplied to process_input_seurat() to select the cluster comparison
- E.PATH curated database (25,789 genes across cigarette smoking, diet, infections, toxic chemicals; 522 gene modules)

## Outputs

- Differentially expressed gene lists between Seurat clusters (severe vs healthy, smoker vs non-smoker)
- Enrichment tibble (results$stats) with gene module, number of DE genes, adjusted p-value, fold enrichment and odds ratio
- CSV enrichment table, e.g. covid19_severe_enriched_modules.csv or smoking_enrichment_results.csv

## How to apply

Load the scRNA-seq dataset (e.g. from GEO) as a Seurat object and run a standard Seurat pipeline to determine graph-based clusters, since E.PAGE requires a Seurat object with clustering already performed. Call E.PAGE's process_input_seurat() with the Seurat object and the cluster identifier to compute differential expression between clusters and extract the DE gene list for the comparison of interest (e.g. severe vs healthy). Run compute() in R 4.0.3, which tests each annotated environmental gene module against the query using a Fisher exact / hypergeometric test with Benjamini-Hochberg false discovery rate correction, returning padj, fold enrichment and odds ratios. Inspect results$stats, a tidyverse tibble of statistically enriched annotations, filter on adjusted p-value, and optionally visualise rankings with plot_stats/plot_overlap; per-category results can be obtained via compute_sets() (CSGE, IGE, DGE, TGE headings). Write the collated table (gene module, number of DE genes, padj, fold enrichment) to CSV, e.g. covid19_severe_enriched_modules.csv or smoking_enrichment_results.csv.

## Related tools

- **E.PAGE** (Consumes the clustered Seurat object via process_input_seurat() to run differential expression between clusters and hypergeometric/Fisher enrichment with BH FDR correction against the curated database) — https://github.com/AhmedMehdiLab/E.PAGE
- **Seurat** (Performs the graph-based clustering of single-cell RNA-seq data and provides the Seurat object input format required by E.PAGE)
- **R 4.0.3** (Language in which E.PAGE was written and in which the clustering-to-enrichment workflow is executed)
- **tidyverse** (Dependency providing the tibble in which statistically enriched annotations (results$stats) are returned)

## Examples

```
input <- process_input_seurat(seurat_obj, 0); results <- compute(input); results$stats
```

## Evaluation signals

- COVID-19 severe-case query reproduces the paper's Table 6: top modules Inflammation (padj 1.18E−60, fold enrichment 3.39), Infection type: Acute (padj 1.02E−56), Immune response (padj 7.61E−49)
- Smoking airway epithelium query reproduces Table 7: top modules Epithelial gene expression (padj 3.07E−87), Cigarette smoking, Airway epithelium, Chronic obstructive pulmonary disease
- results$stats is a tidyverse tibble with columns for gene module, number of DE genes, adjusted p-value (Benjamini-Hochberg), fold enrichment and odds ratio
- Adjusted p-values are Benjamini-Hochberg corrected; enrichment uses Fisher exact / hypergeometric distribution
- compute_sets() returns per-category results under CSGE (cigarette smoking), IGE (infection), DGE (diet) and TGE (toxin) headings

## Limitations

- The curated database covers only four environmental factors (cigarette smoking, diet, infections, toxic chemicals), so queries outside these domains will not enrich meaningfully
- The study is limited to MeSH-term queries of the GEO database for differential gene expression data, and possible dataset bias exists due to limited input data types
- Clustering is a prerequisite: process_input_seurat() expects a Seurat object with graph-based clustering already performed, so DE gene lists (and downstream enrichment) depend on cluster quality

## Evidence

- [methods] E.PAGE used a Seurat object (with clustering performed) as an input and performs differential expression analyses between the clusters: "E.PAGE used a Seurat object (with clustering performed) as an input and performs differential expression analyses between the clusters"
- [readme] This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database.: "This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database."
- [readme] Statistically enriched annotations are stored in a tidyverse tibble, and can be viewed with results$stats: "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with:"
- [discussion] genes differentially expressed in severe cases were linked to gene modules common between bronchoalveolar and peripheral immune environments: "we found that genes differentially expressed in severe cases were linked to gene modules common between bronchoalveolar and peripheral immune environments"
