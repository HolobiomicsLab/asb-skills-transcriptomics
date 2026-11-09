---
name: cell-type-annotation-accuracy-comparison
description: Use when a labeled reference scRNA-seq dataset (e.g., GEO/E-MTAB pancreas,
  PBMC) must be used to annotate one or more independent testing studies and per-method
  annotation accuracy needs to be quantified across all train-test combinations.
when_to_use_negative:
- The new sample may contain cell types absent from the reference/training data —
  NeuCA is restricted to settings where all cell types in the new sample already exist
  in the training data
- No labeled training data exist for the tissue of interest, so a supervised method
  cannot be trained and unsupervised approaches are the only viable solution
- The data likely contain rare/unknown or malignant cells that should be removed first
  (e.g., with inferCNV or copyKAT) before exhaustive label assignment
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: Supervised neural-network classifier trained on the reference study to annotate
    testing studies; auto-selects feed-forward vs hierarchical model via correlation
    threshold
  repo: https://bioconductor.org/packages/NeuCA
- name: scmap
  role: Baseline annotation method run with the same training dataset as reference
    (scmap-cluster and scmap-cell variants)
- name: CHETAH
  role: Baseline hierarchical annotation method compared against NeuCA on identical
    train-test splits
- name: R
  role: 'Environment in which all preprocessing, training, prediction, and accuracy
    computation are performed (R/Bioconductor packages: keras, limma, SingleCellExperiment,
    e1071)'
  repo: https://bioconductor.org/packages/NeuCA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:757ac17656866a2f8c3d439da9086cb95435d24bcf066b2b4fc95cfcc8e9b52d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - NeuCA
  - R
  - scmap
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cell-type-annotation-accuracy-comparison

## Summary

Benchmark supervised cell-type annotation accuracy across cross-study scRNA-seq train/test scenarios (e.g., the four human pancreas datasets Baron, Muraro, Seg, Xin) by training NeuCA alongside scmap-cluster, scmap-cell, and CHETAH on the same reference and comparing predicted labels to published ground truth.

## When to use

Use when a labeled reference scRNA-seq dataset (e.g., GEO/E-MTAB pancreas, PBMC) must be used to annotate one or more independent testing studies and per-method annotation accuracy needs to be quantified across all train-test combinations.

## When NOT to use

- The new sample may contain cell types absent from the reference/training data — NeuCA is restricted to settings where all cell types in the new sample already exist in the training data
- No labeled training data exist for the tissue of interest, so a supervised method cannot be trained and unsupervised approaches are the only viable solution
- The data likely contain rare/unknown or malignant cells that should be removed first (e.g., with inferCNV or copyKAT) before exhaustive label assignment

## Inputs

- Human pancreas scRNA-seq count matrices with published cell type labels: Baron (GSE84133), Xin (GSE81608), Muraro (GSE85241), Seg (E-MTAB-5061)
- Training data as a SingleCellExperiment object (normcounts assay, colData with true labels, rowData$feature_symbol, duplicated features removed)
- Testing data as a SingleCellExperiment object with the same gene space as the training set (overlapped genes only)

## Outputs

- predicted.label vector of per-cell annotations for each testing dataset
- Confusion matrix of predicted vs. true cell labels per scenario
- Accuracy summary table, e.g. pancreas_cross_platform_benchmark.csv (12 train-test scenarios x 4 methods)
- Optional Sankey diagram visualizing prediction accuracy

## How to apply

Download each dataset with its published cell labels as ground truth (e.g., Baron GSE84133, Xin GSE81608, Muraro GSE85241 from GEO; Seg E-MTAB-5061 from EMBL-EBI), then normalize expression matrices and subset each train/test pair to the overlapped genes. For every train-test combination, package both datasets as Bioconductor SingleCellExperiment objects and call NeuCA, which computes per-cell-type mean expression profiles, builds the Pearson correlation matrix, and applies the tau = 0.95 threshold to select a feed-forward or cell lineage-based hierarchical neural network classifier. Run scmap-cluster, scmap-cell, and CHETAH in parallel with the same training data as reference on the same testing data. Compare each method's predicted labels to the published true labels (e.g., via confusion matrix) to compute accuracy for each of the 12 scenarios, and assemble the 12 x 4 results into a summary table such as pancreas_cross_platform_benchmark.csv.

## Related tools

- **NeuCA** (Supervised neural-network classifier trained on the reference study to annotate testing studies; auto-selects feed-forward vs hierarchical model via correlation threshold) — https://bioconductor.org/packages/NeuCA
- **scmap** (Baseline annotation method run with the same training dataset as reference (scmap-cluster and scmap-cell variants))
- **CHETAH** (Baseline hierarchical annotation method compared against NeuCA on identical train-test splits)
- **R** (Environment in which all preprocessing, training, prediction, and accuracy computation are performed (R/Bioconductor packages: keras, limma, SingleCellExperiment, e1071)) — https://bioconductor.org/packages/NeuCA

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- NeuCA attains the highest annotation accuracy in 9 of the 12 pancreas training-testing scenarios
- On the Baron->Seg transfer, NeuCA correctly labels more than 90% of cells while scmap-cluster, scmap-cell, and CHETAH annotate at most 80% correctly
- NeuCA produces no unassigned/intermediate labels when all cell types are expected to be known, unlike CHETAH
- Confusion matrix diagonals dominate: e.g., in the Baron->Seg example, 328/331 alpha, 109/109 beta, 56/56 delta, 64/65 ductal, 9/9 endothelial, 132/132 gamma cells are correctly assigned
- All 12 x 4 accuracy entries are populated in pancreas_cross_platform_benchmark.csv with each accuracy in [0,1]

## Limitations

- NeuCA is designed for situations where all cell types in the new sample already exist in the reference/training data; unknown/novel cell types are not handled
- Existing label assignment methods, including CHETAH, may still label a considerable number of cells as unassigned even when unknown cell types are not expected
- No version numbers are given for keras, TensorFlow, or R in the software description, complicating exact reproduction
- Cross-condition prediction success (shown on ASD data) may not hold in scenarios other than that application

## Evidence

- [methods] NeuCA first computes the averaged expression profile matrix for all the cell types in the training data: "NeuCA first computes the averaged expression profile matrix for all the cell types in the training data"
- [methods] tau threshold selects network type: [evidence span withheld — anchor status: not_found]
- [methods] preprocessing steps: [evidence span withheld — anchor status: not_found]
- [readme] README one-line training/prediction: "predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, 
                        model.size = "big", verbose = FALSE)"
- [readme] README SingleCellExperiment input format: [evidence span withheld — anchor status: external_doc]
