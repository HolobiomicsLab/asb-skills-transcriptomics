---
name: feature-selection-method-comparison
description: Use when the goal is to benchmark which gene/feature-selection method
  (e.g., E-test vs F-test vs M3Drop) yields the highest cell-classification accuracy,
  holding the classifier and train/test protocol fixed across methods and multiple
  labeled scRNA-seq datasets.
when_to_use_negative:
- Input data lacks cell-type labels — stratified 70/30 train/test splits and classification
  accuracy are undefined without annotations
- The analysis goal is cell-type classification itself (e.g., annotation of new data
  with SciBet/scmap) rather than comparing which gene-selection method produces the
  best classifier
- Only a single small dataset is available — the benchmark's conclusion (E-test consistently
  highest across 14 datasets) is a cross-dataset comparison, not a per-dataset recommendation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn (sklearn)
  role: Implements the repeated stratified 7:3 hold-out splits via model_selection.train_test_split
- name: scmap
  role: 'Fixed classifier for the comparison: scmapCluster annotates test cells, without
    considering unassigned cells'
- name: Seurat v3
  role: UMI-data preprocessing (size factor 10,000, add one, log normalization) applied
    identically before feature selection
- name: SciBet
  role: Source of the E-test / top-ΔS feature selection (SelectGene, 500 genes by
    default) being benchmarked
  repo: github.com/PaulingLiu/scibet
- name: Python
  role: Implementation language for the benchmark loop and aggregation into fig2a_feature_selection_accuracy.csv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:5ee1aa012187a15061f7e49c8fbed2659c6c7e3d98e7c3f8ad0ad0b6f414c54a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - SciBet
  - Seurat v3
  - scikit-learn (sklearn)
  - scmap
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# feature-selection-method-comparison

## Summary

Benchmarks gene feature-selection methods for single-cell classification (E-test, F-test/one-way ANOVA, M3Drop) by pairing each with an identical classifier and split protocol, so accuracy differences are attributable to gene selection alone. Used to reproduce the Fig. 2a comparison across 14 scRNA-seq benchmarking datasets from the SciBet paper.

## When to use

Use when the goal is to benchmark which gene/feature-selection method (e.g., E-test vs F-test vs M3Drop) yields the highest cell-classification accuracy, holding the classifier and train/test protocol fixed across methods and multiple labeled scRNA-seq datasets.

## When NOT to use

- Input data lacks cell-type labels — stratified 70/30 train/test splits and classification accuracy are undefined without annotations
- The analysis goal is cell-type classification itself (e.g., annotation of new data with SciBet/scmap) rather than comparing which gene-selection method produces the best classifier
- Only a single small dataset is available — the benchmark's conclusion (E-test consistently highest across 14 datasets) is a cross-dataset comparison, not a per-dataset recommendation

## Inputs

- 14 labeled scRNA-seq benchmarking datasets with accession numbers (Supplementary Table 1 of the SciBet paper)
- Per-dataset raw/full-length or UMI count matrices with cell-type labels
- Feature-selection method definitions: E-test (ΔS ranking), F-test/one-way ANOVA, M3Drop

## Outputs

- Per-dataset mean classification accuracy for each of the three feature-selection methods (aggregated over 50 repeated 70/30 splits)
- Comparison table written to fig2a_feature_selection_accuracy.csv
- Ranking of feature-selection methods by mean accuracy (expected: E-test highest)

## How to apply

Retrieve all 14 benchmarking scRNA-seq datasets listed in Supplementary Table 1 via their accession numbers, and normalize each appropriately: TPM plus log-normalization (add pseudo-value one) for full-length reads, or Seurat v3 UMI preprocessing (size factor 10,000, add one, log normalization) for UMI data. For each dataset, rank genes with each of the three methods — E-test's ΔS statistic (top ΔS, 500 genes by default, as in the SciBet-style SelectGene), F-test/one-way ANOVA, and M3Drop — so all methods feed the same number of informative genes into the classifier. Hold the classifier fixed by using scmapCluster from scmap and ignore unassigned cells when scoring. Form stratified random train/test splits without replacement at 7:3 (train_test_split from scikit-learn), repeat the full procedure 50 times per dataset-method pair, and aggregate the 50 repetitions into a per-dataset mean accuracy for each method.

## Related tools

- **scikit-learn (sklearn)** (Implements the repeated stratified 7:3 hold-out splits via model_selection.train_test_split)
- **scmap** (Fixed classifier for the comparison: scmapCluster annotates test cells, without considering unassigned cells)
- **Seurat v3** (UMI-data preprocessing (size factor 10,000, add one, log normalization) applied identically before feature selection)
- **SciBet** (Source of the E-test / top-ΔS feature selection (SelectGene, 500 genes by default) being benchmarked) — github.com/PaulingLiu/scibet
- **Python** (Implementation language for the benchmark loop and aggregation into fig2a_feature_selection_accuracy.csv)

## Examples

```
from sklearn.model_selection import train_test_split; tr, te = train_test_split(cells, train_size=0.7, stratify=labels)  # then scmapCluster, 50 repeats
```

## Evaluation signals

- Per-dataset mean accuracies for E-test, F-test, and M3Drop must reproduce the relative ordering and magnitudes of Fig. 2a of the SciBet paper
- Each dataset-method pair must be evaluated over exactly 50 stratified 70:3 train/test repetitions (train_size=0.7, stratify=cell labels, no replacement)
- All three methods must select the same number of genes per dataset (top ΔS, 500 by default) so the comparison is not confounded by gene-set size
- Classification must use scmapCluster on every method's selected genes, with unassigned cells excluded from accuracy computation
- Output table fig2a_feature_selection_accuracy.csv must contain one mean accuracy per dataset per method (14 rows × 3 methods)

## Limitations

- Unassigned scmap cells are ignored when computing accuracy, which removes false-positive/negative-control effects from the benchmark
- No seed or randomization parameters are reported for the 50× repeated splits, so exact per-repetition accuracy cannot be bitwise reproduced
- The 14 datasets are enumerated only in Supplementary Table 1, which is not included in the article body; accession numbers must be obtained separately
- Software versions for scikit-learn, scmap, Seurat v3, and M3Drop are not specified, which may affect reproducibility

## Evidence

- [discussion] 50-repeat stratified split protocol: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [discussion] fixed classifier via scmapCluster: "we performed the classification using the function scmapCluster of scmap without considering the unassigned cells"
- [discussion] ΔS-based gene selection, top 500: "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] sklearn hold-out implementation: [evidence span withheld — anchor status: not_found]
- [readme] reproduction scripts location: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
