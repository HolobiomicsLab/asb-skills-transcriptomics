---
name: majority-vote-ensemble-aggregation
description: Use when a single cell must be labeled malignant vs non-malignant from
  scRNA-seq expression and multiple base classifiers (one per training dataset) have
  produced per-cell predictions that need to be combined into a final consensus label.
when_to_use_negative:
- The goal is annotating subpopulations of non-cancer cells (immune, stromal, epithelial)
  — PreCanCell is limited to cancer vs non-cancer annotation only
- Only a single training dataset or single base classifier is available, so no vote
  aggregation is possible
- Input expression values have not been normalized (e.g. raw counts); PreCanCell_data()
  expects any library-depth-normalized matrix (TPM, CPM)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: PreCanCell
  role: R package whose PreCanCell_classifier() function trains the five k-NN base
    classifiers and aggregates their results by majority vote into freq_cancer, freq_non_cancer,
    and pred_label
  repo: https://github.com/WangX-Lab/PreCanCell
- name: fastknn
  role: k-NN backend used to train each base classifier with k = 5 on the DEG features
  repo: https://github.com/davpinto/fastknn
- name: doParallel
  role: Parallel backend for executing the five base classifiers across multiple workers
    controlled by the cores argument
- name: foreach
  role: Loop parallelization machinery used to run the five base k-NN classifiers
    concurrently
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:40739a92b9b2b392b262a0c22f7d8cee712fbf20dee00f249308cefcea5447ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - PreCanCell
  - R
  - doParallel
  - fastknn
  - foreach
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# majority-vote-ensemble-aggregation

## Summary

Aggregates the outputs of five k-NN (k=5) base classifiers — one trained per cancer scRNA-seq training dataset (RCC, HNSCC, melanoma, LUAD, BC) on the common DEG features — into a single malignant/non-malignant call per cell via majority vote, as implemented in PreCanCell.

## When to use

Use when a single cell must be labeled malignant vs non-malignant from scRNA-seq expression and multiple base classifiers (one per training dataset) have produced per-cell predictions that need to be combined into a final consensus label.

## When NOT to use

- The goal is annotating subpopulations of non-cancer cells (immune, stromal, epithelial) — PreCanCell is limited to cancer vs non-cancer annotation only
- Only a single training dataset or single base classifier is available, so no vote aggregation is possible
- Input expression values have not been normalized (e.g. raw counts); PreCanCell_data() expects any library-depth-normalized matrix (TPM, CPM)

## Inputs

- Preprocessed expression matrix from PreCanCell_data() (matched DEG features, min-max scaled to [0,1], genes as rows, cells as columns)
- Five pre-trained k-NN (k=5) base classifiers, one per training dataset (T1_RCC, T2_HNSCC, T3_Melanoma, T4_LUAD, T5_BC)
- Number of cores for parallel execution (cores argument)

## Outputs

- Results table with columns Sample, freq_cancer, freq_non_cancer, pred_label for every cell (e.g. cell_predictions.csv)
- Per-cell consensus label: 'cancer' or 'non_cancer' assigned by majority vote of the five k-NN results

## How to apply

Train one k-NN (k=5) classifier on each of the five cancer training datasets using the 73 TMGs and 186 NMGs as features, after PreCanCell_data() preprocessing (matched genes selected, expression min-max scaled to [0,1]). Run the five base classifiers — optionally in parallel via the cores argument of PreCanCell_classifier() using R's parallel/foreach/doParallel machinery. For each cell, count the base classifiers predicting cancer and non-cancer, compute freq_cancer and freq_non_cancer as the ratio of each count to 5, and assign the final pred_label by majority vote. Verify each row satisfies freq_cancer + freq_non_cancer = 1 and that freq values are multiples of 1/5 (0, 0.2, 0.4, 0.6, 0.8, 1).

## Related tools

- **PreCanCell** (R package whose PreCanCell_classifier() function trains the five k-NN base classifiers and aggregates their results by majority vote into freq_cancer, freq_non_cancer, and pred_label) — https://github.com/WangX-Lab/PreCanCell
- **fastknn** (k-NN backend used to train each base classifier with k = 5 on the DEG features) — https://github.com/davpinto/fastknn
- **doParallel** (Parallel backend for executing the five base classifiers across multiple workers controlled by the cores argument)
- **foreach** (Loop parallelization machinery used to run the five base k-NN classifiers concurrently)

## Examples

```
library(PreCanCell); testdata <- PreCanCell_data(input); results <- PreCanCell_classifier(testdata, cores = 2)
```

## Evaluation signals

- Results table contains exactly the four columns Sample, freq_cancer, freq_non_cancer, pred_label
- For every row, freq_cancer + freq_non_cancer = 1 (each is a ratio to 5)
- All freq values are multiples of 0.2 (0, 0.2, 0.4, 0.6, 0.8, 1), reflecting five base classifiers
- Every pred_label is one of 'cancer' / 'non_cancer' and equals the majority of the five base votes
- On the synthetic GSE157220 + GSE126030 dataset the aggregated predictions yield FPR 0.0089, FNR 0.0041, and accuracy/sensitivity/specificity/balanced accuracy/AUROC of 0.995/0.998/0.993/0.995/0.995

## Limitations

- Training-set predictive performance may be overestimated because the predicted sets were involved in feature selection
- Ties are structurally avoided only because there are an odd number (5) of base classifiers; aggregated labels are limited to binary cancer/non-cancer calls
- No version number is stated for the PreCanCell R package, complicating reproducibility of the ensemble behavior

## Evidence

- [abstract] Majority vote definition: "a single cell is classified as malignant or non-malignant by k-NN (k = 5)"
- [intro] Final aggregation rule: "Finally, the single cell is classified by the majority vote of the five *k*-NN classification results."
- [readme] freq ratio construction: ""testdata" is a output matrix of the function `PreCanCell_data()`.  + "cores" is the number of threads."
- [discussion] Training-set overestimation caveat: "the reported predictive performance in the training set could be overestimated due to the predicted sets being involved in feature selection"
