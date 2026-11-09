---
name: highly-correlated-t-cell-subset-simulation
description: Use when benchmarking supervised cell-annotation methods on scRNA-seq
  data containing closely correlated cell types, or when constructing difficult vs
  easy prediction tasks from real PBMC data to stress-test hierarchical vs flat classifiers.
when_to_use_negative:
- The research question involves identifying novel or unknown cell types — NeuCA assumes
  all cell types in the new sample already exist in the training/reference data
- The data contain no closely correlated cell types and a simple feed-forward classifier
  would suffice (NeuCA's own correlation check would select the flat network, so the
  difficult-subset design adds no signal)
- Training data with known labels are unavailable — supervised methods like NeuCA
  outperform unsupervised ones only when training data exist
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: Primary annotation method under evaluation; auto-selects hierarchical neural
    network via Pearson correlation threshold tau = 0.95 for highly correlated T-cell
    types
  repo: https://bioconductor.org/packages/NeuCA
- name: scmap
  role: Baseline annotation method run on each difficult- and easy-setting split
- name: CHETAH
  role: Baseline annotation method whose unassigned-label behavior is quantified per
    cell type
- name: CellAssign
  role: Baseline marker-based annotation method run on each split
- name: SCINA
  role: Baseline marker-based annotation method run on each split
- name: R
  role: Implementation environment; NeuCA is an R/Bioconductor package
  repo: https://bioconductor.org/packages/NeuCA
- name: Seurat
  role: Referenced single-cell framework in the same ecosystem
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:30dddd2041ec6f9f26c8e332074b9429b50b4b3ad59cc18981d1a4945b71805a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CHETAH
  - CellAssign
  - NeuCA
  - R
  - SC3
  - SCINA
  - Seurat
  - TSCAN
  - scmap
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# highly-correlated-t-cell-subset-simulation

## Summary

A benchmark simulation design that constructs a difficult cell-annotation task by restricting real 10X Genomics PBMC data to the five highly correlated T-cell types, and an easy counterpart by merging them into a single 'T cells' class, to quantify how high inter-type correlation degrades annotation accuracy and ARI across methods.

## When to use

Use when benchmarking supervised cell-annotation methods on scRNA-seq data containing closely correlated cell types, or when constructing difficult vs easy prediction tasks from real PBMC data to stress-test hierarchical vs flat classifiers.

## When NOT to use

- The research question involves identifying novel or unknown cell types — NeuCA assumes all cell types in the new sample already exist in the training/reference data
- The data contain no closely correlated cell types and a simple feed-forward classifier would suffice (NeuCA's own correlation check would select the flat network, so the difficult-subset design adds no signal)
- Training data with known labels are unavailable — supervised methods like NeuCA outperform unsupervised ones only when training data exist

## Inputs

- Real PBMC scRNA-seq count matrix from the 10X Genomics website (normalized expression matrix with known cell-type labels)
- Training/testing splits restricted to the five T-cell types: CD4 T helper, memory T, naive cytotoxic T, naive T, regulatory T
- Training/testing splits with the five T-cell types merged into a single 'T cells' label among major cell types
- SingleCellExperiment objects for training and testing data (NeuCA input format)

## Outputs

- Per-method predicted cell labels for every difficult- and easy-setting split
- Per-method annotation accuracy and Adjusted Rand Index (ARI) for every split
- Fraction of 'unassigned' labels per method per cell type
- benchmark_results.csv containing all metrics

## How to apply

Download and preprocess real PBMC data from the 10X Genomics website, normalizing the expression matrix and subsetting training/testing matrices to their overlapped genes. Construct the difficult simulation by keeping only the five T-cell types (CD4 T helper, memory T, naive cytotoxic T, naive T, regulatory T), with training proportions p = 10%, 20%, 50%, 80% and test sizes n = 800, 1600, 4000. Run NeuCA, scmap, CHETAH, CellAssign, and SCINA on each split; NeuCA's internal Pearson-correlation check (threshold tau = 0.95) should select the hierarchical neural network for these highly correlated types. Build the easy simulation by merging all five T-cell types into a single 'T cells' label among the major cell types and repeat the same p/n splits. Compute per-method accuracy, Adjusted Rand Index (ARI), and the fraction of 'unassigned' labels per method per cell type, then write metrics to benchmark_results.csv.

## Related tools

- **NeuCA** (Primary annotation method under evaluation; auto-selects hierarchical neural network via Pearson correlation threshold tau = 0.95 for highly correlated T-cell types) — https://bioconductor.org/packages/NeuCA
- **scmap** (Baseline annotation method run on each difficult- and easy-setting split)
- **CHETAH** (Baseline annotation method whose unassigned-label behavior is quantified per cell type)
- **CellAssign** (Baseline marker-based annotation method run on each split)
- **SCINA** (Baseline marker-based annotation method run on each split)
- **R** (Implementation environment; NeuCA is an R/Bioconductor package) — https://bioconductor.org/packages/NeuCA
- **Seurat** (Referenced single-cell framework in the same ecosystem)

## Examples

```
predicted.label = NeuCA(train = Tcell5_sce, test = Tcell5_test_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- NeuCA leads all methods in accuracy and ARI under the difficult (five correlated T-cell types) setting
- Existing methods assign >50% 'unassigned' labels to naive T, CD4 T helper, and regulatory T cells in the difficult setting
- CD56 NK and CD14 monocytes are annotated correctly by all methods (sanity check that errors concentrate in correlated T-cell types)
- NeuCA's internal Pearson-correlation check triggers the hierarchical (not feed-forward) network for the five-T-cell-type subsets (off-diagonal correlation >= tau = 0.95)
- All splits and metrics are recorded in benchmark_results.csv with no missing method/split combinations

## Limitations

- NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data
- Existing label assignment methods label a considerable number of cells as unassigned even when unknown cell types are not expected
- Cross-condition prediction success (shown on ASD data) may not hold in other scenarios
- No version numbers given for keras, TensorFlow, or R in the software description, complicating exact reproduction

## Evidence

- [abstract] NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
- [methods] Correlation-based strategy selection threshold: [evidence span withheld — anchor status: not_found]
- [methods] Real PBMC data source: "Real PBMC data from the 10X Genomics website (https://support.10xgenomics.com/)31 were downloaded and processed to generate our simulation datasets"
- [discussion] NeuCA restriction to known cell types: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
- [abstract] Unassigned label problem: "when unknown cell types are not expected, existing methods still label a considerable number of cells as unassigned, which is not desirable"
- [readme] NeuCA auto-detection of correlated types: "NeuCA can detect whether highly-correlated cell types exist in the training dataset, and automatically determine if a general neural-network model will be adopted or a marker-guided hierarchical"
