---
name: class-imbalance-random-under-sampling
description: Use when benchmarking a cell-type classifier (e.g., SciBet vs. scmap/Seurat
  v3) on datasets with unequal cells per type, where rare or closely related subtypes
  could be hidden by class imbalance in the confusion matrix.
when_to_use_negative:
- When the goal is to test performance on rare cell types under realistic prevalence
  — the paper deliberately keeps the human pancreas dataset imbalanced (delta cells
  at 3.8%) for that purpose.
- When losing majority-class data is unacceptable (under-sampling discards cells);
  consider alternatives rather than discarding samples when the majority class is
  very small overall.
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: imbalanced-learn
  role: Python package providing under_sampling.RandomUnderSampler used to down-sample
    the PBMC scRNA-seq data to a balanced per-class design
- name: scikit-learn (sklearn)
  role: Provides model_selection.train_test_split for the stratified 7:3 train/test
    splits applied after under-sampling in each of 50 repeats
- name: SciBet
  role: Downstream multinomial maximum-likelihood classifier (SelectGene, Learn, Bet)
    whose balanced cross-validation accuracy this sampling enables
  repo: https://github.com/PaulingLiu/scibet
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-15523-2
    title: SciBet as a portable and fast single cell type identifier
  source_content_hash: sha256:0802af3406921935bba0489feee3b42e2405062270c84286661db107378a4fd9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/scibet
  tools:
  - Python
  - SciBet
  - imbalanced-learn
  - scikit-learn (sklearn)
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# class-imbalance-random-under-sampling

## Summary

Balances an imbalanced scRNA-seq cell-type label distribution by randomly under-sampling majority classes with imbalanced-learn's RandomUnderSampler, so that per-class accuracy (e.g., rare pancreatic delta cells or confusable CD4 T-cell subtypes) can be fairly evaluated in a confusion matrix rather than being masked by majority-class prevalence.

## When to use

Use when benchmarking a cell-type classifier (e.g., SciBet vs. scmap/Seurat v3) on datasets with unequal cells per type, where rare or closely related subtypes could be hidden by class imbalance in the confusion matrix.

## When NOT to use

- When the goal is to test performance on rare cell types under realistic prevalence — the paper deliberately keeps the human pancreas dataset imbalanced (delta cells at 3.8%) for that purpose.
- When losing majority-class data is unacceptable (under-sampling discards cells); consider alternatives rather than discarding samples when the majority class is very small overall.

## Inputs

- Annotated single-cell expression matrix (scRNA-seq counts, e.g., fresh PBMC dataset) with per-cell type labels
- Label vector of cell types with unequal class sizes (imbalanced classes)

## Outputs

- Balanced expression matrix with equal cells per class (e.g., 2,500 cells per PBMC cell type across 7 types)
- Repeated stratified train/test splits (7:3, 50 repeats) over the balanced data feeding classifier confusion matrices

## How to apply

Apply imbalanced-learn's under_sampling.RandomUnderSampler (Python) to down-sample the annotated scRNA-seq matrix so every class has equal cell counts — in the source task, the fresh PBMC dataset was down-sampled to a balanced 2,500 cells per cell type across 7 cell types. Re-run the entire sampling-and-classification procedure (stratified 7:3 train/test split via sklearn model_selection.train_test_split, repeated 50 times) so the random under-sampling is regenerated in each repeat rather than done once. The rationale is that a balanced design lets the normalized confusion matrix reveal discrimination of close subtypes (CD4 memory vs. CD4 regulatory T cells) independently of class prior; a separate imbalanced dataset (human pancreas, delta cells at 3.8%) is kept unbalanced to test rare-type performance.

## Related tools

- **imbalanced-learn** (Python package providing under_sampling.RandomUnderSampler used to down-sample the PBMC scRNA-seq data to a balanced per-class design)
- **scikit-learn (sklearn)** (Provides model_selection.train_test_split for the stratified 7:3 train/test splits applied after under-sampling in each of 50 repeats)
- **SciBet** (Downstream multinomial maximum-likelihood classifier (SelectGene, Learn, Bet) whose balanced cross-validation accuracy this sampling enables) — https://github.com/PaulingLiu/scibet

## Examples

```
from imblearn.under_sampling import RandomUnderSampler; X_res, y_res = RandomUnderSampler(sampling_strategy={'CD4 Memory': 2500, 'Treg': 2500}).fit_resample(X, y)
```

## Evaluation signals

- The down-sampled PBMC design must contain exactly 2,500 cells per cell type (7 cell types) in each repeat.
- The balanced and imbalanced datasets must be processed identically (stratified 7:3 split, 50 repeats) so confusion matrices are comparable.
- Reproduced normalized confusion matrix must match published Fig. 2f pattern: correct discrimination of CD4 memory T cells vs. CD4 regulatory T cells.
- On the imbalanced pancreas matrix, rare delta cells (3.8% of cells) must still be correctly identified (Fig. 2g pattern).
- Per-class counts in each train/test split should reflect stratified sampling, preserving the 7:3 ratio within every class.

## Limitations

- No seed or randomization parameters were reported for the 50× repeated splits and sampling, so exact reproduction requires re-running the full sampling-and-classification loop 50 times rather than a single fixed split.
- Random under-sampling discards majority-class cells, so it estimates accuracy under a balanced prior rather than under true biological prevalence.
- Software version numbers for imbalanced-learn and scikit-learn are not specified in the article.

## Evidence

- [discussion] down sampling strategy proposed by python package imbalanced-learn22 (under_sampling.RandomUnderSampler): "down sampling strategy proposed by python package imbalanced-learn22 (under_sampling.RandomUnderSampler)"
- [discussion] stratified split repeated 50 times: "we applied the stratified random sampling without replacement to form the training and test set with the ratio 7:3, and repeated this process 50 times"
- [readme] README reproduction pointer: "The scripts for producing all the quantitative results in our manuscript can be found in [scripts](./scripts)."
