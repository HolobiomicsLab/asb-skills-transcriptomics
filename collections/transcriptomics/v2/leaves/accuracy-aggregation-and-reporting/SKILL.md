---
name: accuracy-aggregation-and-reporting
description: Use when benchmarking cell type classifiers over repeated stratified
  train/test splits and you must average per-repeat accuracy into a mean accuracy
  table and verify a reported method ordering.
when_to_use_negative:
- Test cells lack ground-truth labels (e.g., look-up or negative-control scenarios
  like GSE108989 T cells absent from the reference) where accuracy is undefined and
  SciBet's confidence-score/false-positive analysis is the appropriate evaluation
  instead
- The task is benchmarking speed or portability rather than predictive accuracy (SciBet's
  headline claim is an order-of-magnitude speed advantage, which this accuracy-aggregation
  skill does not measure)
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: SciBet
  role: Produces per-repeat predictions (SelectGene, Learn, Bet) whose classification
    accuracy is averaged across the 50 repeats per dataset pair
  repo: https://github.com/PaulingLiu/scibet
- name: Seurat v3
  role: Baseline annotator (FindTransferAnchors + TransferData) whose per-repeat accuracy
    is aggregated into the comparison matrix
- name: scmap
  role: Baseline annotator (scmapCluster, unassigned cells ignored) included in the
    mean accuracy table
- name: scikit-learn (sklearn)
  role: train_test_split generates the stratified 7:3 splits that define each repeat
    to be scored and averaged
- name: imbalanced-learn
  role: RandomUnderSampler down-sampling used with train_test_split in the repeated
    split procedure feeding the accuracy aggregation
- name: tidyverse
  role: R stack used to compute per-repeat accuracy means and write the accuracy table
    CSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:a33cc4e3344b3d7d6e72862841ea02399a4159adddd6f2a4eb9ccb8ac0c9ce60
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - Rcpp
  - SciBet
  - Seurat v3
  - imbalanced-learn
  - scikit-learn (sklearn)
  - scmap
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# accuracy-aggregation-and-reporting

## Summary

Aggregates per-repeat classification accuracies across repeated stratified 7:3 train/test splits into a mean accuracy matrix (dataset pairs × methods) and validates the reported method ordering, as done for SciBet vs. Seurat v3 vs. scmap on cross-platform pancreas pairs.

## When to use

Use when benchmarking cell type classifiers over repeated stratified train/test splits and you must average per-repeat accuracy into a mean accuracy table and verify a reported method ordering.

## When NOT to use

- Test cells lack ground-truth labels (e.g., look-up or negative-control scenarios like GSE108989 T cells absent from the reference) where accuracy is undefined and SciBet's confidence-score/false-positive analysis is the appropriate evaluation instead
- The task is benchmarking speed or portability rather than predictive accuracy (SciBet's headline claim is an order-of-magnitude speed advantage, which this accuracy-aggregation skill does not measure)

## Inputs

- Per-repeat classification accuracy values (50 repeats × 6 dataset pairs × 3 methods: SciBet, Seurat v3, scmap)
- Normalized scRNA-seq train/test matrices per pair (TPM + pseudocount 1 + log for full-length data; size-factor 10,000 + add 1 + log for UMI data)
- Cell type labels for the held-out test sets

## Outputs

- Mean accuracy matrix as a CSV table (6 pairs × SciBet/Seurat v3/scmap columns), e.g., fig3a_benchmark_accuracy.csv
- Ordering verdict: SciBet ≥ Seurat v3 > scmap, consistent with Fig. 3a

## How to apply

For each ordered dataset pair (e.g., the 6 cross-platform human pancreas pairs), form stratified 7:3 train/test splits without replacement using sklearn's model_selection.train_test_split with imbalanced-learn RandomUnderSampler down-sampling, repeating the entire sampling-and-classification procedure 50 times as the paper prescribes. On each repeat, classify the held-out test cells with each method — SciBet's Bet (after SelectGene top-500 ΔS feature selection and Learn with Laplace smoothing), Seurat v3's FindTransferAnchors + TransferData, and scmapCluster ignoring unassigned cells — and compute per-repeat classification accuracy. Average the 50 per-repeat accuracies per pair per method to obtain the mean accuracy matrix, and write it (6 pairs × SciBet/Seurat v3/scmap) to a CSV such as fig3a_benchmark_accuracy.csv. Judge correctness by the schema (all pairs, all method columns), the [0,1] range, and the expected ordering: SciBet slightly above Seurat v3, both above scmap.

## Related tools

- **SciBet** (Produces per-repeat predictions (SelectGene, Learn, Bet) whose classification accuracy is averaged across the 50 repeats per dataset pair) — https://github.com/PaulingLiu/scibet
- **Seurat v3** (Baseline annotator (FindTransferAnchors + TransferData) whose per-repeat accuracy is aggregated into the comparison matrix)
- **scmap** (Baseline annotator (scmapCluster, unassigned cells ignored) included in the mean accuracy table)
- **scikit-learn (sklearn)** (train_test_split generates the stratified 7:3 splits that define each repeat to be scored and averaged)
- **imbalanced-learn** (RandomUnderSampler down-sampling used with train_test_split in the repeated split procedure feeding the accuracy aggregation)
- **tidyverse** (R stack used to compute per-repeat accuracy means and write the accuracy table CSV)

## Examples

```
import pandas as pd; df.groupby(['pair','method'])['accuracy'].mean().unstack().to_csv('fig3a_benchmark_accuracy.csv')
```

## Evaluation signals

- Output CSV contains all 6 dataset pairs and all 3 method columns (SciBet, Seurat v3, scmap) with no missing cells
- Every mean accuracy value lies in [0,1]
- Mean accuracy ordering holds: SciBet ≥ Seurat v3 > scmap, matching the reported Fig. 3a result
- Each reported mean is the average of 50 independent repeats (the paper's prescribed repetition count)
- The stratified 7:3 split ratio is respected within each repeat

## Limitations

- No seed or randomization parameters were reported for the 50× repeated splits, so exact numeric reproduction of the mean accuracy table is not guaranteed
- scmap accuracy was computed ignoring unassigned cells, which can inflate its accuracy relative to a full-assignment accounting
- Software versions (scikit-learn, imbalanced-learn, Seurat v3 minor version) are unspecified, so baseline results may drift across versions

## Evidence

- [discussion] Repeated 7:3 stratified splits: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [discussion] Full procedure repeated per iteration: "we repeated the entire procedure including random sampling and classification for 50 times."
- [discussion] Baselines to score against: "For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions"
- [discussion] scmap treatment in scoring: "we performed the classification using the function scmapCluster of scmap without considering the unassigned cells"
- [readme] Reproduction scripts availability: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
