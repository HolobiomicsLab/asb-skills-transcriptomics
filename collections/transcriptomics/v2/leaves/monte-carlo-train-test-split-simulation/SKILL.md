---
name: monte-carlo-train-test-split-simulation
description: Reference for designing Monte Carlo simulation benchmarks of supervised
  cell annotation methods on labeled scRNA-seq data, e.g. when no fixed train/test
  split exists and repeated random splits with varying training proportions are needed.
when_to_use_negative:
- When all cell types in the new sample are not represented in the reference/training
  data — NeuCA and this exhaustive-annotation benchmark assume no unknown cell types
- When a fixed, pre-specified train/test split already exists and repeated random
  resampling would not answer the question
- When the goal is de novo clustering without any labeled training data (unsupervised-only
  setting)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: NeuCA
  role: Supervised neural-network classifier under benchmark at model sizes small/medium/big
    with tau = 0.95 correlation-based strategy selection
  repo: https://bioconductor.org/packages/NeuCA
- name: scmap
  role: Baseline supervised/correlation-based annotation method (scmap-cluster, scmap-cell)
    run on identical splits
- name: CHETAH
  role: Baseline supervised annotation method run on identical splits
- name: Seurat
  role: Baseline unsupervised clustering method run on identical splits
- name: SC3
  role: Baseline unsupervised clustering method run on identical splits
- name: R
  role: Implementation environment (R/Bioconductor) for all simulation and benchmark
    code
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:d76fec99bc86cc1358b679ce082cb9fd9c31cf0539dfbfa6498c3f3d2c27f154
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# monte-carlo-train-test-split-simulation

## Summary

Benchmark protocol from the NeuCA paper that generates Monte Carlo train/test splits of the FACS-sorted 10X Genomics PBMC dataset (Zheng_8ct_PBMC) to evaluate cell annotation methods by accurately assigned rate and ARI.

## When to use

Reference for designing Monte Carlo simulation benchmarks of supervised cell annotation methods on labeled scRNA-seq data, e.g. when no fixed train/test split exists and repeated random splits with varying training proportions are needed.

## When NOT to use

- When all cell types in the new sample are not represented in the reference/training data — NeuCA and this exhaustive-annotation benchmark assume no unknown cell types
- When a fixed, pre-specified train/test split already exists and repeated random resampling would not answer the question
- When the goal is de novo clustering without any labeled training data (unsupervised-only setting)

## Inputs

- FACS-sorted 10X Genomics PBMC scRNA-seq count matrix with published cell type labels (Zheng_8ct_PBMC, from the R/Bioconductor package DuoClustering2018)
- Normalized expression matrices with training/test cells subset to overlapped genes
- SingleCellExperiment objects for training and testing data (NeuCA input format)

## Outputs

- benchmark_results.csv with accurately assigned rate and ARI per method per split
- Aggregated accuracy/ARI comparisons across 160 Monte Carlo simulations
- Predicted cell label vectors per testing cell for confusion-matrix evaluation

## How to apply

Download the FACS-sorted 10X Genomics PBMC dataset (9139 CD4 T helper, 7959 B, 718 CD14 monocytes, 7601 NK, 9338 memory T, 10993 naive cytotoxic T, 7700 naive T, 7861 regulatory T cells) with published labels, from DuoClustering2018 (Zheng_8ct_PBMC), normalize the expression matrices and subset training/test matrices to overlapped genes. For each Monte Carlo simulation, randomly draw p = 10%, 20%, 50%, or 80% of cells as training data and draw n = 800, 1600, or 4000 testing cells from the remainder, yielding 160 simulations in the paper. Optionally add a T-cell-only setting using the five correlated T cell types and a merged setting collapsing all T cells into one type to test hierarchical-structure effects. Run each method (NeuCA at small/medium/big model sizes with tau = 0.95 strategy threshold, plus scmap-cluster, scmap-cell, CHETAH, Seurat, SC3) on identical splits, then compute accurately assigned rate and Adjusted Rand Index per split and aggregate; validate that NeuCA-med exceeds scmap-cluster by about 10 percentage points and NeuCA-small by at least 5 points in accurately assigned rate.

## Related tools

- **NeuCA** (Supervised neural-network classifier under benchmark at model sizes small/medium/big with tau = 0.95 correlation-based strategy selection) — https://bioconductor.org/packages/NeuCA
- **scmap** (Baseline supervised/correlation-based annotation method (scmap-cluster, scmap-cell) run on identical splits)
- **CHETAH** (Baseline supervised annotation method run on identical splits)
- **Seurat** (Baseline unsupervised clustering method run on identical splits)
- **SC3** (Baseline unsupervised clustering method run on identical splits)
- **R** (Implementation environment (R/Bioconductor) for all simulation and benchmark code)

## Examples

```
predicted.label = NeuCA(train = train_sce, test = test_sce, model.size = "medium", verbose = FALSE)
```

## Evaluation signals

- Number of simulations adds up: 4 training proportions (10/20/50/80%) × 3 test sizes (800/1600/4000) plus T-cell-only and merged settings ≈ 160 Monte Carlo simulations
- Every method is evaluated on identical train/test splits — metrics must be comparable cell-for-cell
- Accurately assigned rate and ARI are computed per split and aggregated; check benchmark_results.csv contains both metrics per method per split
- Expected ordering: all three NeuCA versions rank at the top in both accurately assigned rate and ARI
- Margin check: NeuCA-med exceeds scmap-cluster (best existing method) by ~10 percentage points; NeuCA-small by at least 5 points in accurately assigned rate

## Limitations

- NeuCA is restricted to settings where all cell types in the new sample already exist in the reference/training data — unknown cell types are out of scope
- Existing label assignment methods (e.g. CHETAH) may still label a considerable number of cells as unassigned even when unknown cell types are not expected
- No version numbers are given for keras, TensorFlow, or R in the software description, complicating exact reproducibility

## Evidence

- [methods] Real PBMC data from the 10X Genomics website: "Real PBMC data from the 10X Genomics website (https://support.10xgenomics.com/)31 were downloaded and processed to generate our simulation datasets"
- [methods] Zheng_8ct_PBMC source: "The Zheng_8ct_PBMC data were downloaded from the R/Bioconductor package DuoClustering2018"
- [methods] Preprocessing steps: [evidence span withheld — anchor status: not_found]
- [methods] Correlation threshold for strategy selection: [evidence span withheld — anchor status: not_found]
- [readme] Model size option in README: "predicted.label = NeuCA(train = Baron_sce, test = Seg_sce, 
                        model.size = "big", verbose = FALSE)"
- [discussion] NeuCA scope limitation: "NeuCA is designed to fit the situations where cell types are well-studied, i.e., the situations where all the cell types in the new sample already exist in the reference/training data."
