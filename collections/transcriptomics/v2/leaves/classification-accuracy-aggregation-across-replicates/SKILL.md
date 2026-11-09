---
name: classification-accuracy-aggregation-across-replicates
description: 'Use when you must fairly compare gene/feature-selection methods for
  cell type identification: hold the classifier constant, repeat randomized train/test
  splits many times, and aggregate per-repetition accuracies into per-dataset means
  before ranking methods.'
when_to_use_negative:
- When the classifier itself must vary across methods — this benchmark isolates feature-selection
  effects by holding the classifier (scmapCluster) fixed, so swapping classifiers
  confounds the comparison.
- When evaluating cell-type annotation tools with false-positive control on unrepresented
  cell types — this protocol ignores unassigned cells and measures accuracy only on
  covered types.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scmap
  role: Fixed downstream classifier (scmapCluster function) applied to test cells
    under every feature-selection condition; unassigned cells are not considered
- name: SciBet
  role: Source of the E-test (ΔS entropy-based) feature-selection statistic benchmarked
    against F-test and M3Drop
  repo: github.com/PaulingLiu/scibet
- name: scikit-learn (sklearn)
  role: Provides the stratified 7:3 train/test split via model_selection.train_test_split,
    repeated 50 times
- name: Seurat v3
  role: 'UMI-data preprocessing: size-factor 10,000 normalization, add-one, log transformation'
- name: Python
  role: Implementation language for splitting, classification, accuracy computation,
    and aggregation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:9a44468e2fa3d7c37ae4bfabf13deabdf693225c5c16967be80d9bfd06f5a07d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciBet
  - Seurat v3
  - scikit-learn (sklearn)
  - scmap
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# classification-accuracy-aggregation-across-replicates

## Summary

Benchmark protocol for comparing feature-selection methods (E-test, F-test/one-way ANOVA, M3Drop) by pairing each with a fixed classifier (scmapCluster) under 50× repeated stratified 70/30 hold-out splits across 14 scRNA-seq datasets, then aggregating per-repetition accuracies into per-dataset mean accuracies.

## When to use

Use when you must fairly compare gene/feature-selection methods for cell type identification: hold the classifier constant, repeat randomized train/test splits many times, and aggregate per-repetition accuracies into per-dataset means before ranking methods.

## When NOT to use

- When the classifier itself must vary across methods — this benchmark isolates feature-selection effects by holding the classifier (scmapCluster) fixed, so swapping classifiers confounds the comparison.
- When evaluating cell-type annotation tools with false-positive control on unrepresented cell types — this protocol ignores unassigned cells and measures accuracy only on covered types.

## Inputs

- 14 scRNA-seq benchmarking datasets retrieved via accession numbers from Supplementary Table 1
- Cell-level annotation labels for each dataset
- Preprocessed expression matrices (TPM + log-normalized for full-length; Seurat v3-normalized UMI with size-factor 10,000, add one, log normalization)

## Outputs

- Per-repetition classification accuracy for each dataset × feature-selection-method pair
- Aggregated per-dataset mean accuracy over 50 repetitions per method
- Comparison table written to fig2a_feature_selection_accuracy.csv

## How to apply

For each of the 14 benchmarking scRNA-seq datasets (Supplementary Table 1 accessions), preprocess with technology-appropriate normalization — TPM plus log-normalization with pseudo-count one for full-length reads, or Seurat v3 UMI preprocessing (size-factor 10,000, add one, log-normalization) for UMI data. Rank genes by each feature-selection method's statistic (E-test ΔS, F-test/ANOVA, M3Drop) and select the top 500 informative genes, keeping the downstream classifier fixed. Form stratified random train/test splits without replacement at a 7:3 ratio and repeat the entire procedure — including resampling and classification — 50 times. Classify test cells with scmapCluster from scmap, ignoring unassigned cells, and compute classification accuracy per repetition. Aggregate the 50 repetitions into a per-dataset mean accuracy per feature-selection method; the E-test consistently achieved the highest accuracy, matching Fig. 2a of the SciBet paper.

## Related tools

- **scmap** (Fixed downstream classifier (scmapCluster function) applied to test cells under every feature-selection condition; unassigned cells are not considered)
- **SciBet** (Source of the E-test (ΔS entropy-based) feature-selection statistic benchmarked against F-test and M3Drop) — github.com/PaulingLiu/scibet
- **scikit-learn (sklearn)** (Provides the stratified 7:3 train/test split via model_selection.train_test_split, repeated 50 times)
- **Seurat v3** (UMI-data preprocessing: size-factor 10,000 normalization, add-one, log transformation)
- **Python** (Implementation language for splitting, classification, accuracy computation, and aggregation)

## Examples

```
from sklearn.model_selection import train_test_split; accs = [np.mean(scmap_predict(X_train, y_train, X_test) == y_test) for _ in range(50) for (X_train, X_test, y_train, y_test) in [train_test_split(X, y, test_size=0.3, stratify=y)]]
```

## Evaluation signals

- Per-dataset mean accuracies for the three methods reproduce the relative ordering and magnitudes shown in Fig. 2a of the paper (E-test > F-test/ANOVA and M3Drop).
- Every dataset × method combination contains exactly 50 accuracy values from 50 independent repeated splits.
- Each split is stratified with a 7:3 train/test ratio drawn without replacement, verified by checking class proportions in train and test sets.
- The top-500 gene count is constant across all feature-selection methods so accuracy differences are attributable to gene ranking, not gene-set size.
- Aggregated table matches the schema of fig2a_feature_selection_accuracy.csv (per-dataset mean accuracy per method).

## Limitations

- No seed or randomization parameters are reported for the 50× repeated splits or permutation tests, so exact numeric reproduction is not guaranteed — only relative ordering and magnitudes.
- Unassigned cells are excluded from accuracy computation (scmapCluster run without considering unassigned cells), which inflates apparent accuracy versus real-world annotation scenarios with unknown cell types.
- Preprocessing is technology-dependent (TPM vs UMI); applying the wrong normalization to a dataset invalidates comparability.
- Feature selection uses the top 500 genes by default (top ΔS, SciBet-style); results at other gene counts are not covered by this benchmark.

## Evidence

- [discussion] we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [discussion] scmapCluster without unassigned cells: "we performed the classification using the function scmapCluster of scmap without considering the unassigned cells"
- [discussion] Seurat v3 UMI normalization parameters: "normalizing the UMI count of each cell with size-factor 10,000, adding one and then log normalization"
- [discussion] top ΔS 500 informative genes: "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] sklearn hold-out validation: "We implemented the hold-out validation strategy proposed by the python package sklearn9 (function model_selection.train_test_split)"
- [readme] README reproduction scripts pointer: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
