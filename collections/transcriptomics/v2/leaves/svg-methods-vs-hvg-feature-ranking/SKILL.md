---
name: svg-methods-vs-hvg-feature-ranking
description: Use when comparing feature-selection strategies (SVG-ranking methods
  vs HVGs) as inputs to unsupervised spatial clustering, and you need a quantitative
  ARI comparison against annotated tissue domains on spatially-resolved transcriptomics
  data.
when_to_use_negative:
- The analysis goal is identifying spatial domain marker genes (mean-level changes
  between domains) rather than continuously spatially varying genes — the paper explicitly
  focuses on SVGs, not spatial domain marker genes
- The input is a small preselected gene set (e.g. the 16-gene demo subset) rather
  than a transcriptome-wide ranking — ARI comparison across top-1000 gene sets requires
  ranking the full gene panel
- The data lack spatial coordinates (e.g. dissociated scRNA-seq), since SVG ranking
  and spatially-aware clustering both depend on spatial locations
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: nnSVG
  role: 'Primary SVG-ranking method: nearest-neighbor Gaussian process model with
    BRISC fitting, ranks genes by LR_stat for the top-1000 feature set'
  repo: https://bioconductor.org/packages/nnSVG
- name: SPARK-X
  role: Comparison SVG-ranking method based on nonparametric covariance tests, evaluated
    head-to-head against nnSVG and Moran's I
- name: mclust
  role: Provides adjustedRandIndex to score each clustering against manual cortical
    layer labels
- name: scran
  role: computeLibraryFactors and logNormCounts for library-size normalization and
    log-transformation before PCA
- name: STexampleData
  role: Supplies the Visium_humanDLPFC() SpatialExperiment example dataset with layer
    annotations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:ad6a4f84258b78073acc26686d837318858c037d1b75554698ec4805c356b35d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - SPARK-X
  - STexampleData
  - mclust
  - nnSVG
  - scran
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# svg-methods-vs-hvg-feature-ranking

## Summary

Benchmark whether top-ranked spatially variable genes (SVGs) from methods like nnSVG, SPARK-X, and Moran's I are a better feature-selection input for downstream spatially-aware clustering than top HVGs, quantified by adjusted Rand index against manual annotations on the Visium human DLPFC dataset.

## When to use

Use when comparing feature-selection strategies (SVG-ranking methods vs HVGs) as inputs to unsupervised spatial clustering, and you need a quantitative ARI comparison against annotated tissue domains on spatially-resolved transcriptomics data.

## When NOT to use

- The analysis goal is identifying spatial domain marker genes (mean-level changes between domains) rather than continuously spatially varying genes — the paper explicitly focuses on SVGs, not spatial domain marker genes
- The input is a small preselected gene set (e.g. the 16-gene demo subset) rather than a transcriptome-wide ranking — ARI comparison across top-1000 gene sets requires ranking the full gene panel
- The data lack spatial coordinates (e.g. dissociated scRNA-seq), since SVG ranking and spatially-aware clustering both depend on spatial locations

## Inputs

- Visium human DLPFC SpatialExperiment object with raw counts, spatial coordinates, and manually annotated layer labels (from nnSVG/STexampleData package, e.g. Visium_humanDLPFC(), 33538 genes x 4992 spots, 3639 in-tissue)
- Alternative input: numeric matrix of log-transformed normalized counts (logcounts) plus numeric matrix of spatial coordinates
- Manual cortical layer annotation labels per spot for ARI evaluation

## Outputs

