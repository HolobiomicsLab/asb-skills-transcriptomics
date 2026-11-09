---
name: cell-type-mean-expression-profile-computation
description: Use when preparing labelled scRNA-seq training data (e.g. SingleCellExperiment
  objects with known cell labels) and you need per-cell-type mean expression profiles
  plus a cross-cell-type correlation matrix to decide a classifier strategy.
when_to_use_negative:
- The new sample may contain cell types absent from the reference/training data —
  NeuCA assumes all cell types in the new sample already exist in the training data
- You need to detect unknown or novel cell types; NeuCA requires tools like those
  for rare/unknown cell identification (or inferCNV/copyKAT for malignant cells) as
  a prior step
- The input lacks known cell labels, since mean profiles per cell type cannot be computed
  without labels
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: Provides the averaged-profile and correlation computation and the correlation-based
    strategy selection between feed-forward and hierarchical neural networks
  repo: https://bioconductor.org/packages/NeuCA
- name: R
  role: Language in which NeuCA and its dependency packages (keras, limma, SingleCellExperiment,
    e1071) are implemented and run
  repo: https://bioconductor.org/packages/NeuCA/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:1eb722d93e0b094a30f8c63c011cc59cfde879cff7f3452bac98b37c633b6fe6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cell-type-mean-expression-profile-computation

## Summary

Computes a G x K averaged expression profile matrix from labelled scRNA-seq training data (mean expression per cell type) and the K x K Pearson correlation matrix across cell types, which NeuCA uses to select between a feed-forward neural network and a marker-guided hierarchical neural network (threshold tau = 0.95).

## When to use

Use when preparing labelled scRNA-seq training data (e.g. SingleCellExperiment objects with known cell labels) and you need per-cell-type mean expression profiles plus a cross-cell-type correlation matrix to decide a classifier strategy.

## When NOT to use

- The new sample may contain cell types absent from the reference/training data — NeuCA assumes all cell types in the new sample already exist in the training data
- You need to detect unknown or novel cell types; NeuCA requires tools like those for rare/unknown cell identification (or inferCNV/copyKAT for malignant cells) as a prior step
- The input lacks known cell labels, since mean profiles per cell type cannot be computed without labels

## Inputs

- Labelled scRNA-seq training count/normalized expression matrix (genes x cells)
- Known cell type labels per cell (e.g. as colData of a SingleCellExperiment object)
- Optional testing SingleCellExperiment object subset to overlapped genes

## Outputs

- G x K averaged (mean) expression profile matrix per cell type
- K x K symmetric Pearson correlation matrix of cell type profiles with unit diagonal (e.g. cell_type_correlation_matrix.csv)
- Strategy decision: feed-forward neural network vs hierarchical neural network based on tau = 0.95

## How to apply

Load the labelled training scRNA-seq data (e.g. Zheng_8ct_PBMC from DuoClustering2018 or Baron_scRNA) as a SingleCellExperiment object with an 'normcounts' assay and cell labels in colData, removing duplicated feature symbols. Normalize the expression matrix and subset training/testing data to overlapped genes, as in NeuCA's preprocessing. Compute the averaged expression profile by taking the mean expression value of cells within each cell type, yielding a G x K profile matrix A. Compute the Pearson correlation matrix P = (p_i,j) across the K cell-type columns, then compare all off-diagonal elements of P against the threshold tau = 0.95: feed-forward network when all off-diagonal correlations are below tau, hierarchical route otherwise. Save the correlation matrix (e.g. cell_type_correlation_matrix.csv) and record the strategy decision.

## Related tools

- **NeuCA** (Provides the averaged-profile and correlation computation and the correlation-based strategy selection between feed-forward and hierarchical neural networks) — https://bioconductor.org/packages/NeuCA
- **R** (Language in which NeuCA and its dependency packages (keras, limma, SingleCellExperiment, e1071) are implemented and run) — https://bioconductor.org/packages/NeuCA/

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- The correlation matrix is symmetric K x K with a unit diagonal
- All off-diagonal Pearson correlations fall in [-1, 1] and the recorded strategy decision matches the rule that the feed-forward network is used only when all off-diagonal correlations are below tau = 0.95
- The profile matrix has one column per cell type in the training labels (K matches the number of distinct labels)
- Inputs are valid SingleCellExperiment objects with 'normcounts' assays and no duplicated feature symbols
- Sanity-check downstream annotation accuracy (e.g. confusion matrix against Seg_true_cell_label) shows the chosen route performs as expected

## Limitations

- NeuCA is designed for well-studied situations where all cell types in the new sample already exist in the reference/training data
- Correlation-based methods may struggle when several cell types are simultaneously highly correlated; the tau = 0.95 rule only gates the route choice
- Deciding whether unknown/novel cell types exist is not addressed by the method; expert consultation is recommended

## Evidence

- [intro] NeuCA first obtains the mean gene expression profile for each cell type and calculates the correlation matrix across cell types: "NeuCA first obtains the mean gene expression profile for each cell type and calculates the correlation matrix across cell types"
- [methods] Preprocessing: normalize and subset to overlapped genes: "After the preprocessing steps, such as normalizing the expression matrix and taking subsets of"
- [methods] Threshold tau = 0.95 for strategy selection: "In the cases where all the off-diagonal elements in"
- [readme] NeuCA automatically selects between general and hierarchical neural network: "NeuCA can detect whether highly-correlated cell types exist in the training dataset, and automatically determine if a general neural-network model will be adopted"
- [discussion] NeuCA scope limitation on cell types in reference data: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
