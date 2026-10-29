---
name: density-index-optimization
description: Use when running DUBStepR with optimise.features = TRUE and you must
  decide how many correlated feature genes to keep beyond the seed set, e.g. after
  computing a gene-gene correlation (GGC) matrix on log-normalized scRNA-seq counts.
when_to_use_negative:
- The goal is differential expression testing or marker ranking, not choosing a gene
  subset for cell clustering — the Density Index optimizes cluster separation, not
  DE significance.
- Input is already a fixed feature table or a predefined marker list with no need
  to choose a set size.
- The dataset lacks cell-type structure where marker-gene correlations are expected
  (DUBStepR assumes cell-type-specific marker genes are well correlated).
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DUBStepR
  role: Implements stepwise-regression seed selection, guilt-by-association expansion,
    and Density Index optimization via getOptimalFeatureSet
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Provides the log-normalized Seurat object input and receives the optimal feature
    set in the var.features slot
  repo: https://satijalab.org/seurat/
- name: R
  role: Runtime environment; DUBStepR requires R >= 3.5.0
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:f915d3c852e3204c400731b33f6d4ba3bed034527ba14fce83ba8e80a4925b3e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# density-index-optimization

## Summary

DUBStepR's feature-set size optimization: expand the minimally redundant stepwise-regression seed genes by guilt-by-association and select the gene set size that maximizes the Density Index as the optimal feature set for clustering single-cell RNA-seq data.

## When to use

Use when running DUBStepR with optimise.features = TRUE and you must decide how many correlated feature genes to keep beyond the seed set, e.g. after computing a gene-gene correlation (GGC) matrix on log-normalized scRNA-seq counts.

## When NOT to use

- The goal is differential expression testing or marker ranking, not choosing a gene subset for cell clustering — the Density Index optimizes cluster separation, not DE significance.
- Input is already a fixed feature table or a predefined marker list with no need to choose a set size.
- The dataset lacks cell-type structure where marker-gene correlations are expected (DUBStepR assumes cell-type-specific marker genes are well correlated).

## Inputs

- Filtered, log-normalized gene-by-cell expression matrix (e.g. from a Seurat object built on the 10X pbmc_1k_v2 filtered feature-barcode matrix)
- Ordered gene list from guilt-by-association expansion of the stepwise-regression seed features
- Elbow point from findElbow (used to define the candidate feature-set size range)

## Outputs

- Optimal feature gene set maximizing the Density Index (e.g. written to optimal_feature_genes.csv)
- Ranked/ordered gene list with Density Index values per feature-set size
- Updated Seurat variable features slot (seuratObj@assays$RNA@var.features)

## How to apply

After DUBStepR constructs the GGC matrix and performs stepwise regression to identify minimally redundant seed features, iteratively expand the feature set by adding the candidate gene most strongly correlated with the current set (guilt-by-association ordering). At each feature-set size, call getOptimalFeatureSet(filt.data, ordered.genes, elbow.pt, k, num.pcs, error) to recompute the Density Index, clustering cells in the first num.pcs (e.g. 20) principal components with k nearest neighbors (e.g. k = 10). Select the gene set size that maximizes the Density Index as the optimal feature gene set and assign it to seuratObj@assays$RNA@var.features. The rationale is that cell-type-specific marker genes are well correlated with each other, so expanding around coherent seed genes with a density criterion enriches for informative, cell-type-specific DE genes. In benchmarks, DUBStepR approaches its maximum silhouette index at roughly 200–500 feature genes, a useful sanity range for the selected set size.

## Related tools

- **DUBStepR** (Implements stepwise-regression seed selection, guilt-by-association expansion, and Density Index optimization via getOptimalFeatureSet) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Provides the log-normalized Seurat object input and receives the optimal feature set in the var.features slot) — https://satijalab.org/seurat/
- **R** (Runtime environment; DUBStepR requires R >= 3.5.0)

## Examples

```
dub <- DUBStepR(input.data = norm.data, min.cells = 0.05*ncol(seuratObj), optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- The reported optimal feature-set size corresponds to the maximum Density Index across all evaluated set sizes in getOptimalFeatureSet output.
- The optimal set size falls in the expected 200–500 gene range where DUBStepR approaches its maximum silhouette index in benchmarks.
- The selected genes are a superset of the minimally redundant stepwise-regression seed features and are ordered by guilt-by-association correlation.
- seuratObj@assays$RNA@var.features matches the gene set written to optimal_feature_genes.csv.
- Downstream clustering on the selected features yields coherent, annotatable cell-type clusters (e.g. T, NK, monocyte populations in PBMC data).

## Limitations

- Elbow-point computation is not described in the main text and is deferred to supplementary notes (Supp. Note), so reproducing the candidate size range may require consulting the SI.
- Key supporting results (e.g. silhouette/Density Index behavior across feature counts) are in supplementary figures (Supp. Fig. S3) not included in the main text.
- DUBStepR's Density Index optimization is designed for scRNA-seq clustering; its behavior on other data types (e.g. single-cell ATAC-seq) is straightforward but not specifically tuned here.

## Evidence

- [intro] Seed features are expanded by guilt-by-association: "which are then expanded by adding correlated features (guilt-by-association)"
- [intro] Stepwise regression yields minimally redundant seed features: "DUBStepR then performs stepwise regression on the GGC matrix to identify a minimally redundant subset of seed features"
- [methods] getOptimalFeatureSet API for Density Index optimization: "`getOptimalFeatureSet(filt.data, ordered.genes, elbow.pt, k, num.pcs, error)`"
- [discussion] Expected optimal feature-set size range: "DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3)"
- [readme] Rationale: marker genes are correlated: "DUBStepR is based on the intuition that cell-type-specific marker genes tend to be well correlated with each other"
