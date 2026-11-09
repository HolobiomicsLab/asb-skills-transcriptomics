---
name: correlation-range-feature-scoring
description: Use when selecting an initial candidate feature gene set for single-cell
  RNA-seq clustering via gene-gene correlations, after filtering out mitochondrial,
  ribosomal, spike-in, and pseudogenes from log-normalized count data.
when_to_use_negative:
- The input is already a curated feature gene table or marker list — no correlation-based
  re-selection is needed
- Mitochondrial, ribosomal, spike-in, and pseudogene features have not yet been removed
  — DUBStepR expects these filtered before GGC construction
- The analysis goal is dimensionality reduction or clustering itself rather than selecting
  well-correlated candidate feature genes
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DUBStepR
  role: Provides getGGC to build the gene-gene correlation matrix and getCorrelationRange
    to compute per-gene correlation range scores for feature selection
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Seurat object holds the filtered, log-normalized single-cell expression data
    on which the GGC is computed
- name: hdf5r
  role: Used to load the 10x HDF5 matrix into the Seurat object
- name: R
  role: Runtime environment; DUBStepR requires R version >= 3.5.0
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:883d3a6cee24c27d7be17b7acdfbabc860d2f49038b06b492185a757400d4f3a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - R
  - Seurat
  - hdf5r
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# correlation-range-feature-scoring

## Summary

Scores genes for feature selection by computing per-gene correlation range scores from a gene-gene correlation (GGC) matrix, converting them to expression-binned z-scores, and retaining well-correlated genes (z > 0.7) as DUBStepR's initial candidate feature set.

## When to use

Use when selecting an initial candidate feature gene set for single-cell RNA-seq clustering via gene-gene correlations, after filtering out mitochondrial, ribosomal, spike-in, and pseudogenes from log-normalized count data.

## When NOT to use

- The input is already a curated feature gene table or marker list — no correlation-based re-selection is needed
- Mitochondrial, ribosomal, spike-in, and pseudogene features have not yet been removed — DUBStepR expects these filtered before GGC construction
- The analysis goal is dimensionality reduction or clustering itself rather than selecting well-correlated candidate feature genes

## Inputs

- 10x Genomics HDF5 feature-cell matrix (e.g. pbmc_1k_v2) loaded via hdf5r into a Seurat object
- log-normalized gene-cell expression matrix filtered with min.cells = 0.05*ncol
- gene-gene correlation (GGC) matrix from DUBStepR::getGGC

## Outputs

- gene-level correlation range score and expression-binned z-score table (gene_correlation_range_scores.csv)
- candidate feature gene set (genes with correlation range z-score > 0.7)
- candidate submatrix of the GGC (candidate_ggc_matrix.rds)

## How to apply

Load the filtered 10x matrix (e.g. pbmc_1k_v2 HDF5 via hdf5r into a Seurat object) and log-normalize it with getFilteredData, dropping genes detected in fewer than 5% of cells (min.cells = 0.05*ncol). Build the gene-gene correlation matrix from the log-normalized data using DUBStepR::getGGC. Apply DUBStepR::getCorrelationRange to compute the per-gene correlation range score c_i = max3(G_i) - 0.75*min(G_i). Bin genes by mean expression level and convert each correlation range score to a z-score within its expression bin; retain genes with z-score > 0.7 as the candidate feature set. This captures the intuition that cell-type-specific marker genes tend to be well correlated with each other, so the correlation range prioritizes well-correlated genes before stepwise regression and guilt-by-association expansion.

## Related tools

- **DUBStepR** (Provides getGGC to build the gene-gene correlation matrix and getCorrelationRange to compute per-gene correlation range scores for feature selection) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Seurat object holds the filtered, log-normalized single-cell expression data on which the GGC is computed)
- **hdf5r** (Used to load the 10x HDF5 matrix into the Seurat object)
- **R** (Runtime environment; DUBStepR requires R version >= 3.5.0)

## Examples

```
library(DUBStepR); ggc <- getGGC(filt.data); range_z <- getCorrelationRange(ggc); cand <- names(which(range_z$optimal.correlation.range > 0.7))
```

## Evaluation signals

- gene_correlation_range_scores.csv exists with one row per gene and contains both the correlation range score and z-score columns
- The retained candidate gene set consists of exactly the genes with correlation range z-score > 0.7
- The candidate GGC matrix (candidate_ggc_matrix.rds) is the submatrix of the full GGC restricted to the retained genes
- Genes detected in fewer than 5% of cells (min.cells = 0.05*ncol) and mitochondrial/ribosomal/spike-in/pseudogene features are absent from the score table
- Correlation range scores follow c_i = max3(G_i) - 0.75*min(G_i) when recomputed from the GGC

## Limitations

- Existing feature selection methods perform inconsistently across datasets, occasionally even resulting in poorer clustering accuracy than without feature selection — evaluate on your dataset rather than assuming gains
- DUBStepR approaches its maximum silhouette index value at 200–500 feature genes, but detailed performance results and the elbow point computation are deferred to supplementary figures/notes not in the main text
- The correlation range step only produces the initial candidate set; the minimally redundant seed features and expanded feature set come from subsequent stepwise regression (getOptimalFeatureSet), not from this scoring alone

## Evidence

- [intro] After filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, DUBStepR constructs a GGC matrix: "After filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, DUBStepR constructs a GGC matrix"
- [intro] bins genes by expression to compute correlation range z-scores: "DUBStepR constructs a GGC matrix and bins genes by expression to compute their correlation range z-scores, which are used to select well-correlated genes."
- [readme] cell-type-specific marker genes tend to be well correlated: "DUBStepR is based on the intuition that cell-type-specific marker genes tend to be well correlated with each other"
- [readme] After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix: "After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix to prioritize genes as features for clustering."
- [methods] getCorrelationRange(correlation_matrix): "`getCorrelationRange(correlation_matrix)`"
- [methods] DUBStepR requires your R version to be >= 3.5.0: "DUBStepR requires your R version to be >= 3.5.0"