- Ranked SVG table stored in rowData of the SpatialExperiment (columns LR_stat, rank, pval, padj, prop_sv)
- Per-strategy Louvain cluster labels on spots from the shared nearest-neighbor graph over top-50 PCs
- Comparison table of adjusted Rand index values written to ari_clustering_comparison.csv (one ARI per feature-selection strategy: nnSVG, SPARK-X, Moran's I, HVGs)

## How to apply

Load the Visium human DLPFC data as a SpatialExperiment, keep spots with in_tissue == 1, apply nnSVG's filter_genes() with default parameters (removes 13 mitochondrial genes and low-expressed genes with fewer than 3 counts in 0.5% of locations), and compute scran library-size logcounts. Run each feature-selection method transcriptome-wide — nnSVG ranked by LR_stat, SPARK-X, Moran's I, and HVG detection — then take the top 1000 genes from each ranking and recompute PCA restricted to each gene set, retaining the top 50 PCs. Build a shared nearest-neighbor graph on the 50 PCs and run Louvain clustering per strategy. Compute mclust::adjustedRandIndex between each clustering and the manual cortical layer labels; the rationale is that SVGs capture continuous spatial expression variation (with gene-specific length scale parameters in nnSVG's nearest-neighbor Gaussian process model) that HVGs miss. Validation: the ARI ordering must match the reported result — nnSVG and Moran's I outperform both HVGs and SPARK-X.

## Related tools

- **nnSVG** (Primary SVG-ranking method: nearest-neighbor Gaussian process model with BRISC fitting, ranks genes by LR_stat for the top-1000 feature set) — https://bioconductor.org/packages/nnSVG
- **SPARK-X** (Comparison SVG-ranking method based on nonparametric covariance tests, evaluated head-to-head against nnSVG and Moran's I)
- **mclust** (Provides adjustedRandIndex to score each clustering against manual cortical layer labels)
- **scran** (computeLibraryFactors and logNormCounts for library-size normalization and log-transformation before PCA)
- **STexampleData** (Supplies the Visium_humanDLPFC() SpatialExperiment example dataset with layer annotations)

## Examples

```
spe <- Visium_humanDLPFC(); spe <- spe[, colData(spe)$in_tissue == 1]; spe <- filter_genes(spe); spe <- logNormCounts(computeLibraryFactors(spe)); set.seed(123); spe <- nnSVG(spe)
```

## Evaluation signals

- ARI ordering reproduced: nnSVG and Moran's I must achieve higher adjustedRandIndex against cortical layer labels than both HVGs and SPARK-X
- rowData of the SpatialExperiment contains the expected nnSVG result columns (LR_stat, rank, pval, padj, prop_sv) after running nnSVG()
- Gene filtering matches default filter_genes() behavior: 13 mitochondrial genes removed and 30216/33525 low-expression genes removed on the Visium DLPFC example
- Spot filtering leaves 3639 in-tissue spots from 4992 (in_tissue == 1)
- Significance counts consistent with rank ordering, e.g. table(rowData(spe)$padj <= 0.05) reports the number of significant SVGs

## Limitations

- Runtime is substantial for large slides: transcriptome-wide datasets with >=10,000 spatial locations require runtimes on the order of several hours with 10 processor cores on a high-performance compute cluster; runtime depends on the number of genes and can be reduced with more stringent gene filtering
- Some small negative values in estimated LR statistics occur, mainly for lower-ranked genes, and are difficult to interpret
- P-value calibration for low-expressed genes could be improved; adaptive filtering thresholds are proposed but not yet implemented
- For multiple-sample datasets nnSVG uses rank averaging within samples rather than a principled statistical approach
- SPARK-X fits a fixed combination of covariance functions and length scale parameters across all genes, reducing flexibility to identify SVGs with different spatial ranges

## Evidence

- [abstract] nnSVG is a scalable GP-based SVG method: "we propose nnSVG, a scalable approach to identify spatially variable genes based on nearest-neighbor Gaussian processes"
- [discussion] Top SVGs replace HVGs for unsupervised clustering: "replacing the set of top HVGs with the set of top SVGs from nnSVG, and then perform unsupervised clustering on the set of top SVGs"
- [discussion] Selection of top-ranked or significant SVGs: "either select an arbitrary set of top-ranked SVGs (e.g., top 100 genes) or select a set of statistically significant SVGs with adjusted p-values from the LR test"
- [readme] Results stored in rowData: "The results are stored in the `rowData` of the `SpatialExperiment` object."
- [readme] Alternative matrix input format: "the inputs can also be provided as a numeric matrix of normalized and transformed counts (e.g. log-transformed normalized counts, also known as logcounts)"
- [readme] Default gene filtering thresholds: [evidence span withheld — anchor status: external_doc]
