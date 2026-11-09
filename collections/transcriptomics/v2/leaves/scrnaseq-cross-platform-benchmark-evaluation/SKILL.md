---
name: scrnaseq-cross-platform-benchmark-evaluation
description: Use when several labeled scRNA-seq studies of the same tissue exist and
  you need to quantify how annotation accuracy transfers between studies/platforms
  (inter-study annotation) across competing classifiers.
when_to_use_negative:
- Cell types in the test sample may be novel or not present in the reference data
  — NeuCA assumes all cell types in the new sample already exist in the training data
- No labeled training data exist for the tissue of interest, so only unsupervised
  clustering is viable
- The test data still contains unknown/rare or malignant cells that need prior identification
  (e.g., inferCNV/copyKAT) rather than exhaustive label assignment
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: NeuCA
  role: Supervised neural-network classifier trained on one pancreas study and used
    to predict cell labels on the other three; one-line train/test call with model.size
    argument
  repo: https://bioconductor.org/packages/NeuCA/
- name: scmap
  role: Baseline annotation method (scmap-cluster and scmap-cell modes) run with the
    same training dataset as reference for accuracy comparison
- name: CHETAH
  role: Baseline hierarchical annotation method run with the same reference for accuracy
    comparison
- name: R
  role: Implementation environment; NeuCA is an R/Bioconductor package depending on
    keras, limma, SingleCellExperiment, and e1071
  repo: https://bioconductor.org/packages/NeuCA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:18cf73e72d5832dfade93f7be4fab9d8a28e3e8eb4a468e12ae9b6515c1e5171
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - NeuCA
  - R
  - scmap
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# scrnaseq-cross-platform-benchmark-evaluation

## Summary

A benchmarking protocol for comparing supervised scRNA-seq cell annotation methods (NeuCA, scmap-cluster, scmap-cell, CHETAH) across datasets from different studies/platforms, using one dataset as training reference and the others as independent tests with published labels as ground truth.

## When to use

Use when several labeled scRNA-seq studies of the same tissue exist and you need to quantify how annotation accuracy transfers between studies/platforms (inter-study annotation) across competing classifiers.

## When NOT to use

- Cell types in the test sample may be novel or not present in the reference data — NeuCA assumes all cell types in the new sample already exist in the training data
- No labeled training data exist for the tissue of interest, so only unsupervised clustering is viable
- The test data still contains unknown/rare or malignant cells that need prior identification (e.g., inferCNV/copyKAT) rather than exhaustive label assignment

## Inputs

- Four labeled human pancreas scRNA-seq count matrices: Baron (GSE84133), Xin (GSE81608), Muraro (GSE85241), Seg (E-MTAB-5061)
- Published true cell type labels for each dataset
- SingleCellExperiment objects (normcounts assay, feature_symbol in rowData, labels in colData) for each train/test pair

## Outputs

- Per-cell predicted labels for each test dataset from each method
- A 12 x 4 annotation accuracy summary table (e.g., pancreas_cross_platform_benchmark.csv)
- Optional confusion matrices / Sankey diagrams per train-test scenario

## How to apply

Download the four human pancreas datasets (Baron GSE84133, Xin GSE81608, Muraro GSE85241 from GEO; Seg E-MTAB-5061 from EMBL-EBI) with published cell labels as ground truth. For each of the 12 train-test combinations, preprocess by normalizing expression matrices and subsetting to the genes overlapped between training and testing data, then format train and test as SingleCellExperiment objects with normcounts and true labels in colData, removing duplicated feature symbols. Train NeuCA on the reference (it computes per-cell-type mean expression profiles, builds the Pearson correlation matrix, and applies tau = 0.95 to choose between a feed-forward or cell lineage-based hierarchical neural network) and run scmap-cluster, scmap-cell, and CHETAH with the same reference. Compute annotation accuracy for each method in each scenario by comparing predicted labels against the published true cell type labels, then assemble the 12 x 4 accuracy table (e.g., pancreas_cross_platform_benchmark.csv).

## Related tools

- **NeuCA** (Supervised neural-network classifier trained on one pancreas study and used to predict cell labels on the other three; one-line train/test call with model.size argument) — https://bioconductor.org/packages/NeuCA/
- **scmap** (Baseline annotation method (scmap-cluster and scmap-cell modes) run with the same training dataset as reference for accuracy comparison)
- **CHETAH** (Baseline hierarchical annotation method run with the same reference for accuracy comparison)
- **R** (Implementation environment; NeuCA is an R/Bioconductor package depending on keras, limma, SingleCellExperiment, and e1071) — https://bioconductor.org/packages/NeuCA

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- NeuCA attains the highest accuracy in 9 of the 12 pancreas train-test scenarios
- On the Baron->Seg transfer NeuCA correctly labels more than 90% of cells while scmap-cluster, scmap-cell, and CHETAH reach at most 80%
- Confusion matrix diagonals dominate (README example: 328/331 alpha, 109/109 beta, 132/132 gamma correct on Baron->Seg)
- All 12 train-test combinations produce accuracy values for all four methods (no missing cells in the summary table)

## Limitations

- NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data
- CHETAH can produce an unignorable amount of unassigned/intermediate labels when few or no unknown cell types are expected, complicating accuracy comparison
- Cross-condition prediction may not hold in scenarios other than the ASD data application demonstrated in the paper
- No version numbers given for keras, TensorFlow, or R in the software description, so exact reproducibility of the neural-network stack is not guaranteed

## Evidence

- [results] NeuCA achieves the highest accuracy in 9 of the 12 pancreas training-testing scenarios, and in the Baron->Seg transfer it correctly labels more than 90% of cells while competing methods annotate 80% or less correctly.: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing: normalize expression matrix and subset training and testing data by overlapped genes: [evidence span withheld — anchor status: not_found]
- [methods] Correlation threshold tau = 0.95 selects feed-forward vs hierarchical neural network: [evidence span withheld — anchor status: not_found]
- [readme] README example invocation: NeuCA(train, test) returns predicted labels evaluable via confusion matrix against true labels: "predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, 
                        model.size = "big", verbose = FALSE)"
- [discussion] NeuCA assumes all cell types in the test sample exist in the reference: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
