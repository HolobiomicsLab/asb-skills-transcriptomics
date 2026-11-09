---
name: cross-method-annotation-comparison
description: Use when comparing cell-type annotation/classification methods on labeled
  scRNA-seq data and you need a fair, repeated train/test benchmark (accuracy and
  balanced accuracy) across multiple datasets.
when_to_use_negative:
- The goal is annotating a new unlabeled dataset with a single trained model (e.g.
  via Bet/ExportModel) rather than comparing classifiers head-to-head
- Input data lacks trustworthy cell-type labels, since the stratified 7:3 hold-out
  benchmark requires reference labels for training and evaluation
- Datasets have mismatched gene spaces with no orthology mapping, when the task is
  cross-species transfer rather than same-dataset accuracy comparison
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SciBet
  role: Primary classifier under test; trained with Laplace-smoothed multinomial Learn
    on top-500 E-test genes and predicts via maximum-likelihood Bet
  repo: github.com/PaulingLiu/scibet
- name: scmap
  role: Baseline classifier; scmapCluster run on identical splits with unassigned
    cells ignored when scoring
- name: Seurat v3
  role: Baseline classifier; FindTransferAnchors followed by TransferData on identical
    splits
- name: scikit-learn (sklearn)
  role: Provides model_selection.train_test_split for the stratified 7:3 train/test
    splits repeated 50 times
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:63389fb626100bc540db8ef159852142539a406d13079547472bbcc43919ea2d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciBet
  - Seurat v3
  - scikit-learn (sklearn)
  - scmap
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cross-method-annotation-comparison

## Summary

Benchmarks supervised single-cell cell-type classifiers (SciBet vs scmapCluster vs Seurat v3 anchor-based transfer) on the same labeled scRNA-seq datasets using repeated stratified hold-out validation, comparing accuracy and balanced accuracy per dataset.

## When to use

Use when comparing cell-type annotation/classification methods on labeled scRNA-seq data and you need a fair, repeated train/test benchmark (accuracy and balanced accuracy) across multiple datasets.

## When NOT to use

- The goal is annotating a new unlabeled dataset with a single trained model (e.g. via Bet/ExportModel) rather than comparing classifiers head-to-head
- Input data lacks trustworthy cell-type labels, since the stratified 7:3 hold-out benchmark requires reference labels for training and evaluation
- Datasets have mismatched gene spaces with no orthology mapping, when the task is cross-species transfer rather than same-dataset accuracy comparison

## Inputs

- 14 labeled benchmarking scRNA-seq datasets (Supplementary Table 1) with raw/full-length counts or UMI counts
- Cell-level annotation labels for each dataset
- Installed R packages: SciBet (PaulingLiu/scibet), scmap, Seurat v3; Python scikit-learn for splitting

## Outputs

- 14-dataset × 3-method table of accuracy and balanced accuracy aggregated over 50 repeats (benchmark_accuracy_results.csv)
- Per-dataset winner summary showing which classifier achieves highest accuracy on the majority of datasets

## How to apply

Normalize each dataset per platform convention (TPM plus pseudo-count 1 with log-normalization for full-length data; Seurat v3-style size-factor 10,000, add one, log-normalization for UMI data), then use sklearn's model_selection.train_test_split for stratified 7:3 training/test splits without replacement, repeating the entire procedure 50 times. On each split, rank genes by the E-test ΔS statistic and retain the top 500 informative genes as the feature set for all methods. Train SciBet (Learn with Laplace-smoothed multinomial parameters) and classify test cells by maximum likelihood (Bet); run scmapCluster on the same splits ignoring unassigned cells, and Seurat v3 via FindTransferAnchors followed by TransferData. Compute accuracy and balanced accuracy per method per dataset, aggregate over the 50 repeats, and tabulate the 14-dataset × 3-method results (e.g. benchmark_accuracy_results.csv).

## Related tools

- **SciBet** (Primary classifier under test; trained with Laplace-smoothed multinomial Learn on top-500 E-test genes and predicts via maximum-likelihood Bet) — github.com/PaulingLiu/scibet
- **scmap** (Baseline classifier; scmapCluster run on identical splits with unassigned cells ignored when scoring)
- **Seurat v3** (Baseline classifier; FindTransferAnchors followed by TransferData on identical splits)
- **scikit-learn (sklearn)** (Provides model_selection.train_test_split for the stratified 7:3 train/test splits repeated 50 times)

## Examples

```
devtools::install_github("PaulingLiu/scibet")
```

## Evaluation signals

- Result table contains all 14 datasets × 3 methods with accuracy and balanced-accuracy values in [0, 1]
- SciBet attains the highest mean accuracy and balanced accuracy on the majority of the 14 datasets, consistent with the ordering in Fig. 2c and 2d
- Each split retains exactly the top 500 informative genes ranked by the ΔS statistic
- All methods are evaluated on the identical 50 stratified 7:3 splits (same train_test_split calls) so the comparison is paired
- scmap results are computed ignoring unassigned cells, matching the reported protocol

## Limitations

- No seed or randomization parameters were reported for the 50× repeated splits, so exact reproduction of point estimates may vary
- Software versions for scikit-learn, Seurat v3 minor version, and other dependencies are not specified
- scmap's unassigned cells are excluded from scoring, which affects comparability of its raw accuracy
- The full dataset list is in Supplementary Table 1/5 and is not enumerated in the main text

## Evidence

- [discussion] we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [discussion] For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions: "For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions"
- [discussion] we performed the classification using the function scmapCluster of scmap without considering the unassigned cells: "we performed the classification using the function scmapCluster of scmap without considering the unassigned cells"
- [discussion] we selected genes with top ΔS as informative genes (500 by default): "we selected genes with top ΔS as informative genes (500 by default)"
- [readme] The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts).: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
