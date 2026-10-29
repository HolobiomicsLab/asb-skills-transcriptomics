---
name: neural-network-classifier-training-scrnaseq
description: Use when you have scRNA-seq training data with known cell labels (e.g.
  Baron/Muraro/Seg/Xin pancreas or 10X PBMC) and need to assign labels to testing
  cells in an exhaustive supervised setting.
when_to_use_negative:
- The new sample may contain unknown/novel cell types absent from the reference/training
  data — NeuCA is designed for settings where all cell types in the new sample already
  exist in the reference data
- No labeled training data exist for the tissue of interest, where unsupervised clustering
  (e.g. Seurat, SC3) is the only viable option
- Tumor samples with possible malignant cells — identify malignant cells first with
  inferCNV or copyKAT before applying NeuCA to the remaining cells
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: Core supervised neural-network classifier for exhaustive cell label assignment
    of scRNA-seq data
  repo: https://bioconductor.org/packages/NeuCA
- name: R
  role: Runtime environment; NeuCA is an R/Bioconductor package
- name: scmap
  role: Baseline supervised annotation method (scmap-cluster, scmap-cell) used in
    benchmark comparisons
- name: CHETAH
  role: Baseline supervised hierarchical annotation method used in benchmark comparisons
- name: Seurat
  role: Baseline unsupervised clustering method used in benchmark comparisons
- name: SC3
  role: Baseline unsupervised clustering method used in benchmark comparisons
provenance:
  source_task_ids:
  - task_007
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:b35875942d347d08e0e5d179d6f2450a0f69347b59e5cb8b604dd6a45e1db47e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - NeuCA
  - R
  - SC3
  - Seurat
  - scmap
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# neural-network-classifier-training-scrnaseq

## Summary

Train NeuCA, an R/Bioconductor neural-network-based supervised classifier, on scRNA-seq data with known cell labels and use it to exhaustively assign cell types to new scRNA-seq data, automatically switching between a feed-forward network and a marker-guided hierarchical network depending on cell-type correlation.

## When to use

Use when you have scRNA-seq training data with known cell labels (e.g. Baron/Muraro/Seg/Xin pancreas or 10X PBMC) and need to assign labels to testing cells in an exhaustive supervised setting.

## When NOT to use

- The new sample may contain unknown/novel cell types absent from the reference/training data — NeuCA is designed for settings where all cell types in the new sample already exist in the reference data
- No labeled training data exist for the tissue of interest, where unsupervised clustering (e.g. Seurat, SC3) is the only viable option
- Tumor samples with possible malignant cells — identify malignant cells first with inferCNV or copyKAT before applying NeuCA to the remaining cells

## Inputs

- Training scRNA-seq expression matrix with known cell-type labels (e.g. Baron GSE84133, Xin GSE81608, Muraro GSE85241, Seg E-MTAB-5061, or Zheng_8ct_PBMC from DuoClustering2018)
- Testing scRNA-seq expression matrix (labels optional, only needed for evaluation)
- SingleCellExperiment objects with normcounts assay, colData labels, and rowData$feature_symbol

## Outputs

- Per-cell predicted cell-type label vector for the testing dataset (e.g. predicted.label)
- Confusion matrix of predicted vs true labels and accurately assigned rate
- Benchmark accuracy/ARI tables (e.g. pancreas_cross_platform_benchmark.csv, asd_crosscondition_benchmark.csv, benchmark_results.csv)

## How to apply

Prepare training and testing matrices as SingleCellExperiment objects with normalized counts (normcounts assay), cell labels in colData, and gene names set as rowData feature_symbol after removing duplicated features. NeuCA then computes the averaged expression profile per cell type, builds the Pearson correlation matrix across cell types, and at the tau = 0.95 threshold automatically selects either a feed-forward neural network (all off-diagonal correlations below tau) or a marker-guided, cell-lineage-based hierarchical neural network for closely correlated cell types; networks use keras with ReLU activation and RMSprop, at model sizes 'small' (3 layers/64 nodes), 'medium' (4 layers/64-128), or 'big' (5 layers/64-256). Run prediction in one call with NeuCA(train, test, model.size = ...), which returns a per-cell predicted-label vector. Evaluate with a confusion matrix against true labels (accurately assigned rate) and the Adjusted Rand Index (ARI), as done in the 12 pancreas cross-study scenarios and the ASD cross-condition benchmark versus scmap-cluster, scmap-cell, CHETAH, Seurat, and SC3.

## Related tools

- **NeuCA** (Core supervised neural-network classifier for exhaustive cell label assignment of scRNA-seq data) — https://bioconductor.org/packages/NeuCA
- **R** (Runtime environment; NeuCA is an R/Bioconductor package)
- **scmap** (Baseline supervised annotation method (scmap-cluster, scmap-cell) used in benchmark comparisons)
- **CHETAH** (Baseline supervised hierarchical annotation method used in benchmark comparisons)
- **Seurat** (Baseline unsupervised clustering method used in benchmark comparisons)
- **SC3** (Baseline unsupervised clustering method used in benchmark comparisons)

## Examples

```
library(NeuCA); predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- In the 12 pancreas train-test scenarios, NeuCA attains the highest accuracy in 9 of 12 and exceeds 90% accuracy on Baron->Seg while scmap-cluster, scmap-cell, and CHETAH reach at most 80%
- On 10X PBMC Monte Carlo simulations (160 runs), NeuCA-med exceeds scmap-cluster (best existing method) by about 10 percentage points in accurately assigned rate and NeuCA-small by at least 5 percentage points
- Confusion matrix of predicted.label vs true labels shows near-diagonal structure with negligible off-diagonal misassignments (README example: 328/331 alpha, 109/109 beta correct)
- In ASD cross-condition annotation, NeuCA achieves the highest accuracy and ARI in both train directions and scmap-cell is second-best supervised method with accurately assigned rate around 0.7
- table(predicted.label) length matches the number of test cells and labels are drawn only from the training set's cell types (exhaustive assignment, no 'unassigned' class)

## Limitations

- NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data; it cannot detect novel cell types
- Cross-condition prediction (demonstrated on ASD snRNA-seq) may not hold in scenarios other than the ASD application
- Existing label assignment methods (and NeuCA's setting assumes exhaustive labels) may mislead when unknown cell types are present; expert consultation is recommended to decide whether unknown types exist
- No version numbers are given for keras, TensorFlow, or R in the software description; identify unknown/rare cells with dedicated tools before applying NeuCA

## Evidence

- [abstract] NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
- [intro] NeuCA first obtains the mean gene expression profile for each cell type and calculates the correlation matrix across cell types: "NeuCA first obtains the mean gene expression profile for each cell type and calculates the correlation matrix across cell types"
- [methods] In the cases where all the off-diagonal elements in P are smaller than a threshold of: [evidence span withheld — anchor status: not_found]
- [readme] predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE): "predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, 
                        model.size = "big", verbose = FALSE)"
- [discussion] NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data.: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
