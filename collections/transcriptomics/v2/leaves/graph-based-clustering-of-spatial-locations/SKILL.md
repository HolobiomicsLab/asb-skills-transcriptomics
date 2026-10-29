---
name: graph-based-clustering-of-spatial-locations
description: Use when a spatially-resolved transcriptomics dataset (e.g., Visium human
  DLPFC SpatialExperiment with manual layer labels) must be clustered into spatial
  domains, and you need to compare feature-selection strategies (nnSVG, SPARK-X, Moran's
  I, HVGs) as clustering inputs.
when_to_use_negative:
- The goal is identification and ranking of spatially variable genes themselves rather
  than downstream spatially-aware clustering — use nnSVG/SPARK-X directly instead
  of this clustering step
- Input data have not been preprocessed (no library-size normalization and logcounts
  computed) or still contain mitochondrial and low-expressed genes — run filter_genes()
  and logNormCounts() first
- The dataset lacks ground-truth spatial domain annotations, making ARI evaluation
  against layer labels impossible
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: nnSVG
  role: Provides top-ranked SVGs (ordered by LR_stat in rowData) used as the feature-selection
    input for the clustering; also supplies the filter_genes() preprocessing function
  repo: https://bioconductor.org/packages/nnSVG
- name: SPARK-X
  role: Alternative feature-selection method supplying a competing top-1000 gene ranking
    for the ARI comparison
- name: SpatialExperiment
  role: Bioconductor container holding counts, spatial coordinates, and layer annotations
    used as clustering input
  repo: https://bioconductor.org/packages/SpatialExperiment
- name: STexampleData
  role: Source package for the Visium_humanDLPFC() example dataset clustered in the
    workflow
  repo: https://bioconductor.org/packages/nnSVG
- name: scran
  role: computeLibraryFactors and logNormCounts produce the logcounts matrix used
    for PCA prior to graph construction
- name: mclust
  role: adjustedRandIndex quantifies agreement between each Louvain clustering and
    manual cortical layer labels
- name: Bioconductor
  role: Framework through which nnSVG, SpatialExperiment, and STexampleData are installed
    and run
  repo: https://bioconductor.org/packages/nnSVG
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:6a11ee8491cb98df2b057adc703c9aa4d190218fd83c27913e92a59a50bc007c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - SPARK-X
  - STexampleData
  - SpatialExperiment
  - mclust
  - nnSVG
  - scran
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# graph-based-clustering-of-spatial-locations

## Summary

Perform spatially-aware unsupervised clustering of Visium spatial locations by building a shared nearest-neighbor graph on a top-50 PCA embedding derived from a selected gene set (top-ranked SVGs vs. top HVGs) and running Louvain community detection, then judging recovery of manual cortical layer annotations via adjusted Rand index.

## When to use

