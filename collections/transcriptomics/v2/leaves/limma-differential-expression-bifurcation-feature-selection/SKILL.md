---
name: limma-differential-expression-bifurcation-feature-selection
description: Use when training a NeuCA cell-type classifier on scRNA-seq data whose
  cell-type Pearson correlation matrix contains off-diagonal entries at or above tau
  = 0.95, requiring the marker-guided hierarchical neural-network route.
when_to_use_negative:
- All off-diagonal cell-type Pearson correlations in P are below tau = 0.95 — NeuCA
  then uses a single feed-forward neural network instead of the hierarchical route
- The new sample may contain cell types absent from the reference/training data —
  NeuCA is restricted to exhaustive assignment of known types; identify unknown/rare
  cells first with other tools
- Good candidate marker genes are unavailable and supervised training labels are missing
  — supervised annotation is not applicable without training data
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: NeuCA
  role: Implements the correlation-gated two-route annotation loop; uses limma-selected
    bifurcation features for the hierarchical neural-network branch
  repo: https://bioconductor.org/packages/NeuCA
- name: limma
  role: Linear model framework and differential testing used to pick the top M = 1000
    bifurcation-specific features at each tree node
- name: keras
  role: R API to train the binary feed-forward networks with binary cross-entropy
    at each bifurcation and the main ReLU classifiers
- name: TensorFlow
  role: Backend for the keras neural-network training and softmax/binary predictions
- name: SingleCellExperiment
  role: Bioconductor class used to format the training and testing scRNA-seq inputs
    (normcounts assay + colData labels)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:4e3e60e7bfbf90e7f196cc5c30c6aeac11162f1def2ac664c31deda2331742d5
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# limma-differential-expression-bifurcation-feature-selection

## Summary

Selects bifurcation-specific marker genes with limma differential analysis at each node of a cell lineage tree, then trains binary feed-forward neural networks at each bifurcation — the hierarchical branch of NeuCA for highly correlated cell types (off-diagonal Pearson correlations ≥ 0.95).

## When to use

Use when training a NeuCA cell-type classifier on scRNA-seq data whose cell-type Pearson correlation matrix contains off-diagonal entries at or above tau = 0.95, requiring the marker-guided hierarchical neural-network route.

## When NOT to use

- All off-diagonal cell-type Pearson correlations in P are below tau = 0.95 — NeuCA then uses a single feed-forward neural network instead of the hierarchical route
- The new sample may contain cell types absent from the reference/training data — NeuCA is restricted to exhaustive assignment of known types; identify unknown/rare cells first with other tools
- Good candidate marker genes are unavailable and supervised training labels are missing — supervised annotation is not applicable without training data

## Inputs

- Training SingleCellExperiment object (normcounts assay + cell-type colData, e.g. Baron_scRNA)
- Testing SingleCellExperiment object (normcounts assay, labels optional, e.g. Seg_scRNA)
- Normalized expression matrices subset to overlapped genes
- Parameters: tau = 0.95 correlation threshold, Sen_gk > 0.95 marker sensitivity, nu1 = 10, nu2 = 3, M = 1000 features per bifurcation

## Outputs

- Cell lineage tree built by complete-linkage hierarchical clustering of averaged cell-type profiles
- Set of top 1000 bifurcation-specific limma-selected features per tree node
- Trained binary feed-forward network at each bifurcation (keras/TensorFlow)
- predicted.label vector assigning each test cell exactly one of the K training-defined labels (cell_labels.csv)

## How to apply

After normalizing expression and subsetting training (X0) and testing (X1) matrices to overlapped genes, compute the averaged G x K expression profile matrix A per cell type and the Pearson correlation matrix P; if any off-diagonal correlation reaches the tau = 0.95 threshold, switch from a single feed-forward network to the hierarchical strategy. Build the cell lineage tree by complete-linkage hierarchical clustering on the columns of A, then compute highly sensitive markers per cell type (Sen_gk > 0.95) and pre-assign cells expressing more than nu2 = 3 markers for cell types with at least nu1 = 10 such markers. At each tree bifurcation, run limma differential analysis to select the top M = 1000 bifurcation-specific features, and train a binary feed-forward ReLU network with binary cross-entropy (keras/TensorFlow, RMSprop, mini-batch) restricted to those features. Traverse remaining test cells through the tree and assign each cell the leaf label reached at the output layer.

## Related tools

- **NeuCA** (Implements the correlation-gated two-route annotation loop; uses limma-selected bifurcation features for the hierarchical neural-network branch) — https://bioconductor.org/packages/NeuCA
- **limma** (Linear model framework and differential testing used to pick the top M = 1000 bifurcation-specific features at each tree node)
- **keras** (R API to train the binary feed-forward networks with binary cross-entropy at each bifurcation and the main ReLU classifiers)
- **TensorFlow** (Backend for the keras neural-network training and softmax/binary predictions)
- **SingleCellExperiment** (Bioconductor class used to format the training and testing scRNA-seq inputs (normcounts assay + colData labels))

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- Every test cell receives exactly one of the K training-defined labels — no unassigned or intermediate labels (exhaustive-assignment criterion)
- Predicted.label vector length equals the number of cells in the testing dataset
- Confusion matrix of predicted.label vs. Seg_true_cell_label concentrates matches on the diagonal (e.g. alpha 328/331, beta 109/109, gamma 132/132 in the README example)
- Hierarchical branch triggers only when at least one off-diagonal entry of the Pearson correlation matrix P reaches tau = 0.95
- Each bifurcation's binary network is trained on exactly M = 1000 limma-selected features (or fewer if the gene space is smaller)

## Limitations

- NeuCA is designed for settings where all cell types in the new sample already exist in the reference/training data; unknown/novel type detection is out of scope
- Cross-condition prediction worked on the ASD data application but may not hold in other scenarios
- Existing label assignment methods label many cells as unassigned when unknown types are not expected; NeuCA avoids this but cannot decide whether unknown types exist — consult biological/clinical experts
- No version numbers are given for keras, TensorFlow, or R in the software description

## Evidence

- [methods] hierarchical branch with limma feature selection: [evidence span withheld — anchor status: not_found]
- [methods] correlation-gated strategy switch: [evidence span withheld — anchor status: not_found]
- [abstract] hierarchical structure motivation: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
- [readme] README: hierarchical route description: "When closely correlated cell types exist, NeuCA uses the cell type tree information through a hierarchical structure of neural networks to improve annotation accuracy. Feature selection is performed"
- [readme] README: invocation: "predicted.label = NeuCA(train = Baron_sce, test = Seg_sce,
                        model.size = "big", verbose = FALSE)"
- [discussion] exhaustive assignment scope: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
