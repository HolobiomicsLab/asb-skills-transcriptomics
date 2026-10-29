---
name: pearson-correlation-matrix-computation
description: Use when a supervised cell annotation pipeline must decide between model
  architectures based on cross-cell-type similarity, or when a K x K cell-type correlation
  matrix from averaged scRNA-seq expression profiles is required.
when_to_use_negative:
- The new sample contains cell types that do not exist in the reference/training data
  (NeuCA is designed for exhaustively known cell types)
- Input is raw unnormalized counts without cell type labels — correlation profiles
  cannot be computed per type without labels
- The goal is to identify unknown/rare or malignant cells rather than assign known
  labels; pre-filter with tools like inferCNV or copyKAT first instead
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: NeuCA
  role: 'Implements the correlation-based strategy selection: computes the mean gene
    expression profile per cell type and the correlation matrix to choose between
    feed-forward and hierarchical neural networks'
  repo: https://bioconductor.org/packages/NeuCA
- name: R
  role: Runtime environment; NeuCA is an R/Bioconductor package in which the correlation
    matrix is computed
  repo: https://bioconductor.org/packages/NeuCA
- name: SingleCellExperiment
  role: Bioconductor class used to organize the normalized training data (normcounts
    assay plus colData cell labels) from which profiles are averaged
  repo: https://bioconductor.org/packages/NeuCA/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:75c7fda922a02f387f2a000bdea2a7be1e4ba7c98e349698e582c49456e1fee8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  - SingleCellExperiment
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pearson-correlation-matrix-computation

## Summary

Compute a Pearson correlation matrix across K cell types from their mean gene expression profiles, used by NeuCA to gate its annotation strategy between a feed-forward neural network and a cell lineage-based hierarchical neural network at a threshold of tau = 0.95.

## When to use

Use when a supervised cell annotation pipeline must decide between model architectures based on cross-cell-type similarity, or when a K x K cell-type correlation matrix from averaged scRNA-seq expression profiles is required.

## When NOT to use

- The new sample contains cell types that do not exist in the reference/training data (NeuCA is designed for exhaustively known cell types)
- Input is raw unnormalized counts without cell type labels — correlation profiles cannot be computed per type without labels
- The goal is to identify unknown/rare or malignant cells rather than assign known labels; pre-filter with tools like inferCNV or copyKAT first instead

## Inputs

- Labelled training scRNA-seq expression matrix (genes x cells), e.g. Zheng_8ct_PBMC from DuoClustering2018, as a SingleCellExperiment object with normcounts assay
- Cell type label vector (colData of the SingleCellExperiment) for the training cells

## Outputs

- G x K averaged (mean) expression profile matrix, one column per cell type
- Symmetric K x K Pearson cell type correlation matrix with unit diagonal (e.g. cell_type_correlation_matrix.csv)
- Binary strategy decision: feed-forward network vs. cell lineage-based hierarchical neural network

## How to apply

First normalize the expression matrix of the labelled training data (e.g. Zheng_8ct_PBMC from DuoClustering2018) and subset training and testing data by overlapped genes, keeping the data in SingleCellExperiment format. Compute the averaged expression profile matrix for all the cell types in the training data by taking the mean expression value of cells within each cell type, yielding a G x K profile matrix. Then compute the Pearson correlation matrix P = (p_i,j) across the K cell-type columns. Compare all off-diagonal elements of P against the threshold tau = 0.95: if all off-diagonal correlations are below tau, train a feed-forward neural network; otherwise, adopt the marker-guided hierarchical neural-network route that exploits cell type tree structure. The rationale is that closely correlated cell types need hierarchical structure information to be separated accurately, while weakly correlated types are handled by a simple feed-forward network.

## Related tools

- **NeuCA** (Implements the correlation-based strategy selection: computes the mean gene expression profile per cell type and the correlation matrix to choose between feed-forward and hierarchical neural networks) — https://bioconductor.org/packages/NeuCA
- **R** (Runtime environment; NeuCA is an R/Bioconductor package in which the correlation matrix is computed) — https://bioconductor.org/packages/NeuCA
- **SingleCellExperiment** (Bioconductor class used to organize the normalized training data (normcounts assay plus colData cell labels) from which profiles are averaged) — https://bioconductor.org/packages/NeuCA/

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- Output correlation matrix is symmetric K x K with unit diagonal (Pearson self-correlation)
- All off-diagonal entries fall in [-1, 1]
- Strategy decision matches the rule: feed-forward network only when all off-diagonal elements of P are smaller than tau = 0.95
- Number of columns in the averaged profile matrix equals the number of distinct cell types in the training labels
- Predicted.label from the subsequent NeuCA call covers only cell types present in the training data, consistent with the exhaustive-assignment design

## Limitations

- NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data
- Correlation-based strategy selection uses a fixed threshold (tau = 0.95); borderline correlation structures may be sensitive to this cutoff
- No version numbers given for keras, TensorFlow, or R in the software description, complicating reproducibility of the neural-network stage

## Evidence

- [methods] Strategy selection via correlation matrix: "NeuCA first obtains the mean gene expression profile for each cell type and calculates the correlation matrix across cell types"
- [methods] Threshold tau = 0.95 gating: [evidence span withheld — anchor status: not_found]
- [abstract] Hierarchical rationale: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
- [readme] README strategy description: "NeuCA can detect whether highly-correlated cell types exist in the training dataset, and automatically determine if a general neural-network model will be adopted"
- [readme] README input format: [evidence span withheld — anchor status: external_doc]
