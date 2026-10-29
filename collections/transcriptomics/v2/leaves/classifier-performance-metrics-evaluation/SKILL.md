---
name: classifier-performance-metrics-evaluation
description: Use when you have per-cell predicted labels (cancer/non_cancer) plus
  publication ground-truth annotations for one or more scRNA-seq test sets and need
  to reproduce or compare the paper's five reported metrics.
when_to_use_negative:
- The task is annotating non-cancer cell subpopulations (immune, stromal, epithelial)
  — PreCanCell is limited to binary cancer vs non-cancer calls, so subpopulation-level
  metrics are out of scope
- Evaluating on the five training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD,
  T5_BC) as unbiased performance — the paper warns training-set performance may be
  overestimated because those sets were involved in feature selection
- Input is already a metrics table or the ground-truth annotations for the test cells
  are unavailable — metrics cannot be computed without reference labels
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: PreCanCell
  role: R package producing the per-cell pred_label and freq_cancer/freq_non_cancer
    outputs that the metrics are computed against
  repo: https://github.com/WangX-Lab/PreCanCell
- name: R
  role: Environment for computing accuracy, sensitivity, specificity, balanced accuracy,
    and AUROC and writing test_set_metrics.csv
- name: fastknn
  role: Dependency providing the k-NN (k=5) classifier whose five per-training-set
    votes PreCanCell aggregates before metric computation
  repo: https://github.com/davpinto/fastknn
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:b81d94bfcfcb81179888b262b82dc4f82de9c190cda034742fabe69c55e4f34b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - PreCanCell
  - R
  - fastknn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# classifier-performance-metrics-evaluation

## Summary

Evaluate a binary cell classifier (PreCanCell) by computing accuracy, sensitivity, specificity, balanced accuracy (mean of sensitivity and specificity), and AUROC per dataset, using the original publication's cell annotations as ground truth.

## When to use

Use when you have per-cell predicted labels (cancer/non_cancer) plus publication ground-truth annotations for one or more scRNA-seq test sets and need to reproduce or compare the paper's five reported metrics.

## When NOT to use

- The task is annotating non-cancer cell subpopulations (immune, stromal, epithelial) — PreCanCell is limited to binary cancer vs non-cancer calls, so subpopulation-level metrics are out of scope
- Evaluating on the five training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC) as unbiased performance — the paper warns training-set performance may be overestimated because those sets were involved in feature selection
- Input is already a metrics table or the ground-truth annotations for the test cells are unavailable — metrics cannot be computed without reference labels

## Inputs

- Per-cell predicted labels from PreCanCell_classifier() (Sample, freq_cancer, freq_non_cancer, pred_label)
- Ground-truth malignant/non-malignant cell annotations from the original publications of the 11 test sets (P1_BC through P11_Melanoma)
- Preprocessed test expression matrices intersected with the 259-gene DEG panel (73 TMGs + 186 NMGs), min-max scaled to [0,1]

## Outputs

- Per-dataset table of accuracy, sensitivity, specificity, balanced accuracy, and AUROC (e.g. test_set_metrics.csv)
- Per-cell frequency scores (freq_cancer) used as AUROC inputs

## How to apply

For each test set (e.g. P1_BC, P2_BCC, P3_HCC, P4_SyS, P5_SyS, P6_HCC, P7_GBM, P8_PDAC, P9_mCRPC, P10_GBM, P11_Melanoma), align the classifier output (pred_label, plus freq_cancer/freq_non_cancer frequencies for AUROC) with the original publication's malignant/non-malignant cell annotations as ground truth. Compute accuracy as the fraction of correctly labeled cells, sensitivity on malignant cells, specificity on non-malignant cells, and balanced accuracy as the mean of sensitivity and specificity, per the paper's definition. Compute AUROC from the continuous freq_cancer scores rather than the hard labels. Assemble per-dataset metrics into a single table (e.g. test_set_metrics.csv) and judge correctness against the paper's reported ranges: accuracy > 0.90 in at least 7 of 11 sets, balanced accuracy > 0.80 in all 11, AUROC > 0.80 in at least 8 sets, sensitivity > 0.80 in at least 9 sets, and specificity > 0.80 in all 11.

## Related tools

- **PreCanCell** (R package producing the per-cell pred_label and freq_cancer/freq_non_cancer outputs that the metrics are computed against) — https://github.com/WangX-Lab/PreCanCell
- **R** (Environment for computing accuracy, sensitivity, specificity, balanced accuracy, and AUROC and writing test_set_metrics.csv)
- **fastknn** (Dependency providing the k-NN (k=5) classifier whose five per-training-set votes PreCanCell aggregates before metric computation) — https://github.com/davpinto/fastknn

## Examples

```
results <- PreCanCell_classifier(testdata, cores = 2); # then compute accuracy, sensitivity, specificity, balanced accuracy, AUROC vs publication annotations
```

## Evaluation signals

- Accuracy exceeds 0.90 in at least 7 of the 11 independent test sets
- Balanced accuracy (mean of sensitivity and specificity) exceeds 0.80 in all 11 test sets
- AUROC exceeds 0.80 in at least 8 test sets (computed from freq_cancer scores)
- Sensitivity exceeds 0.80 in at least 9 test sets; specificity exceeds 0.80 in all 11
- All five metrics exceed 0.80 across the tested datasets, consistent with the paper's reported performance

## Limitations

- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection
- PreCanCell annotates only cancer vs non-cancer cells and does not identify subpopulations of non-cancer cells (immune, stromal, epithelial)
- No accession IDs or repository links are provided for several named test sets (P4_SyS, P5_SyS, P7_GBM, P8_PDAC, P10_GBM), which can hinder ground-truth retrieval

## Evidence

- [abstract] In all these datasets, PreCanCell achieved above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC: "In all these datasets, PreCanCell achieved above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC"
- [results] Applying PreCanCell to the 11 independent test sets yielded prediction accuracy above 0.90 in seven datasets: [evidence span withheld — anchor status: not_found]
- [intro] We compared PreCanCell with seven other algorithms in several predictive performance: "We compared PreCanCell with seven other algorithms in several predictive performance, including accuracy, sensitivity, specificality, balanced accuracy (the average of sensitivity and specificity)"
- [discussion] the reported predictive performance in the training set could be overestimated: "the reported predictive performance in the training set could be overestimated due to the predicted sets being involved in feature selection"
- [readme] freq_cancer | freq_non_cancer | pred_label: [evidence span withheld — anchor status: external_doc]
