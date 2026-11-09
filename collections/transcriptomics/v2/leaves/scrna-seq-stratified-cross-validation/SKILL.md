---
name: scrna-seq-stratified-cross-validation
description: Use when benchmarking supervised cell-type identification methods on
  labeled scRNA-seq datasets and a fair, repeated train/test comparison of accuracy
  across datasets is needed.
when_to_use_negative:
- The task is de novo cell-type discovery or clustering rather than evaluating supervised
  classifiers against pre-annotated labels
- Input is a single dataset without ground-truth cell-type labels, so stratified train/test
  splits against reference labels are impossible
- You need to evaluate false-positive control on cell types absent from the reference
  (the null-dataset confidence-score workflow), which this within-reference CV setup
  does not address
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: scikit-learn (sklearn)
  role: Provides model_selection.train_test_split for the stratified 7:3 train/test
    splits repeated 50 times
- name: SciBet
  role: 'Primary classifier under test: Learn with Laplace smoothing, then maximum-likelihood
    assignment via Bet on the top 500 ΔS genes'
  repo: github.com/PaulingLiu/scibet
- name: scmap
  role: 'Baseline classifier: scmapCluster, with unassigned cells excluded from the
    accuracy computation'
- name: Seurat v3
  role: 'Baseline classifier: FindTransferAnchors followed by TransferData on the
    same splits'
- name: imbalanced-learn
  role: under_sampling.RandomUnderSampler used for down-sampling when balancing class
    representation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:cb531eb7ae8e1b2e372540257e41fee8f4578d13c4ff8f475821b1805cc30414
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SciBet
  - Seurat v3
  - imbalanced-learn
  - scikit-learn (sklearn)
  - scmap
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# scrna-seq-stratified-cross-validation

## Summary

A repeated stratified hold-out (70/30) cross-validation benchmark for comparing supervised single-cell RNA-seq cell-type classifiers (SciBet, scmap, Seurat v3) across 14 datasets, using E-test feature selection (top 500 ΔS genes) and accuracy/balanced-accuracy as the metrics.

## When to use

Use when benchmarking supervised cell-type identification methods on labeled scRNA-seq datasets and a fair, repeated train/test comparison of accuracy across datasets is needed.

## When NOT to use

- The task is de novo cell-type discovery or clustering rather than evaluating supervised classifiers against pre-annotated labels
- Input is a single dataset without ground-truth cell-type labels, so stratified train/test splits against reference labels are impossible
- You need to evaluate false-positive control on cell types absent from the reference (the null-dataset confidence-score workflow), which this within-reference CV setup does not address

## Inputs

- 14 labeled single-cell RNA-seq benchmark datasets (Supplementary Table 1), full-length (TPM-normalized) or UMI count matrices with cell-type labels
- Raw UMI count matrices for Seurat v3 preprocessing (size-factor 10,000, +1, log normalization)

## Outputs

- benchmark_accuracy_results.csv: 14-dataset × 3-method table of accuracy and balanced accuracy aggregated over 50 repeats
- Per-split top-500 informative gene sets ranked by the E-test ΔS statistic

## How to apply

Load each of the 14 benchmarking scRNA-seq datasets and normalize per platform convention: TPM plus pseudo-count 1 and log-normalization for full-length data; size-factor 10,000, plus-one, log normalization (Seurat v3 UMI preprocessing) for UMI data. For each dataset, use sklearn's model_selection.train_test_split to draw stratified random training/test splits without replacement at a 7:3 ratio, and repeat the entire sampling-plus-classification procedure 50 times to average out split randomness. On each split, rank genes by the E-test ΔS statistic and retain the top 500 informative genes, then train each classifier on the same splits: SciBet via Learn with Laplace-smoothed multinomial parameters and maximum-likelihood prediction (Bet), scmapCluster ignoring unassigned cells, and Seurat v3's FindTransferAnchors plus TransferData. Compute accuracy and balanced accuracy per method per dataset, aggregate over the 50 repeats, and export a 14-dataset × 3-method table. Validation is done by checking that SciBet achieves the highest mean accuracy and balanced accuracy on the majority of datasets, consistent with Fig. 2c and 2d.

## Related tools

- **scikit-learn (sklearn)** (Provides model_selection.train_test_split for the stratified 7:3 train/test splits repeated 50 times)
- **SciBet** (Primary classifier under test: Learn with Laplace smoothing, then maximum-likelihood assignment via Bet on the top 500 ΔS genes) — github.com/PaulingLiu/scibet
- **scmap** (Baseline classifier: scmapCluster, with unassigned cells excluded from the accuracy computation)
- **Seurat v3** (Baseline classifier: FindTransferAnchors followed by TransferData on the same splits)
- **imbalanced-learn** (under_sampling.RandomUnderSampler used for down-sampling when balancing class representation)

## Examples

```
devtools::install_github("PaulingLiu/scibet")  # then run scripts/ from github.com/PaulingLiu/scibet to reproduce the 50× 70/30 benchmark
```

## Evaluation signals

- benchmark_accuracy_results.csv contains exactly all 14 datasets and 3 methods, with accuracy values constrained to [0, 1]
- SciBet attains the highest mean accuracy and balanced accuracy on the majority of the 14 datasets, matching the ordering in Fig. 2c and 2d
- Each of the 50 repeats uses a 7:3 stratified split drawn without replacement (sklearn train_test_split), so class proportions in train and test sets match
- Feature selection retains exactly the top 500 genes ranked by ΔS on each split, recomputed per repeat rather than fixed across repeats
- scmapCluster results are computed with unassigned cells ignored, so its accuracy is not inflated by 'unassigned' calls

## Limitations

- No random seed or randomization parameters are reported for the 50× repeated splits, so exact numeric reproducibility may vary
- Unassigned cells from scmapCluster are ignored in the comparison, which can bias cross-method accuracy comparisons
- Software versions (scikit-learn, Seurat v3 minor version, imbalanced-learn) are not reported in the article text
- The 14 benchmark datasets are only enumerated in Supplementary Table 1, not in the section body

## Evidence

- [discussion] we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [discussion] We implemented the hold-out validation strategy proposed by the python package sklearn9 (function model_selection.train_test_split): "We implemented the hold-out validation strategy proposed by the python package sklearn9 (function model_selection.train_test_split)"
- [discussion] we selected genes with top ΔS as informative genes (500 by default): "we selected genes with top ΔS as informative genes (500 by default)"
- [discussion] For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions: "For Seurat v3, we identified anchors and classified cells using the FindTransferAnchors function and TransferData functions"
- [discussion] we performed the classification using the function scmapCluster of scmap without considering the unassigned cells: "we performed the classification using the function scmapCluster of scmap without considering the unassigned cells"
- [readme] The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts).: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
