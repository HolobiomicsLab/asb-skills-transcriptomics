---
name: confusion-matrix-normalization-and-visualization
description: Use when validating a supervised cell-type classifier (e.g., SciBet)
  via repeated stratified train/test splits and you need normalized confusion matrices
  to show discrimination of close subtypes and rare cell types.
when_to_use_negative:
- The goal is unsupervised clustering of cells rather than benchmarking a supervised
  label-predicting classifier against known labels
- The reference set does not cover all present cell types — in that case use SciBet's
  confidence-score/null-dataset strategy instead of a closed-set confusion matrix
- Input data are already aggregated classification counts rather than per-cell predictions
  from repeated splits
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3318
tools:
- name: SciBet
  role: Provides SelectGene (top-500 ΔS genes), Learn (Laplace-smoothed multinomial
    MLE), Bet (test-cell classification), and Confusion_heatmap used to build and
    visualize the confusion matrices
  repo: github.com/PaulingLiu/scibet
- name: scikit-learn (sklearn)
  role: model_selection.train_test_split performs the stratified 7:3 train/test splits
    repeated 50 times before matrix accumulation
- name: imbalanced-learn
  role: under_sampling.RandomUnderSampler down-samples PBMC to 2,500 cells per cell
    type so the PBMC confusion matrix reflects a balanced design
- name: Python
  role: Hosts the repeated sampling-and-classification procedure and accumulation
    of predictions into normalized confusion matrices
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:fbe31486ab854e877fc383a2e450fe25a3c1fb27489531fe52bad9b039eb2fac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciBet
  - imbalanced-learn
  - scikit-learn (sklearn)
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# confusion-matrix-normalization-and-visualization

## Summary

Normalize accumulated cell-type classification predictions into per-class confusion matrices and render them as heatmaps to benchmark single-cell classifiers (SciBet vs scmap vs Seurat v3) on balanced and imbalanced scRNA-seq datasets.

## When to use

Use when validating a supervised cell-type classifier (e.g., SciBet) via repeated stratified train/test splits and you need normalized confusion matrices to show discrimination of close subtypes and rare cell types.

## When NOT to use

- The goal is unsupervised clustering of cells rather than benchmarking a supervised label-predicting classifier against known labels
- The reference set does not cover all present cell types — in that case use SciBet's confidence-score/null-dataset strategy instead of a closed-set confusion matrix
- Input data are already aggregated classification counts rather than per-cell predictions from repeated splits

## Inputs

- Annotated scRNA-seq expression matrix (e.g., fresh PBMC dataset, human pancreas dataset) with cell-type labels
- Per-repeat classification predictions from SciBet/Bet over 50 stratified 7:3 train/test splits

## Outputs

- Normalized confusion matrices per dataset (one for balanced PBMC, one for imbalanced pancreas)
- Heatmap renderings of the normalized confusion matrices (Confusion_heatmap)
- normalized_confusion_matrices.csv

## How to apply

First generate predictions: split the scRNA-seq dataset with sklearn's stratified model_selection.train_test_split (7:3 train/test ratio) and repeat the entire sampling-and-classification procedure 50 times. For each split, select the top 500 informative genes by the ΔS statistic with SciBet's SelectGene function, train the multinomial maximum-likelihood model with Learn (with Laplace smoothing), and classify held-out test cells with SciBet/Bet. Accumulate predictions over the 50 repeats and compute the normalized confusion matrix per dataset (e.g., balanced PBMC down-sampled to 2,500 cells/type via imbalanced-learn's under_sampling.RandomUnderSampler, and the imbalanced human pancreas dataset). Render each normalized confusion matrix as a heatmap (Confusion_heatmap function) and write the matrices to normalized_confusion_matrices.csv.

## Related tools

- **SciBet** (Provides SelectGene (top-500 ΔS genes), Learn (Laplace-smoothed multinomial MLE), Bet (test-cell classification), and Confusion_heatmap used to build and visualize the confusion matrices) — github.com/PaulingLiu/scibet
- **scikit-learn (sklearn)** (model_selection.train_test_split performs the stratified 7:3 train/test splits repeated 50 times before matrix accumulation)
- **imbalanced-learn** (under_sampling.RandomUnderSampler down-samples PBMC to 2,500 cells per cell type so the PBMC confusion matrix reflects a balanced design)
- **Python** (Hosts the repeated sampling-and-classification procedure and accumulation of predictions into normalized confusion matrices)

## Examples

```
if (!requireNamespace("devtools", quietly = TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/scibet")
```

## Evaluation signals

- Reproduced confusion matrices match the published Fig. 2f and Fig. 2g patterns: CD4 memory vs CD4 regulatory T cells are discriminated in the PBMC matrix, and rare delta cells (~3.8% of pancreatic cells) are correctly identified
- The PBMC matrix reflects a balanced design of 2,500 cells per cell type (7 cell types) produced by RandomUnderSampler down-sampling
- Matrices are computed from 50 repeated stratified 7:3 splits, not a single hold-out
- normalized_confusion_matrices.csv contains both matrices, with rows/columns normalized per class
- Feature selection in every split uses the top 500 ΔS-ranked informative genes with Laplace-smoothed multinomial parameters

## Limitations

- No seed or randomization parameters are reported for the 50× repeated splits, so exact numeric reproduction of the matrices is not guaranteed
- scmap and Seurat v3 baselines do not effectively discriminate close subtypes (e.g., CD4 memory vs CD4 regulatory T cells), so their confusion matrices differ qualitatively from SciBet's
- Confusion matrices here treat classification as closed-set; unassigned cells are not modeled (scmap results were computed without considering unassigned cells)

## Evidence

- [discussion] stratified split repetition: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [methods] feature selection and smoothing: "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] Laplace smoothing of parameters: "pij across all gene i could be estimated by Xij after normalization across genes and Laplace Smoothing"
- [discussion] down-sampling strategy: "down sampling strategy proposed by python package imbalanced-learn22 (under_sampling.RandomUnderSampler)"
- [readme] reproduction scripts: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
