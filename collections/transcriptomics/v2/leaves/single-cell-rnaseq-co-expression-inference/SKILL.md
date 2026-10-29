---
name: single-cell-rnaseq-co-expression-inference
description: Use when you have raw UMI count matrices from scRNA-seq (e.g., a Seurat
  object like blish_covid.seu.rds) and need unbiased cell-type-specific co-expression
  networks, or donor-level differential co-expression between conditions such as COVID-19
  vs. healthy.
when_to_use_negative:
- When the analysis goal is cell-level (per-cell) co-expression networks rather than
  per-cell-type networks — locCSN-style local networks would be more appropriate
- When input data are already normalized/corrected expression matrices rather than
  raw UMI counts, since CS-CORE's measurement model is built on UMI count distributions
- When cells from one cell type do not share a common expression distribution across
  individuals with different genetic, demographic, or clinical characteristics — an
  assumption CS-CORE explicitly states
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: CSCORE (R package)
  role: Core estimator/tester of cell-type-specific co-expression and differential
    co-expression (CSCORE() and diff_CSCORE() functions)
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: Seurat
  role: Loading the Seurat object, subsetting cells to a cell type (e.g., B cells,
    CD14 Monocytes), and computing mean expression for gene selection
- name: WGCNA
  role: 'Module extraction: adjacency.fromSimilarity on abs(co-expression) with power=1,
    TOMsimilarity, hierarchical clustering'
- name: dynamicTreeCut
  role: cutreeDynamic on the TOM dendrogram (deepSplit=2, minClusterSize=10) to assign
    genes to modules
- name: clusterProfiler
  role: enrichGO functional enrichment of modules (OrgDb='org.Hs.eg.db', keyType='SYMBOL',
    ont='ALL')
- name: enrichplot
  role: dotplot visualization of enriched GO terms for selected modules
- name: pheatmap
  role: Heatmap visualization of Healthy-vs-COVID co-expression for module genes
- name: locCSN
  role: Alternative method referenced for cell-specific (per-cell) network estimation,
    not used in the CS-CORE workflow
  repo: https://github.com/xuranw/locCSN
- name: normalisr
  role: Alternative normalization and co-expression association testing framework
    referenced as a competing method
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:973bedca19f0c50512ffe560c92336a74906f12b58cdda8b7c5bc223726db245
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CSCORE (R package)
  - RColorBrewer
  - Seurat
  - WGCNA
  - clusterProfiler
  - dynamicTreeCut
  - enrichplot
  - gridExtra
  - locCSN
  - normalisr
  - pheatmap
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 36
---

# single-cell-rnaseq-co-expression-inference

## Summary

Estimate and test cell-type-specific gene co-expressions from scRNA-seq UMI count data using CS-CORE, which models sequencing depth variation and measurement error via a latent-variable measurement model and an iteratively re-weighted least squares (IRLS) correlation estimator, then extracts co-expressed modules and tests differential co-expression.

## When to use

Use when you have raw UMI count matrices from scRNA-seq (e.g., a Seurat object like blish_covid.seu.rds) and need unbiased cell-type-specific co-expression networks, or donor-level differential co-expression between conditions such as COVID-19 vs. healthy.

## When NOT to use

- When the analysis goal is cell-level (per-cell) co-expression networks rather than per-cell-type networks — locCSN-style local networks would be more appropriate
- When input data are already normalized/corrected expression matrices rather than raw UMI counts, since CS-CORE's measurement model is built on UMI count distributions
- When cells from one cell type do not share a common expression distribution across individuals with different genetic, demographic, or clinical characteristics — an assumption CS-CORE explicitly states

## Inputs

- Seurat object with UMI count matrix, cell-type labels (e.g., cell.type.coarse), donor labels, and condition labels (e.g., blish_covid.seu.rds from Wilk et al. PBMC COVID dataset)
- Gene selection: top N (5000 or 1000) highly expressed genes by mean relative expression
- Covariate metadata columns (e.g., percent.mt, Sex) for covariate-adjusted inference

## Outputs

