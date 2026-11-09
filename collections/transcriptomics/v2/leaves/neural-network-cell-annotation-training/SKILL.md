---
name: neural-network-cell-annotation-training
description: Use when labeled training scRNA-seq data exist for the tissue and every
  cell type in the test sample is expected to be present in the reference, and exhaustive
  cell label assignment is needed.
when_to_use_negative:
- Unknown or novel cell types may exist in the test sample — NeuCA is restricted to
  settings where all cell types already exist in the reference/training data
- No labeled training data exist for the tissue of interest — unsupervised methods
  like Seurat or SC3 are the only viable option
- Malignant cells are present in tumor samples — identify them first with inferCNV
  or copyKAT, then apply NeuCA to the remaining cells
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: Trains the neural-network classifier (feed-forward or hierarchical, tau=0.95
    correlation threshold) and predicts test cell labels
  repo: https://bioconductor.org/packages/NeuCA
- name: Seurat
  role: Unsupervised clustering baseline for benchmarking annotation accuracy via
    ARI
- name: SC3
  role: Unsupervised clustering baseline for benchmarking annotation accuracy via
    ARI
- name: keras
  role: API underlying the neural-network implementation (ReLU activation, RMSprop
    optimizer, mini-batch training)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:23efb6d4dea30c0147781ae4755488860901973d79de3b766c851da3293837fa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  - SC3
  - Seurat
  - keras
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# neural-network-cell-annotation-training

## Summary

Train NeuCA, a supervised neural-network classifier, on labeled scRNA-seq data to exhaustively assign cell labels to new data, automatically choosing a feed-forward network or a marker-guided hierarchical network based on cell-type correlation.

## When to use

Use when labeled training scRNA-seq data exist for the tissue and every cell type in the test sample is expected to be present in the reference, and exhaustive cell label assignment is needed.

## When NOT to use

- Unknown or novel cell types may exist in the test sample — NeuCA is restricted to settings where all cell types already exist in the reference/training data
- No labeled training data exist for the tissue of interest — unsupervised methods like Seurat or SC3 are the only viable option
- Malignant cells are present in tumor samples — identify them first with inferCNV or copyKAT, then apply NeuCA to the remaining cells

## Inputs

- Training scRNA-seq count matrix with known cell type labels (SingleCellExperiment, normcounts assay)
- Testing scRNA-seq count matrix (SingleCellExperiment, labels optional)
-  rowData$feature_symbol gene names with duplicated features removed

## Outputs

- predicted.label: vector of predicted cell type per test cell
- Confusion matrix of predicted vs. true labels and accurately assigned rate
- ARI when benchmarking against Seurat/SC3 clustering baselines

## How to apply

Prepare training and testing data as Bioconductor SingleCellExperiment objects with 'normcounts' assays, cell labels in colData, and gene symbols in rowData$feature_symbol (removing duplicated features). Preprocess by normalizing the expression matrix and subsetting training and testing data to overlapped genes. NeuCA then computes the averaged expression profile per cell type and the Pearson correlation matrix across cell types; if all off-diagonal correlations are below tau = 0.95 it uses a feed-forward neural network, otherwise a cell lineage-based hierarchical neural network with feature selection. Call NeuCA(train = ..., test = ..., model.size = 'big') to train (ReLU, RMSprop, mini-batch via keras) and predict in one step. Evaluate with a confusion matrix of predicted vs. true labels or accurately assigned rate; compare against unsupervised baselines such as Seurat and SC3 using ARI.

## Related tools

- **NeuCA** (Trains the neural-network classifier (feed-forward or hierarchical, tau=0.95 correlation threshold) and predicts test cell labels) — https://bioconductor.org/packages/NeuCA
- **Seurat** (Unsupervised clustering baseline for benchmarking annotation accuracy via ARI)
- **SC3** (Unsupervised clustering baseline for benchmarking annotation accuracy via ARI)
- **keras** (API underlying the neural-network implementation (ReLU activation, RMSprop optimizer, mini-batch training))

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- NeuCA's accurately assigned rate on PBMC simulations rises from about 0.86 to 0.90 as training proportion increases from 10% to 80%
- As testing size grows from 800 to 4000 cells, Seurat's ARI rises from about 0.45 to 0.5 and SC3's from about 0.60 to 0.65, while NeuCA remains the highest performer
- Confusion matrix diagonal should dominate: e.g. 328/331 alpha cells correctly assigned in the README example
- Strategy check: off-diagonal Pearson correlations of cell-type mean profiles below tau = 0.95 must route to the feed-forward network, higher correlations to the hierarchical network
- SingleCellExperiment objects must have unique feature symbols and matched gene sets (overlapped genes) between train and test

## Limitations

- NeuCA is designed for situations where all cell types in the new sample already exist in the reference/training data; rare or unknown cells should be identified first with dedicated tools before applying NeuCA
- Existing label assignment methods label a considerable number of cells as unassigned even when unknown cell types are not expected; NeuCA avoids this but requires exhaustive reference coverage
- No version numbers are given for keras, TensorFlow, or R in the software description; cross-condition prediction was demonstrated only on the ASD data application and may not hold in other scenarios
- model.size argument ('big', 'medium', 'small') controls network complexity and must be chosen by the user

## Evidence

- [methods] Strategy selection via correlation threshold: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing requirement: [evidence span withheld — anchor status: not_found]
- [abstract] Hierarchical rationale: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
- [readme] README: one-line training and prediction: "predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, 
                        model.size = "big", verbose = FALSE)"
- [readme] README: input formatting requirement: [evidence span withheld — anchor status: external_doc]
- [discussion] Scope limitation: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
