---
name: accuracy-f1-score-computation
description: Use when comparing cell-type identification methods on scRNA-seq data
  with known ground-truth labels, and per-method accuracy and F1-score rankings must
  be produced and verified.
when_to_use_negative:
- The predictions have no ground-truth cell-type labels to score against (e.g. purely
  exploratory clustering without manual annotation)
- The evaluation target is running time or unlabeling rate rather than label correctness
  — the article treats these as separate overall-performance dimensions
- Input is a cluster-level annotation mapping rather than per-cell predicted labels,
  so per-cell accuracy and F1 cannot be computed directly
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper that runs the automatic cell-type identification methods (eagersupervised,
    lazysupervised, markersupervised) whose predictions are then scored with accuracy
    and F1-score
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: caret
  role: Provides createDataPartition and createFolds to build the consistent train/test
    partitions over which accuracy and F1-score are computed in the five-fold cross-validation
    evaluation
- name: Seurat
  role: Used in the evaluation pipeline to find cell type specific markers; its clustering-based
    predictions are one of the scored baselines
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:66cce0273dc59908d97b8fb668eecc1048dc65828242178886e0fc3e9b50e849
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
  - AutomaticCellTypeIdentification
  - CellAssign
  - R
  - SCINA
  - Seurat
  - caret
  - scmap-cluster
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# accuracy-f1-score-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes prediction accuracy (correctly predicted cells divided by total labeled cells, excluding unlabeled) and mean per-cell-type F1-score (harmonic mean of precision and recall averaged over cell types) to benchmark automatic scRNA-seq cell-type identification methods.

## When to use

Use when comparing cell-type identification methods on scRNA-seq data with known ground-truth labels, and per-method accuracy and F1-score rankings must be produced and verified.

## When NOT to use

- The predictions have no ground-truth cell-type labels to score against (e.g. purely exploratory clustering without manual annotation)
- The evaluation target is running time or unlabeling rate rather than label correctness — the article treats these as separate overall-performance dimensions
- Input is a cluster-level annotation mapping rather than per-cell predicted labels, so per-cell accuracy and F1 cannot be computed directly

## Inputs

- Per-cell predicted cell-type labels from one or more methods (e.g. CellAssign probabilistic assignments, cluster-annotated labels from Seurat/SC3/PhenoGraph, SCINA marker-based calls)
- Ground-truth manually annotated cell-type labels for the same cells
- Optional: the train/test split used for evaluation (e.g. caret createDataPartition or createFolds output)

## Outputs

- Per-method accuracy (numeric, correct predictions / total labeled cells)
- Per-method mean F1-score averaged over cell types
- Ranked comparison table, e.g. cellassign_comparison_results.csv, with one row per evaluated method and numeric accuracy and F1-score columns

## How to apply

After each method (e.g. CellAssign, Seurat, SC3, PhenoGraph, densityCut, dynamicTreeCut, scmap-cluster, SCINA) has produced per-cell predicted labels on a labeled test set, compute accuracy as the number of correctly predicted cells divided by the total number of labeled cells, excluding unlabeled cells from the denominator. Compute F1 per cell type as the harmonic mean of precision and recall, then average F1 over cell types to obtain the mean per-cell-type F1-score. Use consistent test partitions across methods — the article's evaluation protocol uses caret's createDataPartition/createFolds in a five-fold cross-validation scheme (four subsets as training, the remaining subset as testing, repeated over five rounds). Rank methods by both metrics and write the comparison table (one row per method with numeric accuracy and F1-score columns), e.g. cellassign_comparison_results.csv. Validation mirrors the article's overall performance framing, which combines accuracy, F1-score, speed, unlabeling rate, sensitivity and specificity.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper that runs the automatic cell-type identification methods (eagersupervised, lazysupervised, markersupervised) whose predictions are then scored with accuracy and F1-score) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **caret** (Provides createDataPartition and createFolds to build the consistent train/test partitions over which accuracy and F1-score are computed in the five-fold cross-validation evaluation)
- **Seurat** (Used in the evaluation pipeline to find cell type specific markers; its clustering-based predictions are one of the scored baselines)

## Examples

```
eagersupervised(train,test,label_train,method='Seurat')
```

## Evaluation signals

- The results table contains one row per evaluated method with numeric accuracy and F1-score columns
- Accuracy equals correctly predicted cells divided by total labeled cells, with unlabeled cells excluded from the denominator
- F1-score is computed as the harmonic mean of precision and recall and then averaged over cell types
- The resulting ranking is consistent with the relative ordering of methods reported in the study (e.g. CellAssign above unsupervised, eager-learning and SCINA baselines on simulated data)
- All methods are evaluated on the same cross-validation folds generated via caret createDataPartition/createFolds

## Limitations

- Accuracy alone can be misleading when unlabeled predictions exist; the article separately tracks unlabeling rate as part of overall performance
- Methods with different labeling behaviors (probabilistic assignment vs. cluster annotation) must be reconciled to per-cell labels before scores are comparable
- The evaluation depends on the train/test split; the article's protocol uses five-fold cross-validation rather than a single holdout

## Evidence

- [methods] Overall performance of automatic methods consists of accuracy, F1-score, speed, unlabeling rate, sensitivity and specificity: "Overall performance of automatic methods consists of accuracy, F1-score, speed, unlabeling rate, sensitivity and specificity"
- [methods] Five-fold cross-validation splits: "Four subsets were used as training datasets, and the remaining subset was used as the testing dataset. After five rounds of cross-validation"
- [abstract] Thirty-two methods evaluated on accuracy and F1: "we discuss and evaluate thirty-two published automatic methods for scRNA-seq data analysis in terms of their prediction accuracy, F1-score, unlabeling rate and running time."
- [readme] SCINA as a marker-supervised method: [evidence span withheld — anchor status: not_found]
