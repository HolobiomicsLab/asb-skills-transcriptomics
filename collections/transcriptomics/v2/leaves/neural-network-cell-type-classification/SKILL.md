---
name: neural-network-cell-type-classification
description: Use when training cell labels exist for scRNA-seq reference data and
  every cell type in the test sample is expected to already exist in the reference,
  especially with closely correlated cell types or inter-study PBMC transfer.
when_to_use_negative:
- Novel or unstudied cell types may exist in the new sample — NeuCA is restricted
  to settings where all cell types in the new sample already exist in the reference/training
  data
- No labeled training data exist for the tissue of interest, so a supervised method
  cannot be trained (unsupervised clustering is the only option there)
- Identifying malignant or rare/unknown cells is the goal — pre-filter with inferCNV/copyKAT
  or rare-cell tools before applying NeuCA
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: NeuCA
  role: Trains the neural network classifier (feed-forward or hierarchical via correlation
    threshold tau=0.95) and predicts test cell labels
  repo: https://bioconductor.org/packages/NeuCA/
- name: R
  role: Environment for NeuCA (R/Bioconductor package) and the benchmarking workflow
    on PBMC datasets
- name: scmap
  role: Competing label-assignment method (scmap-cluster and scmap-cell) run on the
    same training data for accuracy comparison
- name: CHETAH
  role: Competing label-assignment method used as baseline in intra- and inter-study
    annotation benchmarks
- name: SingleCellExperiment
  role: Bioconductor class used to format NeuCA's required training and testing inputs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:e635207c2c5120fcb6900b73a59e0f318fffd08cdef27e05289520d7db5d2a9c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - NeuCA
  - R
  - SingleCellExperiment
  - scmap
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# neural-network-cell-type-classification

## Summary

Train a neural network-based classifier (NeuCA) on labeled scRNA-seq reference data and predict cell labels in new data, choosing between a feed-forward network and a cell lineage-based hierarchical neural network according to the Pearson correlation of averaged cell type profiles.

## When to use

Use when training cell labels exist for scRNA-seq reference data and every cell type in the test sample is expected to already exist in the reference, especially with closely correlated cell types or inter-study PBMC transfer.

## When NOT to use

- Novel or unstudied cell types may exist in the new sample — NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data
- No labeled training data exist for the tissue of interest, so a supervised method cannot be trained (unsupervised clustering is the only option there)
- Identifying malignant or rare/unknown cells is the goal — pre-filter with inferCNV/copyKAT or rare-cell tools before applying NeuCA

## Inputs

- Training scRNA-seq count matrix with known cell type labels (SingleCellExperiment with normcounts assay and colData labels)
- Testing scRNA-seq count matrix as a SingleCellExperiment object (labels optional, used only for evaluation)
- Overlapped gene subset shared by training and testing matrices
- Reference datasets such as Zheng_8ct_PBMC (DuoClustering2018), FACS-sorted NK data (GSE130430), FACS-sorted monocytes (GSE103544)

## Outputs

- predicted.label: vector of predicted cell type per test cell (same length as test cells)
- Per-method annotation accuracy (NeuCA vs scmap-cluster, scmap-cell, CHETAH) per train/test combination, e.g., benchmark_accuracy_table.csv
- Optional confusion matrix of predicted vs true labels and Sankey diagram

## How to apply

Format training and testing count matrices as SingleCellExperiment objects with 'normcounts' assays, cell labels in colData, gene names in rowData$feature_symbol, and duplicated gene names removed. NeuCA computes the averaged expression profile per cell type, builds the Pearson correlation matrix P across cell types, and applies the correlation-based strategy selection with threshold tau = 0.95: a feed-forward neural network when all off-diagonal correlations are below tau, otherwise a marker-guided, cell lineage-based hierarchical neural network with feature selection at each level. Train on the reference (e.g., Zheng_8ct_PBMC from DuoClustering2018, FACS-sorted NK data GSE130430, or monocytes GSE103544) and predict labels on the testing matrix subset to overlapped genes, comparing accuracy against scmap-cluster, scmap-cell, and CHETAH. Choose model.size among 'big', 'medium', or 'small' to set the number of nodes and layers.

## Related tools

- **NeuCA** (Trains the neural network classifier (feed-forward or hierarchical via correlation threshold tau=0.95) and predicts test cell labels) — https://bioconductor.org/packages/NeuCA/
- **R** (Environment for NeuCA (R/Bioconductor package) and the benchmarking workflow on PBMC datasets)
- **scmap** (Competing label-assignment method (scmap-cluster and scmap-cell) run on the same training data for accuracy comparison)
- **CHETAH** (Competing label-assignment method used as baseline in intra- and inter-study annotation benchmarks)
- **SingleCellExperiment** (Bioconductor class used to format NeuCA's required training and testing inputs)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- NeuCA achieves >98% accuracy when trained on one FACS-sorted PBMC dataset and tested on a different independently collected PBMC dataset (inter-study)
- NeuCA accuracy outperforms scmap-cluster, scmap-cell, and CHETAH in intra-study predictions
- Predicted labels form a valid cell-type vector with length equal to the number of test cells and levels matching training cell types (e.g., table(predicted.label) sums to test size)
- Confusion matrix diagonal dominates: e.g., README example shows 328/331 alpha, 109/109 beta, 132/132 gamma correctly assigned
- Correlation-based strategy selection is exercised: all off-diagonal Pearson correlations < 0.95 trigger the feed-forward model, any >= 0.95 triggers the hierarchical model

## Limitations

- NeuCA is designed for situations where all cell types in the new sample already exist in the reference/training data; it does not detect novel or unknown cell types
- It does not decide whether unknown/novel cell types exist in the data; expert consultation is recommended for novel conditions
- Existing label-assignment methods (context for comparison) can label many cells as unassigned even when no unknown types are expected — verify no unassigned labels are acceptable in your setting
- Cross-condition prediction was demonstrated on ASD data but may not hold in other scenarios
- No version numbers are given for keras, TensorFlow, or R in the software description, complicating reproducibility
- A pre-trained classifier is only provided for human PBMC data; other tissues require training a new model

## Evidence

- [methods] Correlation-based strategy selection (threshold tau = 0.95): feed-forward network vs hierarchical network: [evidence span withheld — anchor status: not_found]
- [other] Inter-study accuracy finding: [evidence span withheld — anchor status: not_found]
- [abstract] Hierarchical rationale for closely correlated types: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
- [readme] SingleCellExperiment input preparation and one-line training/prediction: "with both training and testing data as objects in SingleCellExperiment class, now we can train the classifier in NeuCA and predict testing dataset’s cell types"
- [discussion] Scope restriction on known cell types: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
