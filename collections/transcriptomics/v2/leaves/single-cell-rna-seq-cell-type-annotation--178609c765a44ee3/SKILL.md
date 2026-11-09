---
name: single-cell-rna-seq-cell-type-annotation
description: Use when annotating newly sequenced scRNA-seq cells or clusters against
  labeled reference data (e.g., human pancreas, PBMC, Tabula Muris), or when benchmarking
  cross-platform classification against Seurat v3 and scmap.
when_to_use_negative:
- The query cells belong to cell types absent from the labeled reference and no null-dataset
  confidence-score control (conf_score with cutoffs 0.001–0.5) has been set up — unrepresented
  types cannot be identified by the plain Bet classifier
- Input is not single-cell transcriptomic data with a labeled reference (e.g., bulk
  RNA-seq, proteomics, or already-computed feature/annotation tables)
- A fully unsupervised clustering/annotation workflow is desired, since SciBet is
  a supervised classifier requiring pre-annotated training labels
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: SciBet
  role: 'Primary supervised classifier: SelectGene ranks genes by ΔS, Learn fits the
    multinomial model with Laplace smoothing, Bet assigns cell types'
  repo: github.com/PaulingLiu/scibet
- name: Seurat v3
  role: Benchmark annotation method via FindTransferAnchors and TransferData functions
- name: scmap
  role: Benchmark annotation method via scmapCluster, ignoring unassigned cells
- name: scikit-learn (sklearn)
  role: Hold-out validation strategy via model_selection.train_test_split (stratified
    7:3 split)
- name: imbalanced-learn
  role: Down-sampling strategy via under_sampling.RandomUnderSampler (e.g., 2,500
    cells per PBMC cell type)
- name: Python
  role: Implemented the sklearn hold-out validation and downsampling procedure
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:1b4761c487083d0a060c9c437a266e39c3b6fe89f9656454cf4d41ed13389f2f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Rcpp
  - SciBet
  - Seurat v3
  - imbalanced-learn
  - scikit-learn (sklearn)
  - scmap
  - tidyverse
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-rna-seq-cell-type-annotation

## Summary

A supervised single-cell RNA-seq cell type annotation skill based on SciBet, a multinomial maximum-likelihood classifier that ranks informative genes by the ΔS statistic and assigns cell identities with order-of-magnitude speed advantage. It supports cross-validation against Seurat v3 and scmap benchmarks on pancreas and PBMC datasets.

## When to use

Use when annotating newly sequenced scRNA-seq cells or clusters against labeled reference data (e.g., human pancreas, PBMC, Tabula Muris), or when benchmarking cross-platform classification against Seurat v3 and scmap.

## When NOT to use

- The query cells belong to cell types absent from the labeled reference and no null-dataset confidence-score control (conf_score with cutoffs 0.001–0.5) has been set up — unrepresented types cannot be identified by the plain Bet classifier
- Input is not single-cell transcriptomic data with a labeled reference (e.g., bulk RNA-seq, proteomics, or already-computed feature/annotation tables)
- A fully unsupervised clustering/annotation workflow is desired, since SciBet is a supervised classifier requiring pre-annotated training labels

## Inputs

- Normalized scRNA-seq expression matrix (gene x cell) with annotated cell-type labels for the reference/training set
- Held-out test-cell expression matrix from the same or a distinct scRNA-seq platform (full-length TPM or UMI counts)
- Seurat v3 or scmap formatted expression objects for comparative classification

## Outputs

- Per-cell predicted cell-type labels with maximum-likelihood assignments
- Normalized confusion matrix per dataset (e.g., normalized_confusion_matrices.csv for PBMC and pancreas)
- Mean classification accuracy table per training-test pair per method (e.g., fig3a_benchmark_accuracy.csv, 6 pairs x SciBet/Seurat v3/scmap)

## How to apply

Normalize the expression data according to its technology: TPM plus pseudo-value one with log-normalization for full-length reads, or size-factor 10,000, add one, and log normalization for UMI data. Apply a stratified 7:3 train/test split without replacement (sklearn model_selection.train_test_split, optionally with imbalanced-learn RandomUnderSampler for down-sampling), repeating the entire procedure 50 times. On each training set, run SciBet's SelectGene to rank genes by the ΔS statistic and keep the top 500 informative genes, then Learn the multinomial model with Laplace smoothing. Classify held-out cells with Bet (maximum likelihood over informative genes), and for benchmarking compare against Seurat v3's FindTransferAnchors/TransferData and scmap's scmapCluster (ignoring unassigned cells). Average per-repeat accuracy over 50 repeats to produce accuracy or confusion-matrix tables (e.g., fig3a_benchmark_accuracy.csv, normalized_confusion_matrices.csv).

## Related tools

- **SciBet** (Primary supervised classifier: SelectGene ranks genes by ΔS, Learn fits the multinomial model with Laplace smoothing, Bet assigns cell types) — github.com/PaulingLiu/scibet
- **Seurat v3** (Benchmark annotation method via FindTransferAnchors and TransferData functions)
- **scmap** (Benchmark annotation method via scmapCluster, ignoring unassigned cells)
- **scikit-learn (sklearn)** (Hold-out validation strategy via model_selection.train_test_split (stratified 7:3 split))
- **imbalanced-learn** (Down-sampling strategy via under_sampling.RandomUnderSampler (e.g., 2,500 cells per PBMC cell type))
- **Python** (Implemented the sklearn hold-out validation and downsampling procedure)

## Examples

```
devtools::install_github("PaulingLiu/scibet"); genes <- SelectGene(expr, 500); model <- Learn(expr, genes, 500, 1); pred <- Bet(test_expr, model)
```

## Evaluation signals

- Output accuracy table contains all expected dataset pairs and 3 method columns with mean accuracies in [0,1]
- SciBet's mean accuracy ranks at or above Seurat v3's, which ranks above scmap's on the 6 cross-platform pancreas pairs (Fig. 3a ordering)
- PBMC confusion matrix reflects a balanced 2,500-cell-per-type design and discriminates close subtypes CD4 memory T cells vs CD4 regulatory T cells
- Pancreas confusion matrix correctly identifies rare delta cells (3.8% of pancreatic cells), matching Fig. 2f/2g patterns
- Every split used the top 500 ΔS-selected informative genes and the full procedure was repeated 50 times with 7:3 stratified splits

## Limitations

- No explicit version numbers reported for scikit-learn, Seurat v3 minor version, kallisto, or imbalanced-learn
- No seed or randomization parameters reported for the 50x repeated splits and permutation tests
- Significance approximation for the ΔS permutation test and the enumeration of the 16 datasets for 10 training-test pairs are relegated to Supplementary Tables/Notes not included here
- Cell types not covered by the reference require a null dataset as alternate background to control false positives; without it, out-of-reference cells cannot be flagged

## Evidence

- [abstract] We present SciBet, a supervised cell type identifier that accurately predicts cell identity for newly sequenced cells with order-of-magnitude speed advantage.: "We present SciBet, a supervised cell type identifier that accurately predicts cell identity for newly sequenced cells with order-of-magnitude speed advantage."
- [discussion] we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [discussion] we selected genes with top ΔS as informative genes (500 by default): "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions: "For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions"
- [readme] Fast, robust and technology-independent computational methods are needed for supervised cell type annotation of single-cell RNA sequencing data.: "Fast, robust and technology-independent computational methods are needed for supervised cell type annotation of single-cell RNA sequencing data."
