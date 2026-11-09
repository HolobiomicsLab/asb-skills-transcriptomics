---
name: complete-linkage-hierarchical-clustering-cell-lineage-tree
description: Use when the Pearson correlation matrix P over the training data's averaged
  cell-type profiles has off-diagonal entries >= 0.95, so a single feed-forward network
  is insufficient and the hierarchical, marker-guided strategy is triggered.
when_to_use_negative:
- All off-diagonal entries of the cell-type correlation matrix P are below tau = 0.95
  — NeuCA instead selects the single feed-forward neural network classifier (model.size
  = small/medium/big)
- The new sample may contain unknown, rare, or novel cell types absent from the training
  reference — NeuCA is designed for well-studied situations where all cell types already
  exist in the training data; identify such cells first (e.g., inferCNV or copyKAT
  for malignant cells) before applying NeuCA
- No labeled training scRNA-seq data exist for the tissue of interest — supervised
  training is impossible and unsupervised methods are required
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: NeuCA
  role: Package whose hierarchical branch performs the complete-linkage clustering
    and tree-based classification; invoked with SingleCellExperiment train/test objects
    and a model.size argument
  repo: https://bioconductor.org/packages/NeuCA
- name: limma
  role: Linear model framework used to select the top 1000 bifurcation-specific genes
    at each node of the lineage tree
- name: keras
  role: API used to implement and train the binary feed-forward networks at each bifurcation
    of the tree
- name: SingleCellExperiment
  role: Bioconductor class used to format the training and testing scRNA-seq matrices
    (normcounts assay + cell labels in colData) required by NeuCA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:cfc7215902532609a46969b4cd451c5baa4afbb3e3194a5720cc3b3b0c6152c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  - SingleCellExperiment
  - TensorFlow
  - keras
  - limma
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# complete-linkage-hierarchical-clustering-cell-lineage-tree

## Summary

Builds the cell lineage tree used by NeuCA's hierarchical branch: complete-linkage hierarchical clustering on the columns (K cell types) of the averaged G x K gene-expression profile matrix A, defining the tree of binary bifurcations along which neural-network classifiers are trained when cell types are highly correlated.

## When to use

Use when the Pearson correlation matrix P over the training data's averaged cell-type profiles has off-diagonal entries >= 0.95, so a single feed-forward network is insufficient and the hierarchical, marker-guided strategy is triggered.

## When NOT to use

- All off-diagonal entries of the cell-type correlation matrix P are below tau = 0.95 — NeuCA instead selects the single feed-forward neural network classifier (model.size = small/medium/big)
- The new sample may contain unknown, rare, or novel cell types absent from the training reference — NeuCA is designed for well-studied situations where all cell types already exist in the training data; identify such cells first (e.g., inferCNV or copyKAT for malignant cells) before applying NeuCA
- No labeled training scRNA-seq data exist for the tissue of interest — supervised training is impossible and unsupervised methods are required

## Inputs

- Training scRNA-seq expression matrix with known cell-type labels as a SingleCellExperiment object (normcounts assay, cell labels in colData, feature_symbol in rowData, duplicated gene names removed)
- Testing scRNA-seq expression matrix as a SingleCellExperiment object (labels optional)
- Averaged G x K expression profile matrix A over the K training cell types

## Outputs

- Cell lineage tree (dendrogram) over the K training cell types from complete-linkage clustering
- Per-bifurcation feature sets (top 1000 limma-selected genes) and trained binary feed-forward classifiers along the tree
- Leaf-level predicted cell label for every test cell, written to cell_labels.csv (one of the K training labels per cell, no unassigned/intermediate labels)

## How to apply

First normalize the training matrix X0 and testing matrix X1 and subset both to their overlapped genes, then compute the averaged expression profile for each of the K cell types and the Pearson correlation matrix P; switch to this hierarchical strategy only when some off-diagonal correlations reach the tau = 0.95 threshold. Build the cell lineage tree by complete-linkage hierarchical clustering on the columns of the averaged G x K profile matrix A, so closely correlated cell types are forced into the same subtree and are resolved by binary decisions higher in the tree. Each internal bifurcation of the tree then selects its top M = 1000 bifurcation-specific features via limma differential analysis, and a binary feed-forward network with binary cross-entropy is trained at that node. Before tree traversal, highly sensitive markers (Sen_gk > 0.95) pre-assign cells expressing more than nu2 = 3 markers of any cell type with at least nu1 = 10 such markers, reducing the cell set that must traverse the tree. Rationale: hierarchical structure information of cell types improves annotation accuracy precisely for closely correlated cell types, and feature selection at each bifurcation further improves classification accuracy.

## Related tools

- **NeuCA** (Package whose hierarchical branch performs the complete-linkage clustering and tree-based classification; invoked with SingleCellExperiment train/test objects and a model.size argument) — https://bioconductor.org/packages/NeuCA
- **limma** (Linear model framework used to select the top 1000 bifurcation-specific genes at each node of the lineage tree)
- **keras** (API used to implement and train the binary feed-forward networks at each bifurcation of the tree)
- **SingleCellExperiment** (Bioconductor class used to format the training and testing scRNA-seq matrices (normcounts assay + cell labels in colData) required by NeuCA)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- Strategy-selection invariant: the hierarchical branch is only taken when at least one off-diagonal element of the Pearson correlation matrix P is >= 0.95; otherwise the single feed-forward network is used
- Every test cell receives exactly one of the K training-defined labels — no unassigned or intermediate (leaf-vs-node) labels appear in cell_labels.csv
- Each tree bifurcation is trained on its own top M = 1000 limma-selected bifurcation-specific features, with binary cross-entropy as the loss
- Marker pre-assignment respects the thresholds: only cell types with >= 10 markers of sensitivity > 0.95 are pre-assigned, and only cells expressing > 3 of those markers are assigned before traversal
- Confusion matrix of predicted vs. known test labels shows accuracy consistent with the reported NeuCA advantage (e.g., the README example: 328/331 alpha, 109/109 beta, 132/132 gamma)

## Limitations

- NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data; it does not decide whether novel or unknown cell types are present (consult biological/clinical experts)
- Cross-condition prediction was demonstrated on ASD data and may not hold in other scenarios
- keras, TensorFlow, and R version numbers are not specified in the software description (README session info shows R 4.0.3, keras_2.4.0, tensorflow_2.4.0)

## Evidence

- [methods] Hierarchical strategy is triggered by high correlation: "NeuCA first computes the averaged expression profile matrix for all the cell types in the training data"
- [readme] Hierarchical branch rationale: "When closely correlated cell types exist, NeuCA uses the cell type tree information through a hierarchical structure of neural networks to improve annotation accuracy."
- [readme] Feed-forward alternative: "When cell type correlations are not high, a feed-forward neural network is adopted."
- [readme] Automatic detection of correlated cell types: "NeuCA can detect whether highly-correlated cell types exist in the training dataset"
- [methods] Correlation matrix computation: [evidence span withheld — anchor status: not_found]
