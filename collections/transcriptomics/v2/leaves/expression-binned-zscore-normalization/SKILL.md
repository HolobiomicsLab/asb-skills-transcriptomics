---
name: expression-binned-zscore-normalization
description: Use when selecting well-correlated candidate feature genes from a log-normalized
  scRNA-seq matrix, and gene-level correlation statistics must be normalized against
  expression-level confounding by binning genes by mean expression.
when_to_use_negative:
- Input is already a ranked feature set or a cell-cluster assignment — this skill
  only selects the initial candidate feature genes
- The expression matrix has not been filtered/normalized (raw counts with mitochondrial,
  ribosomal, spike-in, and pseudogenes still present), since the GGC and range scores
  would be dominated by technical genes
- You need the final minimally redundant feature set — that requires the subsequent
  DUBStepR stepwise regression and guilt-by-association expansion steps
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DUBStepR
  role: Provides getGGC, getCorrelationRange, and getFilteredData used to build the
    GGC matrix, compute range scores, and filter genes
  repo: github.com/prabhakarlab/DUBStepR
- name: R
  role: Runtime environment; DUBStepR requires R version >= 3.5.0
- name: Seurat
  role: Recommended package for building and handling the single-cell data object
- name: hdf5r
  role: Used to load the 10x HDF5 feature-cell matrix into the Seurat object
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:90bb7cec87f5b3c69f6399a39dd070c15ff334fe1c3db8bf49af7ee64a067dbd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - R
  - Seurat
  - hdf5r
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Expression-binned z-score normalization of gene-gene correlation range scores

## Summary

DUBStepR's initial candidate feature selection step: after removing mitochondrial, ribosomal, spike-in, and pseudogenes, it builds a gene-gene correlation (GGC) matrix, computes each gene's correlation range score c_i = max3(G_i) - 0.75*min(G_i), bins genes by mean expression, converts range scores to z-scores within bins, and retains genes with z-score > 0.7.

## When to use

Use when selecting well-correlated candidate feature genes from a log-normalized scRNA-seq matrix, and gene-level correlation statistics must be normalized against expression-level confounding by binning genes by mean expression.

## When NOT to use

- Input is already a ranked feature set or a cell-cluster assignment — this skill only selects the initial candidate feature genes
- The expression matrix has not been filtered/normalized (raw counts with mitochondrial, ribosomal, spike-in, and pseudogenes still present), since the GGC and range scores would be dominated by technical genes
- You need the final minimally redundant feature set — that requires the subsequent DUBStepR stepwise regression and guilt-by-association expansion steps

## Inputs

- Filtered, log-normalized gene-by-cell expression matrix (e.g. 10x pbmc_1k_v2 HDF5 loaded via hdf5r)
- Gene-gene correlation (GGC) matrix from DUBStepR::getGGC
- Per-gene mean expression levels used to define expression bins

## Outputs

- Gene-level table with correlation range scores and expression-binned z-scores (gene_correlation_range_scores.csv)
- Candidate GGC submatrix restricted to genes with z-score > 0.7 (candidate_ggc_matrix.rds)

## How to apply

Load the filtered 10x feature-cell matrix (HDF5 via hdf5r), log-normalize it, and filter genes detected in fewer than 5% of cells (min.cells = 0.05*ncol) with getFilteredData. Build the gene-gene correlation matrix with DUBStepR::getGGC. Apply DUBStepR::getCorrelationRange to compute the per-gene correlation range score c_i = max3(G_i) - 0.75*min(G_i), which rewards genes with both strong positive and negative correlations to other genes (marker genes tend to be well correlated). Bin genes by mean expression level and convert each range score to a z-score within its expression bin, since raw correlation ranges vary with expression. Retain genes with z-score > 0.7 as the candidate feature set and extract the corresponding GGC submatrix for downstream stepwise regression.

## Related tools

- **DUBStepR** (Provides getGGC, getCorrelationRange, and getFilteredData used to build the GGC matrix, compute range scores, and filter genes) — github.com/prabhakarlab/DUBStepR
- **R** (Runtime environment; DUBStepR requires R version >= 3.5.0)
- **Seurat** (Recommended package for building and handling the single-cell data object)
- **hdf5r** (Used to load the 10x HDF5 feature-cell matrix into the Seurat object)

## Examples

```
range_scores <- getCorrelationRange(ggc_matrix)
```

## Evaluation signals

- gene_correlation_range_scores.csv exists with one row per gene and contains both a range-score and a z-score column
- The retained candidate gene set consists of exactly the rows with z-score > 0.7
- Each gene's z-score is computed within its expression bin (genes sharing a mean-expression bin are normalized together)
- The candidate GGC submatrix is the submatrix of the full GGC restricted to the retained genes (row/column set match)
- Genes entering the GGC step are those that passed getFilteredData with min.cells = 0.05*ncol and exclusion of mitochondrial, ribosomal, spike-in, and pseudogenes

## Limitations

- Elbow point and downstream optimization details are deferred to supplementary notes, so this skill alone does not determine the final feature count
- The z > 0.7 threshold and the 0.75 weight on the minimum correlation are method-specific parameters; sensitivity to these settings is not characterized in the provided context
- Correlation range scoring presupposes that cell-type-specific marker genes are well correlated; datasets violating this intuition may yield poor candidates

## Evidence

- [intro] DUBStepR constructs a GGC matrix and bins genes by expression to compute their correlation range z-scores, which are used to select well-correlated genes.: "DUBStepR constructs a GGC matrix and bins genes by expression to compute their correlation range z-scores, which are used to select well-correlated genes."
- [intro] After filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, DUBStepR constructs a GGC matrix: "After filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, DUBStepR constructs a GGC matrix"
- [methods] `getCorrelationRange(correlation_matrix)`: "getCorrelationRange(correlation_matrix)"
- [readme] After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene–gene correlation matrix to prioritize genes as features for clustering.: "After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene–gene correlation matrix to prioritize genes as features for clustering."
- [discussion] DUBStepR is based on the intuition that cell-type-specific marker genes tend to be well correlated with each other: "DUBStepR is based on the intuition that cell-type-specific marker genes tend to be well correlated with each other"
