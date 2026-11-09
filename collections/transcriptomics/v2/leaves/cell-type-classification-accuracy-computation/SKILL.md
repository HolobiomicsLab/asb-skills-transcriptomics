---
name: cell-type-classification-accuracy-computation
description: Use when benchmarking supervised cell-type classifiers on annotated scRNA-seq
  data and you need repeatable accuracy and balanced-accuracy estimates for SciBet,
  scmap, and Seurat v3 across many datasets.
when_to_use_negative:
- Input data lacks curated cell-type labels, so a supervised hold-out benchmark cannot
  be run
- Goal is rejecting cells of unknown type rather than measuring classification accuracy
  — use SciBet's confidence score C (likelihood ratio vs a null dataset, cutoffs 0.001
  to 0.5) instead of plain accuracy
- Input is a small single-dataset query where a full 50-repeat cross-validation benchmark
  across 14 datasets is unnecessary
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SciBet
  role: 'Primary classifier: E-test feature selection, Laplace-smoothed multinomial
    training (Learn), maximum-likelihood prediction (Bet)'
  repo: github.com/PaulingLiu/scibet
- name: scmap
  role: Baseline classifier via the scmapCluster function, evaluated without considering
    unassigned cells
- name: Seurat v3
  role: Baseline classifier using FindTransferAnchors followed by TransferData
- name: scikit-learn (sklearn)
  role: Provides model_selection.train_test_split for the repeated stratified 7:3
    hold-out splits
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:2b885e9328f941e390950f5b637fac1bb64494eccebdc7f162f243744b27677e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciBet
  - Seurat v3
  - scikit-learn (sklearn)
  - scmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cell-type-classification-accuracy-computation

## Summary

Reproduces the SciBet-vs-baseline cell-type classification benchmark: 50 repeats of stratified 7:3 hold-out splits on 14 scRNA-seq datasets, E-test feature selection of the top 500 informative genes, and accuracy/balanced-accuracy comparison of SciBet against scmapCluster and Seurat v3.

## When to use

Use when benchmarking supervised cell-type classifiers on annotated scRNA-seq data and you need repeatable accuracy and balanced-accuracy estimates for SciBet, scmap, and Seurat v3 across many datasets.

## When NOT to use

- Input data lacks curated cell-type labels, so a supervised hold-out benchmark cannot be run
- Goal is rejecting cells of unknown type rather than measuring classification accuracy — use SciBet's confidence score C (likelihood ratio vs a null dataset, cutoffs 0.001 to 0.5) instead of plain accuracy
- Input is a small single-dataset query where a full 50-repeat cross-validation benchmark across 14 datasets is unnecessary

## Inputs

- 14 annotated single-cell RNA-seq benchmarking datasets enumerated in Supplementary Table 1 (full-length and UMI count matrices)
- Per-dataset cell type labels for training/test stratification
- SciBet R package functions: SelectGene, Entropy, Learn, Bet
- scmap (scmapCluster) and Seurat v3 (FindTransferAnchors, TransferData) baseline packages

## Outputs

- benchmark_accuracy_results.csv — a 14-dataset × 3-method table of accuracy and balanced-accuracy aggregated over 50 repeated splits
- Per-split top-500 informative gene sets selected by ΔS (E-test)

## How to apply

Load each of the 14 benchmarking scRNA-seq datasets and normalize per platform convention: TPM plus pseudo-count 1 with log-normalization for full-length data, or Seurat v3 preprocessing (size-factor 10,000, plus-one, log normalization) for UMI data. For each dataset, use sklearn's model_selection.train_test_split to draw stratified, without-replacement 7:3 train/test splits, repeating the full procedure 50 times. On each split, apply the E-test (SelectGene/Entropy) to rank genes by the ΔS statistic and retain the top 500 informative genes, then train SciBet via Learn with Laplace-smoothed multinomial parameters and classify test cells by maximum likelihood via Bet. Classify the identical splits with the baselines — scmap's scmapCluster function ignoring unassigned cells, and Seurat v3's FindTransferAnchors followed by TransferData. Aggregate accuracy and balanced accuracy over the 50 repeats into a 14-dataset × 3-method table and confirm SciBet ranks highest on the majority of datasets, consistent with Fig. 2c and 2d.

## Related tools

- **SciBet** (Primary classifier: E-test feature selection, Laplace-smoothed multinomial training (Learn), maximum-likelihood prediction (Bet)) — github.com/PaulingLiu/scibet
- **scmap** (Baseline classifier via the scmapCluster function, evaluated without considering unassigned cells)
- **Seurat v3** (Baseline classifier using FindTransferAnchors followed by TransferData)
- **scikit-learn (sklearn)** (Provides model_selection.train_test_split for the repeated stratified 7:3 hold-out splits)

## Examples

```
devtools::install_github("PaulingLiu/scibet")
```

## Evaluation signals

- benchmark_accuracy_results.csv contains all 14 datasets × 3 methods with accuracy and balanced-accuracy values within [0, 1]
- SciBet attains the highest mean accuracy and balanced accuracy on the majority of the 14 datasets, matching the ordering reported in Fig. 2c and 2d
- Each split retains exactly 500 informative genes ranked by top ΔS from the E-test
- The 70/30 split is stratified and sampled without replacement, and the full procedure is repeated 50 times per dataset
- Unassigned cells are excluded from the scmapCluster accuracy computation, mirroring the paper's protocol

## Limitations

- No random seed or randomization parameters are reported for the 50 repeated splits, so exact numerical reproduction is not guaranteed
- Software versions for scikit-learn, Seurat v3 minor version, and imbalanced-learn are not specified, which may affect baseline comparability
- Accuracy comparison ignores scmap's unassigned cells, which favors scmap relative to a strict label-assignment protocol

## Evidence

- [discussion] we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [discussion] For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions: "For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions"
- [discussion] we performed the classification using the function scmapCluster of scmap without considering the unassigned cells: "we performed the classification using the function scmapCluster of scmap without considering the unassigned cells"
- [discussion] we selected genes with top ΔS as informative genes (500 by default): "we selected genes with top ΔS as informative genes (500 by default)"
- [readme] devtools::install_github("PaulingLiu/scibet"): "devtools::install_github("PaulingLiu/scibet")"
