---
name: five-fold-cross-validation-splitting
description: Use when a single labeled scRNA-seq dataset (e.g. PBMC SCP424) must be
  split into train/test subsets to benchmark supervised cell-type identification methods
  under self-projection, before computing accuracy, F1-score and unlabeling rate.
when_to_use_negative:
- Cross-platform prediction where training and testing datasets come from different
  studies (e.g. seven-platform PBMC training vs six-platform testing) — the split
  is already defined by dataset origin, not by within-dataset folding
- Datasets already partitioned with caret's createDataPartition (the paper's alternative
  single-split scheme) or with a predefined train/test division — do not re-fold
- Datasets with very few cells per cell type, where a fifth of cells held out per
  fold leaves training folds without representatives of a type
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: caret (R package)
  role: createFolds is used to generate the five cross-validation folds over the labeled
    cells
- name: AutomaticCellTypeIdentification
  role: wrapper whose eagersupervised/lazysupervised/markersupervised interfaces receive
    the four-fold training data and one-fold testing data in each round
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:4c3999bd748e64c14724f543120f9630178b7f29543a2393ea1f9e65134a1e5a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiebb123456/AutomaticCellTypeIdentification
  repo_url: https://github.com/xiebb123456/AutomaticCellTypeIdentification
  tools:
  - ACTINN
  - AutomaticCellTypeIdentification
  - CELLBLAST
  - CaSTLe
  - CellFishing.jl
  - Garnett
  - R
  - SciBet
  - caret
  - caret (R package)
  - scID
  - scLearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# five-fold-cross-validation-splitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partitions a labeled scRNA-seq dataset (e.g. PBMC SCP424 count matrix with manual labels) into five folds with caret's createFolds and runs five rounds of cross-validation (four folds training, one testing) so automatic cell-type identification methods can be compared on accuracy, mean F1-score and unlabeling rate.

## When to use

Use when a single labeled scRNA-seq dataset (e.g. PBMC SCP424) must be split into train/test subsets to benchmark supervised cell-type identification methods under self-projection, before computing accuracy, F1-score and unlabeling rate.

## When NOT to use

- Cross-platform prediction where training and testing datasets come from different studies (e.g. seven-platform PBMC training vs six-platform testing) — the split is already defined by dataset origin, not by within-dataset folding
- Datasets already partitioned with caret's createDataPartition (the paper's alternative single-split scheme) or with a predefined train/test division — do not re-fold
- Datasets with very few cells per cell type, where a fifth of cells held out per fold leaves training folds without representatives of a type

## Inputs

- Preprocessed scRNA-seq count matrix (rows = genes, columns = cells) with manually annotated cell-type labels, e.g. PBMC SCP424 from the Single Cell Portal
- Number of folds (k = 5) and the label vector used for fold construction

## Outputs

- Five fold-index assignments over cells (caret createFolds output)
- Per-round trained models and predicted cell-type labels for each held-out fold
- Per method/per fold metrics: accuracy (excluding unlabeled cells), mean per-cell-type F1-score, unlabeling rate; aggregated into pbmc_cross_platform_results.csv

## How to apply

Load the preprocessed count matrix (genes not expressed in any cells removed; cells with <200 expressed genes, extreme gene counts >2 SD of the mean, >20% mitochondrial expression, or missing labels deleted) together with its manual cell-type labels. Use the createFolds function in the caret R package to split the cells into five folds, following the paper's self-projection design in which four subsets are the training data and the remaining subset is the testing data, repeated for five rounds of cross-validation. For each round, pass the four-fold training matrix, the held-out test matrix and the corresponding training labels to the AutomaticCellTypeIdentification package (e.g. eagersupervised, lazysupervised, markersupervised interfaces) with a consistent method argument. Repeat over all fold assignments, then compute per method and platform pair the accuracy (excluding unlabeled cells), mean per-cell-type F1-score, and unlabeling rate, and aggregate ranks (e.g. clustifyr, scLearn, scPred leading mean accuracy) into the results table. The rationale is that stratified repeated splitting removes dependence on one arbitrary train/test partition when ranking eager, lazy and marker-learning methods.

## Related tools

- **caret (R package)** (createFolds is used to generate the five cross-validation folds over the labeled cells)
- **AutomaticCellTypeIdentification** (wrapper whose eagersupervised/lazysupervised/markersupervised interfaces receive the four-fold training data and one-fold testing data in each round) — https://github.com/xiebb123456/AutomaticCellTypeIdentification

## Examples

```
folds <- caret::createFolds(label_train, k = 5); pred <- lapply(folds, function(te) AutomaticCellTypeIdentification::eagersupervised(train[, -te], test, label_train[-te], method='Seurat'))
```

## Evaluation signals

- The five folds are disjoint and cover every labeled cell exactly once, so each cell appears in exactly one held-out test fold across the five rounds
- Each round's training set contains exactly four of the five folds, matching the 'four subsets training, remaining subset testing' design
- Method calls per round use identical usage formats across methods (same eagersupervised/lazysupervised/markersupervised signatures)
- Reported accuracy excludes unlabeled cells and F1-score is the mean over per-cell-type F1 values
- Final metric table covers every method and platform pair over the five rounds (e.g. exported to pbmc_cross_platform_results.csv)

## Limitations

- Marker-learning methods (markersupervised interface) take a marker list rather than a training matrix, so the same folds cannot be applied uniformly — the paper's marker methods (CellAssign, SCSA, MarkerCount) performed best in their category but with slightly lower accuracy and F1-score than top eager methods
- Self-projection cross-validation measures within-dataset generalization only; it does not predict cross-platform performance
- The caret package also offers createDataPartition, which is a different (single-split) scheme — mixing the two functions changes the evaluation design

## Evidence

- [readme] The R wrapper interfaces that consume each train/test split.: [evidence span withheld — anchor status: external_doc]
- [readme] Input format expected for training and testing data in each fold.: [evidence span withheld — anchor status: external_doc]
