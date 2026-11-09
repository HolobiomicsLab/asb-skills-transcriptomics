---
name: cross-dataset-classifier-comparison
description: Use when a new cell-annotation or classification method must be compared
  against established algorithms on the same ground-truth-labeled scRNA-seq test sets
  with ranked, per-dataset metrics.
when_to_use_negative:
- The goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial)
  rather than the binary cancer vs non-cancer task this comparison covers
- Input is raw counts without library-depth normalization (e.g., TPM/CPM), since PreCanCell_data()
  requires a normalized expression matrix
- The test data were used for feature selection of the candidate method, which would
  overestimate training-set performance
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: PreCanCell
  role: 'Candidate ensemble classifier: DEG-based k-NN (k=5) with majority vote over
    five training datasets'
  repo: https://github.com/WangX-Lab/PreCanCell
- name: CHETAH
  role: Comparator run via its R package with default parameters except thresh = 0
- name: SciBet
  role: Comparator run via its R package with default parameters
- name: SCINA
  role: Comparator run via its R package with default parameters except allow_unknown
    = 0
- name: scmap-cell
  role: Comparator run via its R package with default parameters
- name: scmap-cluster
  role: Comparator run via its R package with default parameters
- name: SingleR
  role: Comparator run via its R package with default parameters
- name: ikarus
  role: Comparator run via its Python package with default parameters
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:e7e2b73d56f2afcacd1d4d8f1ff8aa0609781e5361a3e8c0982b2156662c938c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - CHETAH
  - PreCanCell
  - SCINA
  - SciBet
  - SingleR
  - ikarus
  - scmap-cell
  - scmap-cluster
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# cross-dataset-classifier-comparison

## Summary

Benchmark a supervised single-cell classifier (PreCanCell) head-to-head against seven comparators (CHETAH, SciBet, SCINA, scmap-cell, scmap-cluster, SingleR, ikarus) on identical test sets using accuracy, sensitivity, specificity, balanced accuracy, and AUROC.

## When to use

Use when a new cell-annotation or classification method must be compared against established algorithms on the same ground-truth-labeled scRNA-seq test sets with ranked, per-dataset metrics.

## When NOT to use

- The goal is annotating non-cancer cell subpopulations (immune, stromal, epithelial) rather than the binary cancer vs non-cancer task this comparison covers
- Input is raw counts without library-depth normalization (e.g., TPM/CPM), since PreCanCell_data() requires a normalized expression matrix
- The test data were used for feature selection of the candidate method, which would overestimate training-set performance

## Inputs

- Normalized scRNA-seq expression matrices for 11 test datasets (rownames = genes, colnames = cells)
- Ground-truth malignant/non-malignant cell annotations for each test set
- Training datasets T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC for the candidate method
- DEG feature set (73 TMGs and 186 NMGs)

## Outputs

- Per-dataset metric table: accuracy, sensitivity, specificity, balanced accuracy, AUROC for each of 8 algorithms on each of 11 test sets
- Accuracy ranking of the eight algorithms across test sets

## How to apply

Load the normalized scRNA-seq expression matrices for the test datasets (e.g., P2_BCC, P3_HCC, P7_GBM) together with malignant/non-malignant ground-truth annotations. Run the comparators with their native packages using default parameters, except 'allow_unknown = 0' in SCINA and 'thresh = 0' in CHETAH, and run ikarus via its Python package. Run the candidate method (PreCanCell) using the five training datasets (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC), the DEG features, and k-NN (k = 5) with majority vote. Compute accuracy, sensitivity, specificity, balanced accuracy, and AUROC for each algorithm against the ground-truth labels, aggregate into a single comparison table across all test sets, and rank algorithms by accuracy. Validate that the table contains rows for all eight algorithms across all 11 test sets with AUROC, accuracy, and balanced accuracy columns populated.

## Related tools

- **PreCanCell** (Candidate ensemble classifier: DEG-based k-NN (k=5) with majority vote over five training datasets) — https://github.com/WangX-Lab/PreCanCell
- **CHETAH** (Comparator run via its R package with default parameters except thresh = 0)
- **SciBet** (Comparator run via its R package with default parameters)
- **SCINA** (Comparator run via its R package with default parameters except allow_unknown = 0)
- **scmap-cell** (Comparator run via its R package with default parameters)
- **scmap-cluster** (Comparator run via its R package with default parameters)
- **SingleR** (Comparator run via its R package with default parameters)
- **ikarus** (Comparator run via its Python package with default parameters)

## Examples

```
testdata <- PreCanCell_data(input); results <- PreCanCell_classifier(testdata, cores = 2)
```

## Evaluation signals

- Comparison table contains rows for all eight algorithms across all 11 test sets with AUROC, accuracy, and balanced accuracy columns populated
- Candidate method ranks first or second in accuracy on the test sets (highest in three, second highest in three in the reported benchmark)
- AUROC and balanced accuracy values fall above 0.8 for PreCanCell in tested datasets, consistent with reported performance
- Per-cell predictions resolve to binary labels (cancer/non_cancer) so confusion-matrix-derived metrics can be computed against ground truth
- Comparator settings match the protocol: allow_unknown = 0 for SCINA and thresh = 0 for CHETAH

## Limitations

- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection
- No version numbers are stated for PreCanCell or for the compared algorithms, limiting reproducibility
- No accession IDs or repository links are provided for some named train/test datasets (e.g., P1_BC, T5_BC, P4_SyS, P5_SyS, P7_GBM, P8_PDAC, P10_GBM)

## Evidence

- [abstract] Comparator default settings: "seven other algorithms, including CHETAH, SciBet, SCINA, scmap-cell, scmap-cluster, SingleR, and ikarus"
- [abstract] Comparison purpose: "Compared to these algorithms, PreCanCell displays the advantages of higher accuracy and simpler implementation"
- [intro] Evaluation metrics: "We compared PreCanCell with seven other algorithms in several predictive performance, including accuracy, sensitivity, specificality, balanced accuracy (the average of sensitivity and specificity)"
- [intro] PreCanCell classification scheme: "With each of the five datasets as the training set and the DEGs as the features, a single cell is classified as malignant or non-malignant by *k*-NN (*k* = 5)"
- [abstract] Performance result: "In all these datasets, PreCanCell achieved above 0.8 accuracy, sensitivity, specificity, balanced accuracy and AUROC"
- [readme] README usage: "The function `PreCanCell_classifier()` is used to identify malignant and non-malignant cells from single-cell transcriptomes, containing 2 parameters: testdata and cores."
