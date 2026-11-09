---
name: scrna-seq-cell-type-annotation
description: Use when training scRNA-seq data with known cell labels exist and every
  test cell must receive exactly one of the K reference labels, especially with closely
  correlated cell types.
when_to_use_negative:
- 'When unknown/novel cell types may exist in the test sample: NeuCA is restricted
  to settings where all cell types in the new sample already exist in the reference/training
  data; screen for rare/unknown cells first (e.g., inferCNV or copyKAT for malignant
  cells) before applying it'
- When no labeled training data exist for the tissue — supervised methods like NeuCA
  require a reference; unsupervised clustering (Seurat, SC3, TSCAN) is the alternative
- Cross-condition prediction beyond validated scenarios (e.g., the ASD data application)
  may not hold; consult domain experts before trusting cross-condition labels
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: Supervised neural-network classifier that performs the exhaustive cell label
    assignment, auto-selecting feed-forward vs hierarchical strategy via the Pearson
    correlation check (tau = 0.95)
  repo: https://bioconductor.org/packages/NeuCA/
- name: keras
  role: R API used to build and train the ReLU feed-forward and hierarchical binary
    networks with RMSprop and cross-entropy losses
- name: limma
  role: Linear model framework and testing used to select the top M = 1000 bifurcation-specific
    marker features at each tree node
- name: SingleCellExperiment
  role: Bioconductor data container required for both train and test inputs to NeuCA
- name: e1071
  role: Provides probability and predictive functions used by NeuCA
- name: scmap
  role: Benchmark comparator (best existing supervised method, scmap-cluster) used
    to quantify NeuCA's accuracy gain on the same Monte Carlo splits
- name: CHETAH
  role: Benchmark comparator; known to produce large numbers of unassigned/intermediate
    labels under the exhaustive-assignment expectation
merged_aliases:
- scrna-seq-metadata-cell-type-annotation-handling
merged_alias_records:
- alias: scrnaseq-cell-type-annotation-benchmarking
  slug: scrnaseq-cell-type-annotation-benchmarking
  cosine_score: 0.8418
  method: embedding-cosine
  decision: needs_review
- alias: single-cell-rna-seq-cell-type-annotation
  slug: single-cell-rna-seq-cell-type-annotation
  cosine_score: 0.8489
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:5c5a75825e227eb069402d9dd7e044279f060d893b757febf92954415e18bd4d
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
  - SingleCellExperiment
  - TSCAN
  - TensorFlow
  - devtools
  - e1071
  - keras
  - limma
  - rTensor
  - scRank
  - scmap
  - tensorly
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# scrna-seq-cell-type-annotation

## Summary

Exhaustively annotate scRNA-seq cells against a labeled reference using NeuCA, a supervised R/Bioconductor neural-network classifier that automatically switches between a feed-forward network and a marker-guided hierarchical (cell lineage tree) network when cell types are highly correlated.

## When to use

Use when training scRNA-seq data with known cell labels exist and every test cell must receive exactly one of the K reference labels, especially with closely correlated cell types.

## When NOT to use

- When unknown/novel cell types may exist in the test sample: NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data; screen for rare/unknown cells first (e.g., inferCNV or copyKAT for malignant cells) before applying it
- When no labeled training data exist for the tissue — supervised methods like NeuCA require a reference; unsupervised clustering (Seurat, SC3, TSCAN) is the alternative
- Cross-condition prediction beyond validated scenarios (e.g., the ASD data application) may not hold; consult domain experts before trusting cross-condition labels

## Inputs

- Training scRNA-seq count matrix with known cell labels as a SingleCellExperiment object (normcounts assay, colData labels, feature_symbol rowData, duplicated features removed)
- Testing scRNA-seq count matrix as a SingleCellExperiment object (labels optional)
- model.size argument: 'small', 'medium', or 'big'

## Outputs

- predicted.label: vector of one cell-type label per test cell, drawn from the K training-defined types
- Optional confusion matrix / Sankey diagram against true test labels
- benchmark_results.csv of accuracy and Adjusted Rand Index per method per split (benchmark workflows)