- Symmetric co-expression (correlation) estimate matrix with BH-adjusted p-values, non-significant entries set to 0, estimates bounded in [-1, 1]
- Differential co-expression difference matrix with donor-level permutation p-values (diff_CSCORE)
- Co-expressed / differentially co-expressed gene modules from WGCNA TOM clustering
- GO enrichment results per module (enrichGO) and dotplot/pheatmap visualizations

## How to apply

Load the Seurat object and subset cells to the cell type of interest (e.g., CD14 Monocytes, pbmc$cell.type.coarse), selecting highly expressed genes (e.g., top 5000, or top 1000 by mean relative expression in the COVID workflow) to limit the test set. Run CSCORE() on the per-condition subset — or diff_CSCORE() with group_label='Status', sample_label='Donor', group_levels=c('Healthy','COVID'), n_permu=100, seed=2024 — which accounts for sequencing depth and measurement error via an IRLS approach and, for the differential case, shuffles donor-level labels for permutation p-values. BH-adjust p-values (threshold 0.05) and set non-significant co-expressions to 0 in the estimate matrix. Build modules with WGCNA (adjacency.fromSimilarity on abs(co-exp) or |difference|/2 with power=1, TOMsimilarity, average-linkage hclust, dynamicTreeCut::cutreeDynamic with deepSplit=2, minClusterSize=10), then annotate modules with clusterProfiler::enrichGO (OrgDb='org.Hs.eg.db', keyType='SYMBOL', ont='ALL') and visualize with enrichplot::dotplot and pheatmap. Covariates such as 'percent.mt' and 'Sex' can be adjusted via CSCORE's covariate_names argument.

## Related tools

- **CSCORE (R package)** (Core estimator/tester of cell-type-specific co-expression and differential co-expression (CSCORE() and diff_CSCORE() functions)) — https://github.com/ChangSuBiostats/CS-CORE
- **Seurat** (Loading the Seurat object, subsetting cells to a cell type (e.g., B cells, CD14 Monocytes), and computing mean expression for gene selection)
- **WGCNA** (Module extraction: adjacency.fromSimilarity on abs(co-expression) with power=1, TOMsimilarity, hierarchical clustering)
- **dynamicTreeCut** (cutreeDynamic on the TOM dendrogram (deepSplit=2, minClusterSize=10) to assign genes to modules)
- **clusterProfiler** (enrichGO functional enrichment of modules (OrgDb='org.Hs.eg.db', keyType='SYMBOL', ont='ALL'))
- **enrichplot** (dotplot visualization of enriched GO terms for selected modules)
- **pheatmap** (Heatmap visualization of Healthy-vs-COVID co-expression for module genes)
- **locCSN** (Alternative method referenced for cell-specific (per-cell) network estimation, not used in the CS-CORE workflow) — https://github.com/xuranw/locCSN
- **normalisr** (Alternative normalization and co-expression association testing framework referenced as a competing method) — https://github.com/lingfeiwang/normalisr

## Examples

```
install_github("ChangSuBiostats/CS-CORE"); CSCORE_result <- CSCORE(pbmc_B_healthy, genes = genes_selected)
```

## Evaluation signals

- Co-expression estimates must lie within [-1, 1]
- Differentially co-expressed interferon/defense-response module shows significant GO enrichment with BH-adjusted p < 1e-3 for defense response to virus terms and >= 10 enriched terms
- Permutation p-values from diff_CSCORE (n_permu=100, donor-level label shuffling) should be BH-adjusted with threshold 0.05 before module construction
- Compared against existing methods (rho-sctransform, rho-analytic PR), CS-CORE estimates should show lower false positives and greater reproducibility/enrichment for known transcription factor-target pairs

## Limitations

- The measurement model uses only a Poisson distribution; adaptation to negative binomial or other count models is left to future work
- CS-CORE assumes gene expressions from cells of the same cell type follow the same distribution, which may not hold across individuals with different genetic, demographic and clinical characteristics
- Individualized co-expression networks and cross-genotype/condition comparisons beyond the implemented diff_CSCORE workflow are planned but not implemented
- Existing methods (and by comparison, naive UMI correlations) suffer from inflated type-I errors and estimation biases confounded by sequencing depth