Use when a spatially-resolved transcriptomics dataset (e.g., Visium human DLPFC SpatialExperiment with manual layer labels) must be clustered into spatial domains, and you need to compare feature-selection strategies (nnSVG, SPARK-X, Moran's I, HVGs) as clustering inputs.

## When NOT to use

- The goal is identification and ranking of spatially variable genes themselves rather than downstream spatially-aware clustering — use nnSVG/SPARK-X directly instead of this clustering step
- Input data have not been preprocessed (no library-size normalization and logcounts computed) or still contain mitochondrial and low-expressed genes — run filter_genes() and logNormCounts() first
- The dataset lacks ground-truth spatial domain annotations, making ARI evaluation against layer labels impossible

## Inputs

- SpatialExperiment Bioconductor object with raw counts, spatial coordinates, and manually annotated layer labels (Visium human DLPFC from the nnSVG package / STexampleData, 33538 genes x 4992 spots, 3639 over-tissue spots)
- Log-transformed normalized counts (logcounts) computed with library-size factors
- Ranked gene lists per feature-selection strategy: nnSVG rank/LR_stat from rowData, SPARK-X ranks, Moran's I ranks, and HVG ranks

## Outputs

- Louvain cluster labels per spatial location for each gene-set strategy
- Adjusted Rand index (ARI) per strategy computed against manual cortical layer annotations with mclust::adjustedRandIndex
- Comparison table written to ari_clustering_comparison.csv (four ARI values: nnSVG, SPARK-X, Moran's I, HVGs)

## How to apply

Load the SpatialExperiment with raw counts, spatial coordinates, and manual layer annotations (e.g., Visium_humanDLPFC() from STexampleData), keep spots over tissue, and preprocess with library-size normalization (computeLibraryFactors) and log-transformation (logNormCounts) before running PCA on all spots. For each feature-selection strategy, take the top 1000 ranked genes (nnSVG ranked by LR_stat, SPARK-X, Moran's I, or HVGs), subset the top-50-PC embedding accordingly, build a shared nearest-neighbor graph on those 50 PCs, and run graph-based Louvain clustering. The rationale, per the article, is to replace the set of top HVGs with the set of top SVGs from nnSVG so clustering becomes spatially aware. Compute adjustedRandIndex against the manual cortical layer labels (mclust) for each strategy, assemble the ARI values into a comparison table (ari_clustering_comparison.csv), and validate that the reproduced ordering matches the reported finding: nnSVG and Moran's I outperform both HVGs and SPARK-X.

## Related tools

- **nnSVG** (Provides top-ranked SVGs (ordered by LR_stat in rowData) used as the feature-selection input for the clustering; also supplies the filter_genes() preprocessing function) — https://bioconductor.org/packages/nnSVG
- **SPARK-X** (Alternative feature-selection method supplying a competing top-1000 gene ranking for the ARI comparison)
- **SpatialExperiment** (Bioconductor container holding counts, spatial coordinates, and layer annotations used as clustering input) — https://bioconductor.org/packages/SpatialExperiment
- **STexampleData** (Source package for the Visium_humanDLPFC() example dataset clustered in the workflow) — https://bioconductor.org/packages/nnSVG
- **scran** (computeLibraryFactors and logNormCounts produce the logcounts matrix used for PCA prior to graph construction)
- **mclust** (adjustedRandIndex quantifies agreement between each Louvain clustering and manual cortical layer labels)
- **Bioconductor** (Framework through which nnSVG, SpatialExperiment, and STexampleData are installed and run) — https://bioconductor.org/packages/nnSVG

## Examples

```
library(nnSVG); library(STexampleData); spe <- Visium_humanDLPFC(); spe <- spe[, colData(spe)$in_tissue == 1]; spe <- filter_genes(spe); spe <- computeLibraryFactors(spe); spe <- logNormCounts(spe); set.seed(123); spe <- nnSVG(spe)
```

## Evaluation signals

- Reproduced ARI ordering matches the reported result: nnSVG and Moran's I outperform HVGs and SPARK-X in adjusted Rand index against the cortical layer labels
- All four strategies yield an ARI value computed with mclust::adjustedRandIndex and are assembled into ari_clustering_comparison.csv
- Each clustering is built on a shared nearest-neighbor graph over exactly 50 PCs computed from the top 1000 ranked genes of the respective strategy
- Over-tissue spot filtering is applied consistently across strategies (Visium DLPFC: 4992 spots reduced to 3639 in_tissue == 1 spots) so cluster labels and layer labels align on the same spot set
- Louvain runs on the same SNN graph construction across strategies so ARI differences are attributable to the gene set, not the graph parameters

## Limitations

- nnSVG ranks SVGs per sample; for multi-sample datasets ranking is averaged across samples rather than derived from a principled joint statistical model, which affects the gene set fed into clustering
- Small negative LR statistics occur mainly for lower-ranked genes and are difficult to interpret, which can perturb the top-1000 ranking boundary
- P-value calibration for low-expressed genes is imperfect; adaptive filtering thresholds that would remove low-expressed genes are proposed but not yet implemented
- Transcriptome-wide datasets with >=10,000 spatial locations require several hours of runtime with 10 processor cores on an HPC cluster, so gene filtering stringency controls feasibility
- nnSVG does not group SVGs into metagenes; clustering operates on individual top-ranked genes only

## Evidence

- [discussion] Replace top HVGs with top SVGs from nnSVG and perform unsupervised clustering on the SVG set for spatially-aware clustering: "replacing the set of top HVGs with the set of top SVGs from nnSVG, and then perform unsupervised clustering on the set of top SVGs"
- [discussion] Use the set of top-ranked SVGs as input for downstream spatially-aware unsupervised clustering: "use the set of top-ranked SVGs as the input for further downstream analyses, such as spatially-aware unsupervised clustering"
- [discussion] Select either an arbitrary number of top-ranked SVGs or statistically significant SVGs via adjusted p-values: "either select an arbitrary set of top-ranked SVGs (e.g., top 100 genes) or select a set of statistically significant SVGs with adjusted p-values from the LR test"
- [readme] nnSVG results stored in rowData of the SpatialExperiment with rank, LR_stat, pval, padj, and prop_sv columns: "LR_stat`: likelihood ratio (LR) statistics used to rank SVGs
- `rank`: rank of top SVGs according to LR statistics"
- [readme] Load the Visium human DLPFC example dataset and preprocess by keeping over-tissue spots, filtering genes, and computing logcounts: "spe <- Visium_humanDLPFC()
spe <- spe[, colData(spe)$in_tissue == 1]
spe <- filter_genes(spe)
spe <- computeLibraryFactors(spe)
spe <- logNormCounts(spe)"
- [abstract] nnSVG scales linearly with the number of spatial locations, supporting application to clustering on datasets with thousands of locations: "nnSVG scales linearly with the number of spatial locations"
