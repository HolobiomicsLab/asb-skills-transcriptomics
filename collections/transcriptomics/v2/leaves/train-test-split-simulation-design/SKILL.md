---
name: train-test-split-simulation-design
description: Use when benchmarking cell-annotation or classification methods under
  controlled sample-size conditions, especially to stress-test them on closely correlated
  classes versus easy low-correlation classes.
when_to_use_negative:
- The data contain novel/unknown cell types absent from the reference training data
  — NeuCA assumes all cell types in the new sample exist in the reference, and the
  merged-class easy task presumes exhaustive labeling.
- The scientific question is clustering rather than supervised label assignment using
  known training labels — unsupervised tools (Seurat, SC3, TSCAN) are the alternative
  when no training data exist.
- The aim is a single one-off prediction rather than a controlled size/correlation
  sweep — a simple train/test split without the p/n grid suffices.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_0091
tools:
- name: NeuCA
  role: Primary supervised annotator under evaluation; auto-selects feed-forward vs
    marker-guided hierarchical neural network via Pearson correlation threshold tau
    = 0.95
  repo: https://bioconductor.org/packages/NeuCA/
- name: scmap
  role: Competing correlation-based annotation method benchmarked on every split
- name: CHETAH
  role: Competing hierarchical annotation method; produces unassigned/intermediate
    labels tracked in evaluation
- name: CellAssign
  role: Competing marker-based annotation method benchmarked on every split
- name: SCINA
  role: Competing marker-based annotation method benchmarked on every split
- name: Seurat
  role: Unsupervised clustering reference tool in the article's tool set
- name: SC3
  role: Unsupervised clustering reference tool in the article's tool set
- name: TSCAN
  role: Unsupervised clustering reference tool in the article's tool set
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:ca16fb49967d3999f1af12839e9b93d4708f59d386e81d8560eda6224ccdafd5
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# train-test-split-simulation-design

## Summary

Designs controlled train/test simulation benchmarks from real PBMC scRNA-seq data by varying training proportions (p = 10%, 20%, 50%, 80%) and test sizes (n = 800, 1600, 4000) under a difficult (five highly correlated T-cell types) and an easy (merged major cell types) prediction task, then scores accuracy, ARI, and unassigned-label fractions per method.

## When to use

Use when benchmarking cell-annotation or classification methods under controlled sample-size conditions, especially to stress-test them on closely correlated classes versus easy low-correlation classes.

## When NOT to use

- The data contain novel/unknown cell types absent from the reference training data — NeuCA assumes all cell types in the new sample exist in the reference, and the merged-class easy task presumes exhaustive labeling.
- The scientific question is clustering rather than supervised label assignment using known training labels — unsupervised tools (Seurat, SC3, TSCAN) are the alternative when no training data exist.
- The aim is a single one-off prediction rather than a controlled size/correlation sweep — a simple train/test split without the p/n grid suffices.

## Inputs

- Real PBMC scRNA-seq data from the 10X Genomics website (normalized expression matrices with known cell-type labels)
- Cell-type label vector for training cells
- Sweep grid: training proportions p = 10%, 20%, 50%, 80%; test sizes n = 800, 1600, 4000

## Outputs

- benchmark_results.csv with per-method annotation accuracy and ARI for every difficult- and easy-setting split
- Per-method fraction of 'unassigned' labels per cell type per split
- Predicted cell-label vectors for each method on each split

## How to apply

Download and preprocess real PBMC data from 10X Genomics, normalizing the expression matrix and subsetting training and testing matrices to their overlapped genes. Build the difficult task by restricting data to the five highly correlated T-cell types (CD4 T helper, memory T, naive cytotoxic T, naive T, regulatory T), and the easy task by merging all five T-cell types into a single 'T cells' class among the major cell types. For each setting, generate splits across training proportions p = 10%, 20%, 50%, 80% and test sizes n = 800, 1600, 4000, then run all competing annotators (NeuCA, scmap, CHETAH, CellAssign, SCINA) on every split, allowing NeuCA's internal Pearson-correlation check (threshold tau = 0.95) to choose between its feed-forward and hierarchical neural networks. Compute per-method accuracy and Adjusted Rand Index (ARI) per split, plus the fraction of 'unassigned' labels per method per cell type, and write all metrics to benchmark_results.csv.

## Related tools

- **NeuCA** (Primary supervised annotator under evaluation; auto-selects feed-forward vs marker-guided hierarchical neural network via Pearson correlation threshold tau = 0.95) — https://bioconductor.org/packages/NeuCA/
- **scmap** (Competing correlation-based annotation method benchmarked on every split)
- **CHETAH** (Competing hierarchical annotation method; produces unassigned/intermediate labels tracked in evaluation)
- **CellAssign** (Competing marker-based annotation method benchmarked on every split)
- **SCINA** (Competing marker-based annotation method benchmarked on every split)
- **Seurat** (Unsupervised clustering reference tool in the article's tool set)
- **SC3** (Unsupervised clustering reference tool in the article's tool set)
- **TSCAN** (Unsupervised clustering reference tool in the article's tool set)

## Examples

```
predicted.label = NeuCA(train = Tcell5_train_sce, test = Tcell5_test_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- Under the difficult (five T-cell type) setting, NeuCA must outperform all existing methods on both accuracy and ARI for every (p, n) split.
- Existing methods should assign >50% 'unassigned' labels to naive T, CD4 T helper, and regulatory T cells in the difficult setting.
- CD56 NK and CD14 monocytes should be annotated correctly by all methods, serving as a sanity floor for the benchmark.
- Each split's predicted-label vector must contain only labels drawn from the training set's cell-type vocabulary (exhaustive assignment).
- benchmark_results.csv must have one accuracy/ARI row per method per (setting, p, n) combination with no missing entries.

## Limitations

- NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data; the merged-class easy task cannot be extended to novel cell types.
- Existing methods (e.g., CHETAH) label a considerable number of cells as unassigned even when unknown cell types are not expected, inflating the unassigned-fraction metric.
- Deciding whether unknown/novel cell types exist in the data is not addressed by the method; expert consultation is recommended.
- No version numbers are given for keras, TensorFlow, or R in the software description, complicating reproducibility of the sweep.

## Evidence

- [results] difficult setting of highly correlated T-cell types, NeuCA outperforms all existing methods in both accuracy and ARI: [evidence span withheld — anchor status: not_found]
- [methods] training proportions and test sizes grid: [evidence span withheld — anchor status: not_found]
- [methods] preprocessing: [evidence span withheld — anchor status: not_found]
- [methods] correlation threshold: [evidence span withheld — anchor status: not_found]
- [readme] NeuCA auto strategy selection: "NeuCA can detect whether highly-correlated cell types exist in the training dataset, and automatically determine if a general neural-network model will be adopted or a marker-guided hierarchical"
- [abstract] unassigned labels even when not expected: "when unknown cell types are not expected, existing methods still label a considerable number of cells as unassigned, which is not desirable"
- [discussion] restriction to known reference types: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
