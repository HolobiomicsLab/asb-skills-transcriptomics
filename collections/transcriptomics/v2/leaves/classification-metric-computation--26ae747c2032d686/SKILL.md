---
name: classification-metric-computation
description: Use when cell-type predictions from identification methods (e.g. Seurat,
  SingleR, SciBet, CELLBLAST) must be scored against manual ground-truth labels to
  rank methods by accuracy, F1-score and unlabeling rate in a cross-validation or
  cross-platform evaluation.
when_to_use_negative:
- The dataset has no manually annotated cell-type labels, so accuracy and F1-score
  against ground truth cannot be computed
- The goal is method runtime or memory profiling rather than prediction-quality metrics
  (speed is reported separately in the paper's overall performance score)
- Predictions are unsupervised cluster assignments without per-cell type calls, so
  unlabeling rate and per-cell accuracy are undefined
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: AutomaticCellTypeIdentification
  role: R wrapper that runs the evaluated methods (eagersupervised, lazysupervised,
    markersupervised interfaces) to produce the predictions whose accuracy, F1-score
    and unlabeling rate are computed
  repo: https://github.com/xiebb123456/AutomaticCellTypeIdentification
- name: caret
  role: Provides createFolds/createDataPartition to build the five cell-type-stratified
    cross-validation splits over which the metrics are averaged
- name: Seurat
  role: Used to find cell-type-specific markers (FindAllMarkers, top 5 and 15 genes
    per type) that feed marker-learning methods whose predictions are then scored
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.10.027
    title: Automatic cell type identification methods for single-cell RNA sequencing
  source_content_hash: sha256:b7fd2636424fbe4441d3d5a6e034d008612b803262f675d8c6eb59e3b5a52d22
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
  - Seurat
  - SingleR
  - caret
  - clustifyr
  - scHPL
  - scID
  - scLearn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 21
---

# classification-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes prediction accuracy (on labeled cells only), mean per-cell-type F1-score, and unlabeling rate to quantitatively evaluate automatic cell-type identification methods on scRNA-seq data against manually annotated labels, as done in this benchmark of thirty-two published methods.

## When to use

Use when cell-type predictions from identification methods (e.g. Seurat, SingleR, SciBet, CELLBLAST) must be scored against manual ground-truth labels to rank methods by accuracy, F1-score and unlabeling rate in a cross-validation or cross-platform evaluation.

## When NOT to use

- The dataset has no manually annotated cell-type labels, so accuracy and F1-score against ground truth cannot be computed
- The goal is method runtime or memory profiling rather than prediction-quality metrics (speed is reported separately in the paper's overall performance score)
- Predictions are unsupervised cluster assignments without per-cell type calls, so unlabeling rate and per-cell accuracy are undefined

## Inputs

- Per-cell predicted cell-type labels from each identification method (output of eagersupervised/lazysupervised/markersupervised calls)
- Manually annotated cell-type labels for the same cells (Tabula Muris, PBMC SCP424, GSE131907 lung, GSE116470 brain)
- Cell-type-stratified fold assignments from caret::createFolds (five folds, five cross-validation rounds)
- Preprocessed count matrices (genes not expressed in any cell removed, cells <200 expressed genes, >2 SD of mean genes, >20% mitochondrial expression, or lacking labels deleted)

## Outputs

- Method-by-tissue (or method-by-platform) matrix of mean accuracy, mean F1-score and mean unlabeling rate, averaged over five cross-validation rounds
- Aggregated metric tables such as cross_platform_performance.csv and pbmc_cross_platform_results.csv
- Heatmap (Fig. 2B layout) of metric values with one row per method and one column per retained tissue
- Ranked method lists per category (eager, lazy, marker learning)

## How to apply

Run each method's predictions through the AutomaticCellTypeIdentification R package (via its eagersupervised, lazysupervised or markersupervised interfaces) on cell-type-stratified train/test splits, e.g. five-fold cross-validation built with the caret package's createFolds function, averaging metrics over five rounds. For each method and dataset, compare predictions to the manually annotated labels and compute accuracy on labeled cells only, the F1-score as the mean per-cell-type harmonic mean of precision and recall, and the unlabeling rate (fraction of cells left unlabeled by the method). Assemble the mean accuracy, F1-score and unlabeling-rate values into a method-by-tissue (or method-by-platform) matrix for heatmap plotting (Fig. 2B layout) and write aggregated tables such as cross_platform_performance.csv or pbmc_cross_platform_results.csv. Finally rank methods within their categories (eager, lazy, marker learning) to expose top performers. The rationale is that the paper's overall performance score is defined by these metrics — accuracy, F1-score, speed, unlabeling rate, sensitivity and specificity — and averaging across cross-validation rounds guards against fold-specific variance.

## Related tools

- **AutomaticCellTypeIdentification** (R wrapper that runs the evaluated methods (eagersupervised, lazysupervised, markersupervised interfaces) to produce the predictions whose accuracy, F1-score and unlabeling rate are computed) — https://github.com/xiebb123456/AutomaticCellTypeIdentification
- **caret** (Provides createFolds/createDataPartition to build the five cell-type-stratified cross-validation splits over which the metrics are averaged)
- **Seurat** (Used to find cell-type-specific markers (FindAllMarkers, top 5 and 15 genes per type) that feed marker-learning methods whose predictions are then scored)

## Examples

```
eagersupervised(train,test,label_train,method='Seurat')
```

## Evaluation signals

- All computed accuracy, F1-score and unlabeling-rate values fall within the 0–1 range reported for these metrics in the paper's self-projection evaluation
- The method-by-tissue heatmap file contains exactly one row per evaluated method and one column per retained tissue (eleven Tabula Muris tissues)
- Accuracy is computed excluding unlabeled cells, so per-method accuracy plus unlabeling rate can be cross-checked for consistency with prediction counts
- Cross-platform rankings reproduce the reported top performers: clustifyr, scLearn and scHPL for accuracy, and scHPL, SciBet and SingleR for F1-score
- Metric values are means over five rounds of five-fold cross-validation, and high accuracy with low unlabeling rate is expected across tissues except marrow

## Limitations

- Accuracy computed on labeled cells only can look inflated for methods with a high unlabeling rate, so unlabeling rate must always be reported alongside accuracy
- Most methods show high accuracy, high F1-score and low unlabeling rate across the eleven Tabula Muris tissues except marrow, where metrics degrade
- Metrics depend on the preprocessing thresholds (≥200 expressed genes, ≤2 SD of mean, ≤20% mitochondrial expression), so results are not comparable without the same filtering

## Evidence

- [abstract] Thirty-two published methods evaluated on accuracy, F1-score, unlabeling rate: [evidence span withheld — anchor status: not_found]
- [readme] Package interfaces generating predictions to score: [evidence span withheld — anchor status: external_doc]
