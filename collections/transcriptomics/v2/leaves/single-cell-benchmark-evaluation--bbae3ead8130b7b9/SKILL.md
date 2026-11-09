---
name: single-cell-benchmark-evaluation
description: 'Use when comparing cell-type classification methods on labeled scRNA-seq
  datasets: you need a repeated 7:3 train/test benchmark with per-dataset accuracy/balanced-accuracy
  tables aggregating 50 repeats.'
when_to_use_negative:
- The task is unsupervised clustering or de novo cell discovery — this benchmark presupposes
  an annotated training set and supervised classifiers
- Only a single dataset with too few cells per type is available for a stable 70/30
  stratified split repeated 50 times
- The query cells are of types absent from the reference (false-positive control via
  the null dataset/confidence score is a separate skill, not this accuracy benchmark)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SciBet
  role: 'Method under test: Learn fits Laplace-smoothed multinomial parameters; Bet
    classifies test cells by maximum likelihood over the 500 selected genes'
  repo: https://github.com/PaulingLiu/scibet
- name: scmap
  role: 'Baseline classifier: scmapCluster assigns hold-out cells; unassigned cells
    are ignored when scoring accuracy'
- name: Seurat v3
  role: 'Baseline classifier: FindTransferAnchors followed by TransferData for label
    transfer to the test set'
- name: scikit-learn (sklearn)
  role: Provides model_selection.train_test_split for the stratified, without-replacement
    7:3 train/test splitting repeated 50 times
- name: E-test (SelectGene/Entropy)
  role: 'Feature selection: ranks genes by the ΔS statistic; top 500 informative genes
    retained per split'
  repo: https://github.com/PaulingLiu/scibet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:8ef46a006e9c5d1d4a4e083037f4d59b8486910756f9c8b38986ff3eb1d1e84a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - E-test (SelectGene/Entropy)
  - SciBet
  - Seurat v3
  - scikit-learn (sklearn)
  - scmap
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-benchmark-evaluation

## Summary

Benchmark supervised single-cell classifiers (SciBet vs scmap vs Seurat v3) across multiple scRNA-seq datasets using repeated stratified 70/30 hold-out validation with E-test feature selection, reporting accuracy and balanced accuracy per dataset. It operationalizes the head-to-head comparison that established SciBet's accuracy advantage.

## When to use

Use when comparing cell-type classification methods on labeled scRNA-seq datasets: you need a repeated 7:3 train/test benchmark with per-dataset accuracy/balanced-accuracy tables aggregating 50 repeats.

## When NOT to use

- The task is unsupervised clustering or de novo cell discovery — this benchmark presupposes an annotated training set and supervised classifiers
- Only a single dataset with too few cells per type is available for a stable 70/30 stratified split repeated 50 times
- The query cells are of types absent from the reference (false-positive control via the null dataset/confidence score is a separate skill, not this accuracy benchmark)

## Inputs

- 14 labeled scRNA-seq benchmarking datasets enumerated in Supplementary Table 1 (e.g. GSE104276, GSE108989, Tabula Muris subsets)
- Per-dataset raw or normalized expression matrices (counts or TPM) with cell type annotations
- Normalization convention per dataset: full-length (TPM + pseudo-count 1 + log) or UMI (size-factor 10,000 + 1 + log)

## Outputs

- benchmark_accuracy_results.csv: 14 datasets × 3 methods table of accuracy and balanced accuracy aggregated over 50 repeats
- Per-dataset per-method classification predictions on the 30% hold-out cells

## How to apply

Load each of the 14 benchmarking datasets from Supplementary Table 1 and normalize by platform convention: TPM plus pseudo-count 1 with log-normalization for full-length data, and Seurat v3-style size-factor 10,000, plus-one, log normalization for UMI data. For each dataset, draw a stratified 7:3 training/test split without replacement (sklearn model_selection.train_test_split) and repeat the entire procedure 50 times. On each split, apply the E-test to rank genes by the ΔS statistic and retain the top 500 informative genes, then train SciBet (Learn with Laplace-smoothed multinomial parameters) and classify test cells by maximum likelihood (Bet). Run the same splits through scmapCluster (ignoring unassigned cells) and Seurat v3's FindTransferAnchors + TransferData as baselines. Aggregate accuracy and balanced accuracy over the 50 repeats into a 14-dataset × 3-method table and confirm SciBet ranks highest on the majority of datasets, matching Fig. 2c/2d ordering.

## Related tools

- **SciBet** (Method under test: Learn fits Laplace-smoothed multinomial parameters; Bet classifies test cells by maximum likelihood over the 500 selected genes) — https://github.com/PaulingLiu/scibet
- **scmap** (Baseline classifier: scmapCluster assigns hold-out cells; unassigned cells are ignored when scoring accuracy)
- **Seurat v3** (Baseline classifier: FindTransferAnchors followed by TransferData for label transfer to the test set)
- **scikit-learn (sklearn)** (Provides model_selection.train_test_split for the stratified, without-replacement 7:3 train/test splitting repeated 50 times)
- **E-test (SelectGene/Entropy)** (Feature selection: ranks genes by the ΔS statistic; top 500 informative genes retained per split) — https://github.com/PaulingLiu/scibet

## Examples

```
from sklearn.model_selection import train_test_split; X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.3, stratify=y)
```

## Evaluation signals

- The results table contains all 14 datasets and 3 methods, with every accuracy and balanced-accuracy value in [0, 1]
- SciBet achieves the highest mean accuracy and balanced accuracy on the majority of the 14 datasets, consistent with the ordering reported in Fig. 2c and 2d
- Each dataset-metric cell aggregates 50 independent 7:3 stratified repeats, not a single split
- Feature selection returns exactly 500 informative genes per split, ranked by the ΔS statistic
- scmap accuracy is computed after excluding unassigned cells, so per-method evaluated cell counts may differ and must be documented

## Limitations

- No seed or randomization parameters were reported for the 50× repeated splits or the ΔS permutation tests, so exact numeric reproduction of means may vary
- Software versions for scikit-learn, Seurat v3 minor version, scmap, and imbalanced-learn are not specified in the text
- Full enumeration of the 14 benchmarking datasets resides in Supplementary Table 1/5, which is referenced but not embedded in the main text
- Ignoring scmap's unassigned cells biases comparison toward each method's own evaluated subset; unassigned-cell handling differs across methods

## Evidence

- [discussion] stratified 7:3 split, 50 repeats: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [discussion] E-test top-500 gene selection: "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] Seurat v3 baseline: "For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions"
- [discussion] scmap baseline scoring rule: "we performed the classification using the function scmapCluster of scmap without considering the unassigned cells"
- [discussion] repetition definition: "we repeated the entire procedure including random sampling and classification for 50 times."
- [readme] repository scripts for reproduction: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
- [discussion] full-length normalization: "we calculated Transcript Per Million (TPM)20, added pseudo value one to handle 0 values and performed log-normalization"
