---
name: classification-metric-evaluation
description: Use when a binary cell-level classifier (e.g. PreCanCell k-NN predictions,
  or CHETAH/SciBet/SCINA/scmap/SingleR/ikarus outputs) must be scored against ground-truth
  malignant/non-malignant labels using accuracy, sensitivity, specificity, balanced
  accuracy and AUROC.
when_to_use_negative:
- The task is annotating non-cancer cell subpopulations (immune, stromal, epithelial)
  — PreCanCell and this binary cancer/non-cancer evaluation do not cover that
- The classifier output is a continuous score without a binary cancer/non_cancer call
  and no ground-truth labels exist to compute accuracy, sensitivity, specificity or
  AUROC
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: PreCanCell
  role: Ensemble k-NN (k = 5) classifier whose per-cell predictions are scored with
    the metric set
  repo: https://github.com/WangX-Lab/PreCanCell
- name: CHETAH
  role: Comparator algorithm run with thresh = 0 and evaluated with the same metrics
- name: SingleR
  role: Comparator algorithm evaluated with the same metrics across the 11 test sets
- name: ikarus
  role: Comparator algorithm (Python package) evaluated with the same metrics
- name: R
  role: Language in which metrics are computed and metric tables (cv_metrics_training_sets.csv,
    evaluation_metrics.csv) are written
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:c5afdd6f59b47843f44607fe924b96eb24e8023a69abbd90ef84647bba04cb36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - CHETAH
  - PreCanCell
  - R
  - SCINA
  - SciBet
  - SingleR
  - doParallel
  - fastknn
  - foreach
  - ikarus
  - scmap-cluster
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# classification-metric-evaluation

## Summary

Compute accuracy, sensitivity, specificity, balanced accuracy (average of sensitivity and specificity) and AUROC to evaluate binary malignant vs non-malignant cell classifiers such as PreCanCell, as done across the 19 tested single-cell datasets in the PreCanCell paper.

## When to use

Use when a binary cell-level classifier (e.g. PreCanCell k-NN predictions, or CHETAH/SciBet/SCINA/scmap/SingleR/ikarus outputs) must be scored against ground-truth malignant/non-malignant labels using accuracy, sensitivity, specificity, balanced accuracy and AUROC.

## When NOT to use

- The task is annotating non-cancer cell subpopulations (immune, stromal, epithelial) — PreCanCell and this binary cancer/non-cancer evaluation do not cover that
- The classifier output is a continuous score without a binary cancer/non_cancer call and no ground-truth labels exist to compute accuracy, sensitivity, specificity or AUROC

## Inputs

- Per-cell predicted labels (cancer / non_cancer) from PreCanCell_classifier() or comparator algorithms
- Ground-truth malignant/non-malignant annotations for the test or cross-validation held-out cells
- Normalized scRNA-seq expression matrices (PreCanCell_data() output, [0,1]-scaled DEG features)

## Outputs

- Per-dataset metric table (accuracy, sensitivity, specificity, balanced accuracy, AUROC), e.g. cv_metrics_training_sets.csv
- Benchmark comparison table ranking PreCanCell vs CHETAH, SciBet, SCINA, scmap-cell, scmap-cluster, SingleR and ikarus across the 11 test sets
- Error rates: false positive rate on GSE126030 (0.0089) and false negative rate on GSE157220 (0.0041)

## How to apply

Take the per-cell predicted labels (cancer/non_cancer) from PreCanCell_classifier() or a comparator algorithm and the ground-truth annotations, then compute accuracy, sensitivity, specificity, balanced accuracy (the average of sensitivity and specificity) and AUROC for each test set. Aggregate the per-dataset metrics into a single table with one row per dataset and one column per metric (e.g. cv_metrics_training_sets.csv or evaluation_metrics.csv). For the benchmark, run all eight algorithms (PreCanCell plus CHETAH, SciBet, SCINA, scmap-cell, scmap-cluster, SingleR, ikarus) on the same 11 test sets with default parameters (SCINA allow_unknown = 0, CHETAH thresh = 0) and rank by accuracy. Validate acceptance criteria such as all metrics > 0.90 under 10-fold cross-validation on the five training datasets, FPR 0.0089 on GSE126030, and 0.995 accuracy/AUROC on the synthetic merged dataset.

## Related tools

- **PreCanCell** (Ensemble k-NN (k = 5) classifier whose per-cell predictions are scored with the metric set) — https://github.com/WangX-Lab/PreCanCell
- **CHETAH** (Comparator algorithm run with thresh = 0 and evaluated with the same metrics)
- **SingleR** (Comparator algorithm evaluated with the same metrics across the 11 test sets)
- **ikarus** (Comparator algorithm (Python package) evaluated with the same metrics)
- **R** (Language in which metrics are computed and metric tables (cv_metrics_training_sets.csv, evaluation_metrics.csv) are written)

## Examples

```
results <- PreCanCell_classifier(testdata, cores = 2)
```

## Evaluation signals

- All five metrics (accuracy, sensitivity, specificity, balanced accuracy, AUROC) exceed 0.90 in each of the five training datasets under 10-fold cross-validation
- On the synthetic merged dataset, metrics match reported values: accuracy 0.995, sensitivity 0.998, specificity 0.993, balanced accuracy 0.995, AUROC 0.995
- FPR on GSE126030 equals 0.0089 and FNR on GSE157220 equals 0.0041
- Benchmark table contains rows for all eight algorithms across all 11 test sets with AUROC, accuracy and balanced accuracy populated
- PreCanCell ranks first or second in accuracy (highest in three datasets, second highest in three) among the seven comparators

## Limitations

- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection
- No version numbers are stated for the PreCanCell R package or the comparator algorithms, hindering exact reproduction
- Metrics above 0.8 hold for the specific 19 tested single-cell datasets; transfer to other data types is unverified

## Evidence

- [intro] We compared PreCanCell with seven other algorithms in several predictive performance, including accuracy, sensitivity, specificality, balanced accuracy (the average of sensitivity and specificity): "We compared PreCanCell with seven other algorithms in several predictive performance, including accuracy, sensitivity, specificality, balanced accuracy (the average of sensitivity and specificity)"
- [abstract] In all these datasets, PreCanCell achieved above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC: "In all these datasets, PreCanCell achieved above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC"
- [abstract] Compared to these algorithms, PreCanCell displays the advantages of higher accuracy and simpler implementation: "Compared to these algorithms, PreCanCell displays the advantages of higher accuracy and simpler implementation"
- [methods] running `PreCanCell_classifier()` function to predict malignant and non-malignant cells: "running `PreCanCell_classifier()` function to predict malignant and non-malignant cells"
- [readme] results <- PreCanCell_classifier(testdata, cores = 2): "results <- PreCanCell_classifier(testdata, cores = 2)"