## How to apply

Normalize the expression matrices and subset training (X0) and testing (X1) data to their overlapped genes, wrapping both as SingleCellExperiment objects with normcounts assays and feature_symbol rowData. Compute the averaged G x K expression profile per cell type and the Pearson correlation matrix P; if any off-diagonal correlation reaches tau = 0.95, NeuCA builds a cell lineage tree by complete-linkage hierarchical clustering and trains binary feed-forward networks (RMSprop, ReLU) at bifurcations using the top M = 1000 limma-selected bifurcation-specific features, after pre-assigning cells expressing > nu2 = 3 of > nu1 = 10 highly sensitive markers (Sen_gk > 0.95); otherwise it trains a single feed-forward network (small: 3 layers/64 nodes; medium: 4 layers/64,128; big: 5 layers/64,128,256) with categorical cross-entropy and softmax output. Run via one call `NeuCA(train, test, model.size = ...)`, which returns one predicted label per test cell.

## Related tools

- **NeuCA** (Supervised neural-network classifier that performs the exhaustive cell label assignment, auto-selecting feed-forward vs hierarchical strategy via the Pearson correlation check (tau = 0.95)) — https://bioconductor.org/packages/NeuCA/
- **keras** (R API used to build and train the ReLU feed-forward and hierarchical binary networks with RMSprop and cross-entropy losses)
- **limma** (Linear model framework and testing used to select the top M = 1000 bifurcation-specific marker features at each tree node)
- **SingleCellExperiment** (Bioconductor data container required for both train and test inputs to NeuCA)
- **e1071** (Provides probability and predictive functions used by NeuCA)
- **scmap** (Benchmark comparator (best existing supervised method, scmap-cluster) used to quantify NeuCA's accuracy gain on the same Monte Carlo splits)
- **CHETAH** (Benchmark comparator; known to produce large numbers of unassigned/intermediate labels under the exhaustive-assignment expectation)

## Examples

```
predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- Every test cell receives exactly one of the K training-defined labels — no unassigned or intermediate labels in predicted.label
- table(predicted.label) sums to the number of test cells and contains only reference cell-type names
- Confusion matrix against true test labels is near-diagonal (README example: alpha 328/331, gamma 132/135 correct)
- On the difficult five-T-cell-type PBMC setting, accuracy/ARI should exceed competing methods (scmap, CHETAH, CellAssign, SCINA), which assign >50% 'unassigned' to naive T, CD4 T helper, and regulatory T cells
- In the 10X PBMC Monte Carlo benchmark, verify NeuCA-med exceeds scmap-cluster by ~10 percentage points and NeuCA-small by at least 5 in accurately assigned rate

## Limitations

- NeuCA is designed for well-studied situations where all cell types in the new sample already exist in the reference/training data; it cannot flag novel types
- Whether unknown/novel cell types exist is not addressed by the method — consult biological/clinical experts for tissues from novel conditions
- Cross-condition annotation (demonstrated on ASD data) may not hold in other scenarios
- No version numbers are given for keras, TensorFlow, or R in the software description

## Evidence

- [abstract] NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types: "NeuCA can utilize the hierarchical structure information of the cell types to improve the annotation accuracy, which is especially helpful when data contain closely correlated cell types"
- [methods] Correlation-based strategy selection (threshold tau = 0.95): feed-forward network vs hierarchical network: [evidence span withheld — anchor status: not_found]
- [methods] Preprocessing: normalize expression matrix and subset training and testing data by overlapped genes: [evidence span withheld — anchor status: not_found]
- [readme] NeuCA can detect whether highly-correlated cell types exist in the training dataset, and automatically determine if a general neural-network model will be adopted or a marker-guided hierarchical neural-network will be adopted for classification.: "NeuCA can detect whether highly-correlated cell types exist in the training dataset, and automatically determine if a general neural-network model will be adopted or a marker-guided hierarchical"
- [discussion] NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data.: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
