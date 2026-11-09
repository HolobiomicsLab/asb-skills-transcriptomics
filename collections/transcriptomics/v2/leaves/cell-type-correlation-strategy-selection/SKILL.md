---
name: cell-type-correlation-strategy-selection
description: Use when training a supervised single-cell label-assignment model on
  scRNA-seq data with known cell types and you must decide between a flat vs hierarchical
  classifier based on how highly correlated the training cell types are.
when_to_use_negative:
- The new sample is expected to contain cell types or rare/unknown populations absent
  from the reference/training data — NeuCA is designed for settings where all cell
  types in the new sample already exist in the training data
- No labeled training scRNA-seq data exist for the tissue of interest, so an unsupervised
  clustering approach is required instead of supervised label assignment
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: Performs the correlation computation and automatic strategy selection (feed-forward
    vs hierarchical neural network) during training and prediction
  repo: https://bioconductor.org/packages/NeuCA
- name: keras
  role: R API used by NeuCA to implement the feed-forward and hierarchical neural
    network classifiers
- name: SingleCellExperiment
  role: Bioconductor data container required for NeuCA training and testing inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:c9e68c4f45221319350cd794372f17b84a55fe9ffac5b2fca4d708f686fe0408
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  - SingleCellExperiment
  - keras
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cell-type-correlation-strategy-selection

## Summary

NeuCA's automatic classifier-strategy selection: compute averaged expression profiles per cell type in the training data, calculate the Pearson correlation matrix across cell types, and if any pairwise off-diagonal correlation exceeds the threshold tau = 0.95, use a cell lineage-based hierarchical neural network (with marker-guided feature selection); otherwise use a single feed-forward neural network.

## When to use

Use when training a supervised single-cell label-assignment model on scRNA-seq data with known cell types and you must decide between a flat vs hierarchical classifier based on how highly correlated the training cell types are.

## When NOT to use

- The new sample is expected to contain cell types or rare/unknown populations absent from the reference/training data — NeuCA is designed for settings where all cell types in the new sample already exist in the training data
- No labeled training scRNA-seq data exist for the tissue of interest, so an unsupervised clustering approach is required instead of supervised label assignment

## Inputs

- Training scRNA-seq count/expression matrix with known cell-type labels, as a Bioconductor SingleCellExperiment object (normcounts assay, colData labels, rowData$feature_symbol)
- Testing scRNA-seq matrix as a SingleCellExperiment object, normalized and subset to genes overlapping the training matrix
- Averaged per-cell-type expression profiles and their Pearson correlation matrix (computed internally by NeuCA)

## Outputs

- A chosen classification strategy: feed-forward neural network or cell lineage-based hierarchical neural network
- Predicted cell-type labels for every cell in the testing dataset (vector of predicted labels)
- Per-method annotation accuracies when benchmarked against scmap-cluster, scmap-cell, and CHETAH (e.g. benchmark_accuracy_table.csv)

## How to apply

After normalizing the expression matrix and subsetting training and testing matrices to their overlapped genes, compute the averaged expression profile for each cell type in the training data and build the Pearson correlation matrix P across cell types. Inspect the off-diagonal elements of P: if all are below the threshold tau = 0.95, train a feed-forward neural network on all features; if any pairwise correlation is high (e.g. closely related PBMC subsets in Zheng_8ct_PBMC or FACS-sorted monocyte/NK data), invoke the hierarchical strategy, which uses cell-type tree/lineage structure to split the problem and performs feature selection at each hierarchy level. In practice NeuCA performs this decision automatically when called, and the same logic applies across intra-study and inter-study train/test combinations (e.g. training on one FACS-sorted PBMC dataset and testing on another). This selection is the mechanism that lets NeuCA handle closely correlated cell types without hand-designed marker genes.

## Related tools

- **NeuCA** (Performs the correlation computation and automatic strategy selection (feed-forward vs hierarchical neural network) during training and prediction) — https://bioconductor.org/packages/NeuCA
- **keras** (R API used by NeuCA to implement the feed-forward and hierarchical neural network classifiers)
- **SingleCellExperiment** (Bioconductor data container required for NeuCA training and testing inputs)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- Inter-study annotation accuracy on FACS-sorted PBMC datasets exceeds 98% (train on one dataset, test on a different independently collected PBMC dataset)
- The correlation matrix P of averaged cell-type profiles shows off-diagonal elements compared against the tau = 0.95 threshold, and the chosen strategy (feed-forward vs hierarchical) matches that comparison
- Accuracy comparison against scmap-cluster, scmap-cell, and CHETAH shows NeuCA outperforming these methods on the same train/test combinations
- Predicted.label vector length equals the number of cells in the testing SingleCellExperiment object, and a confusion matrix against true labels shows few off-diagonal assignments

## Limitations

- NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data
- Deciding whether unknown/novel cell types exist in the data is not addressed by the method; expert consultation is recommended
- No version numbers are given for keras, TensorFlow, or R in the software description
- Existing label-assignment methods (which NeuCA benchmarks against) still label a considerable number of cells as unassigned even when unknown cell types are not expected

## Evidence

- [methods] Correlation threshold tau=0.95 decides strategy: [evidence span withheld — anchor status: not_found]
- [intro] Correlation computation precedes strategy choice: "NeuCA first obtains the mean gene expression profile for each cell type and calculates the correlation matrix across cell types"
- [abstract] Hierarchical structure rationale: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
- [readme] Single-call usage in README: "NeuCA can detect whether highly-correlated cell types exist in the training dataset, and automatically determine if a general neural-network model will be adopted or a marker-guided hierarchical"
- [discussion] Restriction to known cell types: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
