---
name: scmap-cluster-cell-type-classification
description: Use when you need to classify annotated scRNA-seq cells against a labeled
  reference with scmapCluster and compare classifiers or feature-selection methods
  by repeated hold-out accuracy.
when_to_use_negative:
- Test data contains cell types absent from the reference and you need false-positive
  control — the paper only scores scmapCluster 'without considering the unassigned
  cells', so unrepresented-type scenarios (e.g. GSE108989 T cells vs a reference lacking
  them) require a null-dataset-aware approach instead
- Cells are already classified/annotated and you only need per-gene informativeness
  ranking — use the feature-selection step (ΔS, E-test, F-test, M3Drop) rather than
  the classifier
- Data are raw unnormalized counts — scmapCluster must receive properly normalized
  expression (TPM + log for full-length; Seurat v3 size-factor 10,000, add-one, log
  for UMI)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scmap
  role: Provides the scmapCluster function used to classify 30% hold-out test cells
    against the training-cell reference; unassigned cells are ignored when computing
    accuracy
- name: scikit-learn (sklearn)
  role: Implements the stratified 7:3 train/test splitting via model_selection.train_test_split
    for the repeated hold-out validation
- name: Seurat v3
  role: UMI preprocessing (size-factor 10,000, add one, log normalization) before
    classification
- name: Python
  role: Language for the split/aggregation workflow and accuracy computation
- name: SciBet
  role: Feature-selection companion (ΔS-based top-500 gene selection) whose accuracy
    is benchmarked against scmapCluster's under identical splits
  repo: github.com/PaulingLiu/scibet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:1db3aebee691a1f0200f7d507136cfa04f9bdd8199ff04d80604d43ccaff7849
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
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
yes_rate: 0.8947368421052632
n_claims: 19
---

# scmap-cluster-cell-type-classification

## Summary

Benchmark-oriented cell-type classification using scmap's scmapCluster function: project 70/30 stratified hold-out test cells from scRNA-seq datasets onto a training-derived reference and score classification accuracy. In the SciBet paper this is one of the classifier backbones used to compare feature-selection methods (E-test, F-test/one-way ANOVA, M3Drop) across 14 benchmarking datasets.

## When to use

Use when you need to classify annotated scRNA-seq cells against a labeled reference with scmapCluster and compare classifiers or feature-selection methods by repeated hold-out accuracy.

## When NOT to use

- Test data contains cell types absent from the reference and you need false-positive control — the paper only scores scmapCluster 'without considering the unassigned cells', so unrepresented-type scenarios (e.g. GSE108989 T cells vs a reference lacking them) require a null-dataset-aware approach instead
- Cells are already classified/annotated and you only need per-gene informativeness ranking — use the feature-selection step (ΔS, E-test, F-test, M3Drop) rather than the classifier
- Data are raw unnormalized counts — scmapCluster must receive properly normalized expression (TPM + log for full-length; Seurat v3 size-factor 10,000, add-one, log for UMI)

## Inputs

- Annotated scRNA-seq count matrix (full-length read counts or UMI counts) with cell-type labels
- Dataset accession IDs for the 14 benchmarking scRNA-seq datasets (Supplementary Table 1)
- Selected informative gene list (e.g. top 500 ΔS genes, or E-test / F-test / M3Drop gene rankings)
- Train/test split specification: stratified 7:3 without replacement, 50 repetitions

## Outputs

- Per-repetition classification accuracy of scmapCluster on the 30% test cells (unassigned cells excluded)
- Per-dataset mean accuracy over 50 repetitions for each feature-selection method
- Comparison table, e.g. fig2a_feature_selection_accuracy.csv

## How to apply

Preprocess each dataset first: for full-length read data compute TPM, add a pseudo-value of one, and log-normalize; for UMI data apply Seurat v3 preprocessing (size-factor 10,000, add one, log normalization). Select informative genes per method (rank by ΔS, top 500 by default for the SciBet-style selection, or via E-test / F-test / M3Drop), then apply stratified random sampling without replacement to form training and test sets at a 7:3 ratio, repeating the entire procedure 50 times. Build the reference from the 70% training cells and run scmapCluster (from scmap) on the 30% test cells, discarding unassigned cells when scoring. Compute classification accuracy per repetition, average over the 50 repetitions to get a per-dataset mean accuracy for each feature-selection method, and write the comparison table (e.g. fig2a_feature_selection_accuracy.csv). Validate by checking that per-dataset mean accuracies reproduce the relative ordering and magnitudes reported in Fig. 2a, where E-test ranked highest.

## Related tools

- **scmap** (Provides the scmapCluster function used to classify 30% hold-out test cells against the training-cell reference; unassigned cells are ignored when computing accuracy)
- **scikit-learn (sklearn)** (Implements the stratified 7:3 train/test splitting via model_selection.train_test_split for the repeated hold-out validation)
- **Seurat v3** (UMI preprocessing (size-factor 10,000, add one, log normalization) before classification)
- **Python** (Language for the split/aggregation workflow and accuracy computation)
- **SciBet** (Feature-selection companion (ΔS-based top-500 gene selection) whose accuracy is benchmarked against scmapCluster's under identical splits) — github.com/PaulingLiu/scibet

## Examples

```
from sklearn.model_selection import train_test_split; X_tr, X_te, y_tr, y_te = train_test_split(X, y, test_size=0.3, stratify=y)  # then classify X_te with scmapCluster in R
```

## Evaluation signals

- Per-dataset mean accuracies (over 50 repetitions) match the relative ordering and magnitudes in Fig. 2a of the paper, with E-test > F-test/one-way ANOVA and M3Drop under the same scmapCluster classifier
- Split integrity check: every repetition uses a stratified 7:3 split without replacement (verify via sklearn train_test_split with stratify=y), and exactly 50 repetitions are aggregated per dataset-method pair
- Normalization invariant: full-length datasets are TPM + add-one + log transformed; UMI datasets use Seurat v3 size-factor 10,000 + add-one + log — confirm none are fed as raw counts
- Gene-count invariant: the classifier receives the same number of informative genes per method (top 500 by default for the SciBet/ΔS selection) so comparisons are fair
- Accuracy values fall in [0, 1] per repetition and unassigned cells are excluded from both numerator and denominator, per the scmapCluster scoring convention used

## Limitations

- Unassigned cells returned by scmapCluster are ignored during scoring, so false-positive behavior on cell types not represented in the reference is not measured in this benchmark setting
- No random seeds or randomization parameters are reported for the 50× repeated splits, so exact numerical reproduction of mean accuracies is not guaranteed
- The 14 benchmarking datasets are enumerated only by accession numbers in Supplementary Table 1, which is referenced but not fully enumerated in the section body; software versions (scikit-learn, Seurat v3 minor version) are not specified

## Evidence

- [discussion] scmap classification without unassigned cells: "we performed the classification using the function scmapCluster of scmap without considering the unassigned cells"
- [discussion] stratified 7:3 split repeated 50 times: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [discussion] Seurat v3 UMI normalization parameters: "normalizing the UMI count of each cell with size-factor 10,000, adding one and then log normalization"
- [discussion] ΔS top-500 feature selection: "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] sklearn hold-out split function: "We implemented the hold-out validation strategy proposed by the python package sklearn9 (function model_selection.train_test_split)"
