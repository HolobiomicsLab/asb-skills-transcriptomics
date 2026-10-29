---
name: wgcna-gene-module-detection
description: Use when a cell-type-specific co-expression matrix (e.g., from CS-CORE
  or diff_CSCORE) needs to be partitioned into co-expressed gene modules for downstream
  GO enrichment of disease-relevant pathways.
when_to_use_negative:
- Input is raw UMI counts rather than an inferred co-expression matrix — run CS-CORE/CSCORE
  first, as naive correlation-based modules are confounded by sequencing depth and
  measurement error
- The analysis goal is single gene-pair significance testing rather than module-level
  structure discovery
- Cell types are not subsetted and all cells are pooled — the paper warns this risks
  Simpson's paradox in co-expression inference
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: WGCNA
  role: Builds adjacency.fromSimilarity (power = 1) and TOMsimilarity from the thresholded
    CS-CORE co-expression matrix
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: dynamicTreeCut
  role: cutreeDynamic cuts the average-linkage hierarchical clustering dendrogram
    into modules with minClusterSize = 10
- name: CS-CORE (CSCORE R package)
  role: Supplies the cell-type-specific co-expression estimates and BH-adjusted p-values
    that are the input to module detection
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: clusterProfiler
  role: Performs enrichGO on each extracted module (OrgDb = org.Hs.eg.db, keyType
    = SYMBOL, BH p-value cutoff 0.05)
- name: Seurat
  role: Subsets the scRNA-seq object to a cell type of interest before CSCORE and
    module extraction
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:932b3f467e2bd28a2fb72031599dee59f66649156e5f5c1c11ce1e7d04d3d687
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CS-CORE (CSCORE R package)
  - CSCORE
  - CSCORE (R package)
  - RColorBrewer
  - Seurat
  - WGCNA
  - clusterProfiler
  - diff_CSCORE
  - dynamicTreeCut
  - enrichplot
  - gridExtra
  - p.adjust BH
  - pheatmap
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5263157894736842
n_claims: 19
---

# wgcna-gene-module-detection

## Summary

Extracts co-expressed gene modules from CS-CORE-inferred cell-type-specific co-expression matrices by thresholding non-significant pairs, building a WGCNA adjacency and topological overlap matrix, and cutting the hierarchical clustering tree with dynamicTreeCut.

## When to use

Use when a cell-type-specific co-expression matrix (e.g., from CS-CORE or diff_CSCORE) needs to be partitioned into co-expressed gene modules for downstream GO enrichment of disease-relevant pathways.

## When NOT to use

- Input is raw UMI counts rather than an inferred co-expression matrix — run CS-CORE/CSCORE first, as naive correlation-based modules are confounded by sequencing depth and measurement error
- The analysis goal is single gene-pair significance testing rather than module-level structure discovery
- Cell types are not subsetted and all cells are pooled — the paper warns this risks Simpson's paradox in co-expression inference

## Inputs

- CS-CORE co-expression estimate matrix (genes x genes) with BH-adjusted p-values
- diff_CSCORE differential co-expression matrix (observed difference with permutation p-values)
- Vector of selected genes (e.g., top 1000/5000 by mean relative expression)
- Seurat object subset to a cell type of interest (e.g., CD14 Monocytes from blish_covid.seu.rds)

## Outputs

- Gene module assignments per cell type (cutreeDynamic cluster membership)
- List of co-expressed gene modules suitable for enrichGO input
- Module-level GO enrichment tables (BH-adjusted p-values per GO term)
- ad_coexpression_results.csv compiling modules and enrichment results

## How to apply

First BH-adjust CS-CORE p-values (p.adjust method='BH') and set co-expression entries with adjusted p > 0.05 to zero so only significant co-expressions feed module detection. Build an adjacency with WGCNA::adjacency.fromSimilarity on the absolute co-expression estimates using power = 1 (no soft-thresholding since CS-CORE already yields signed estimates in [-1, 1]), compute TOMsimilarity, run average-linkage hierarchical clustering, and call dynamicTreeCut::cutreeDynamic with minClusterSize = 10 (deepSplit = 2 in the COVID-19 differential module workflow). Apply the same recipe to |difference|/2 matrices from diff_CSCORE to obtain differentially co-expressed modules between groups (e.g., COVID vs Healthy monocytes, AD vs control microglia). Then run clusterProfiler::enrichGO per module (OrgDb = org.Hs.eg.db, keyType = SYMBOL, ont = ALL, pAdjustMethod = BH, universe = selected genes, pvalueCutoff = 0.05).

## Related tools

- **WGCNA** (Builds adjacency.fromSimilarity (power = 1) and TOMsimilarity from the thresholded CS-CORE co-expression matrix) — https://github.com/ChangSuBiostats/CS-CORE
- **dynamicTreeCut** (cutreeDynamic cuts the average-linkage hierarchical clustering dendrogram into modules with minClusterSize = 10)
- **CS-CORE (CSCORE R package)** (Supplies the cell-type-specific co-expression estimates and BH-adjusted p-values that are the input to module detection) — https://github.com/ChangSuBiostats/CS-CORE
- **clusterProfiler** (Performs enrichGO on each extracted module (OrgDb = org.Hs.eg.db, keyType = SYMBOL, BH p-value cutoff 0.05))
- **Seurat** (Subsets the scRNA-seq object to a cell type of interest before CSCORE and module extraction)

## Examples

```
adj = WGCNA::adjacency.fromSimilarity(abs(CSCORE_coexp), power = 1); TOM = WGCNA::TOMsimilarity(adj); memb = dynamicTreeCut::cutreeDynamic(dendro = hclust(hclust_dist, method = 'average'), minClusterSize = 10)
```

## Evaluation signals

- All non-significant co-expression entries (BH-adjusted p > 0.05) must be set to zero before adjacency construction
- Co-expression estimates fed into adjacency.fromSimilarity should lie within [-1, 1]
- Each module from cutreeDynamic contains at least minClusterSize = 10 genes
- Top modules should show at least one GO term with BH-adjusted p < 0.001 and at least 10 enriched GO terms
- The COVID-19 monocyte workflow should recover a module enriched for 'defense response to virus' with BH-adjusted p < 1e-3

## Limitations

- Module quality depends on upstream CS-CORE estimates; existing methods suffered from inflated false positives and biased estimates that would propagate into biased clustering
- CS-CORE assumes gene expressions from cells of the same cell type follow the same distribution, which may not hold across individuals with different genetic, demographic and clinical characteristics
- The measurement model uses only a Poisson distribution; adaptation to other count distributions such as the negative binomial is deferred to future work

## Evidence

- [methods] adjacency power = 1 on thresholded co-expression: "adj = WGCNA::adjacency.fromSimilarity(abs(CSCORE_coexp), power = 1)"
- [methods] cutreeDynamic module extraction: "memb = dynamicTreeCut::cutreeDynamic(dendro = hclust_dist, "
- [methods] threshold non-significant co-expressions before module detection: "CSCORE_coexp[p_matrix_BH > 0.05] <- 0"
- [methods] GO enrichment on modules: "enrichGO(gene = module_list[[i]],"
- [abstract] biased estimates corrupt downstream clustering: "most existing methods suffered from inflated false positives as well as biased co-expression estimates and clustering analysis"
