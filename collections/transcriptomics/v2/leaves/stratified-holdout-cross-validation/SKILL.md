---
name: stratified-holdout-cross-validation
description: Use when benchmarking a supervised cell-type classifier (e.g., SciBet
  vs scmap vs Seurat v3) on labeled scRNA-seq datasets and a repeated, class-balanced
  hold-out estimate of classification accuracy is needed.
when_to_use_negative:
- The dataset has no trusted cell-type labels, so a supervised hold-out accuracy comparison
  is not meaningful
- You need a one-time train/test evaluation rather than a benchmark averaged over
  50 repeated splits (e.g., a single deployment prediction)
- Evaluating prediction of cell types absent from the reference (negative/unrepresented
  cells), which uses the null-dataset confidence score C instead of plain held-out
  accuracy
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn (sklearn)
  role: Provides model_selection.train_test_split for the stratified 7:3 train/test
    splits without replacement
- name: imbalanced-learn
  role: under_sampling.RandomUnderSampler down-sampling of imbalanced datasets before
    splitting (e.g., 2,500 cells per PBMC cell type)
- name: SciBet
  role: 'Classifier retrained per split: SelectGene (top 500 ΔS genes), Learn (Laplace
    smoothing), Bet for held-out prediction'
  repo: github.com/PaulingLiu/scibet
- name: scmap
  role: Comparison classifier invoked as scmapCluster, ignoring unassigned cells,
    on the same splits
- name: Seurat v3
  role: Comparison classifier using FindTransferAnchors plus TransferData on the same
    splits; also supplies UMI normalization
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:685e69268fc98187e9c363ca302efb7a6517b8ced1026173791212b56056b49e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Rcpp
  - SciBet
  - Seurat v3
  - imbalanced-learn
  - scikit-learn
  - scikit-learn (sklearn)
  - scmap
  - tidyverse
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# stratified-holdout-cross-validation

## Summary

Benchmarking strategy from the SciBet paper: repeatedly split a labeled scRNA-seq dataset into stratified train/test sets at a 7:3 ratio (sklearn's train_test_split, sampling without replacement), retrain and classify on each split, and average accuracy over 50 repetitions for a robust per-dataset/per-method comparison.

## When to use

Use when benchmarking a supervised cell-type classifier (e.g., SciBet vs scmap vs Seurat v3) on labeled scRNA-seq datasets and a repeated, class-balanced hold-out estimate of classification accuracy is needed.

## When NOT to use

- The dataset has no trusted cell-type labels, so a supervised hold-out accuracy comparison is not meaningful
- You need a one-time train/test evaluation rather than a benchmark averaged over 50 repeated splits (e.g., a single deployment prediction)
- Evaluating prediction of cell types absent from the reference (negative/unrepresented cells), which uses the null-dataset confidence score C instead of plain held-out accuracy

## Inputs

- Labeled scRNA-seq expression matrix (log-normalized counts, e.g., TPM+1 log for full-length, Seurat v3 size-factor-10,000 log for UMI)
- Per-cell type labels (e.g., 7 PBMC cell types, pancreatic cell types including rare delta cells ~3.8%)
- Benchmark method implementations (SciBet, scmapCluster, Seurat v3 FindTransferAnchors/TransferData)

## Outputs

- Per-repeat classification accuracy on the held-out test set for each dataset-method pair
- Mean accuracy over the 50 repetitions (e.g., fig2a_feature_selection_accuracy.csv, fig3a_benchmark_accuracy.csv)
- Accumulated normalized confusion matrices across 50 repeats (normalized_confusion_matrices.csv)

## How to apply

For each labeled single-cell dataset, normalize first (TPM plus pseudo-value one and log-normalization for full-length reads; Seurat v3-style size-factor 10,000, add one, log normalization for UMI data), then form a stratified random training/test split without replacement at a 7:3 ratio using sklearn's model_selection.train_test_split. For imbalanced datasets, combine the split with imbalanced-learn's under_sampling.RandomUnderSampler down-sampling (e.g., 2,500 cells per cell type for the PBMC benchmark). On each training set, re-run the full pipeline from scratch — SelectGene (top 500 genes by ΔS), Learn (Laplace smoothing), then Bet — and classify the held-out test cells, ignoring unassigned calls. Repeat the entire procedure including random sampling and classification 50 times, aggregate per-repeat accuracies into a per-dataset mean, and compare methods on the same splits.

## Related tools

- **scikit-learn (sklearn)** (Provides model_selection.train_test_split for the stratified 7:3 train/test splits without replacement)
- **imbalanced-learn** (under_sampling.RandomUnderSampler down-sampling of imbalanced datasets before splitting (e.g., 2,500 cells per PBMC cell type))
- **SciBet** (Classifier retrained per split: SelectGene (top 500 ΔS genes), Learn (Laplace smoothing), Bet for held-out prediction) — github.com/PaulingLiu/scibet
- **scmap** (Comparison classifier invoked as scmapCluster, ignoring unassigned cells, on the same splits)
- **Seurat v3** (Comparison classifier using FindTransferAnchors plus TransferData on the same splits; also supplies UMI normalization)

## Examples

```
from sklearn.model_selection import train_test_split; X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.3, stratify=y)  # repeat 50x
```

## Evaluation signals

- Each of the 50 splits is stratified: the 7:3 train/test ratio holds within every cell type (sampling without replacement)
- Mean accuracy for each method lies in [0,1] and the comparison table contains all expected dataset pairs × method columns
- Per-dataset mean accuracies reproduce the published ordering (e.g., E-test/scmap pairing in Fig. 2a; SciBet ≥ Seurat v3 > scmap in Fig. 3a)
- Normalized confusion matrices match Fig. 2f/2g patterns: CD4 memory vs regulatory T cells discriminated in PBMC; rare delta cells (3.8%) correctly identified
- Accuracy distribution across the 50 repeats is stable enough that the per-dataset mean reflects a consistent method ranking rather than split noise

## Limitations

- No seed or randomization parameters were reported for the 50× repeated splits, so exact reproduction of individual repeats is not possible
- Per-split results are sensitive to class imbalance unless RandomUnderSampler down-sampling is applied; scmap and Seurat v3 cannot discriminate close subtypes (CD4 memory vs regulatory T cells) even under this balanced protocol
- scmapCluster unassigned cells must be explicitly excluded (ignored) from accuracy computation, which inflates apparent accuracy relative to methods that always assign a label

## Evidence

- [discussion] Stratified hold-out design: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [discussion] Implementation via sklearn: "We implemented the hold-out validation strategy proposed by the python package sklearn9 (function model_selection.train_test_split)"
- [discussion] Down-sampling of imbalanced data: "down sampling strategy proposed by python package imbalanced-learn22 (under_sampling.RandomUnderSampler)"
- [discussion] Repetition and aggregation protocol: "we repeated the entire procedure including random sampling and classification for 50 times."
- [readme] Reproduction scripts: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
