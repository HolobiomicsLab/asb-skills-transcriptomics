---
name: marker-gene-based-classification
description: Use when benchmarking cell-type label prediction on multi-batch scRNA-seq
  data, specifically to compare a marker/reference-database classifier such as CellTypist
  against zero-shot predictors using accuracy and macro-F1.
when_to_use_negative:
- The task is gene network inference or denoising rather than cell-type label prediction
  — scPRINT's primary purpose is GN inference and classification is only a zero-shot
  side evaluation
- No tissue-matched CellTypist reference database exists, so the marker-based baseline
  would cover only a small fraction of test-set cell types (e.g. a ceiling of 4 of
  14 database cell types covering ~70% of cells) and the comparison is unfair
- The dataset is spatial omics or non-single-cell — excluded from this benchmarking
  scope
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellTypist
  role: Marker/reference-database classifier run with default parameters ('Adult_Human_PancreaticIslet'
    database) as the supervised baseline in the pancreas benchmark
- name: scPRINT
  role: Zero-shot cell label predictor (no fine-tuning, 4000-gene context, random
    gene sampling) compared against the marker-based classifier
  repo: https://github.com/cantinilab/scPRINT
- name: scikit-learn
  role: Supplies classifiers and metrics for the openproblems baselines and accuracy/macro-F1
    computation (e.g. RidgeClassifier with L2 penalty 1 elsewhere in the paper's benchmarks)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:b3cfb306fc34cfa30b30aeeb085e0f4f4b14bdfdb5f7a10b037accba1ee6ecb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - CellTypist
  - Python
  - scPRINT
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# marker-gene-based-classification

## Summary

Runs marker-gene-referenced cell-type classification (CellTypist with a curated marker/reference database) as a supervised baseline against zero-shot foundation-model label predictors such as scPRINT, on batch-split scRNA-seq benchmarks like the open-problems human pancreas dataset.

## When to use

Use when benchmarking cell-type label prediction on multi-batch scRNA-seq data, specifically to compare a marker/reference-database classifier such as CellTypist against zero-shot predictors using accuracy and macro-F1.

## When NOT to use

- The task is gene network inference or denoising rather than cell-type label prediction — scPRINT's primary purpose is GN inference and classification is only a zero-shot side evaluation
- No tissue-matched CellTypist reference database exists, so the marker-based baseline would cover only a small fraction of test-set cell types (e.g. a ceiling of 4 of 14 database cell types covering ~70% of cells) and the comparison is unfair
- The dataset is spatial omics or non-single-cell — excluded from this benchmarking scope

## Inputs

- Multi-batch scRNA-seq count matrix (train-test split by batch) with cell-type labels, e.g. the open-problems human pancreas dataset
- CellTypist reference database matching the tissue (e.g. 'Adult_Human_PancreaticIslet', 14 cell types)
- Count-normalized, log1p-transformed expression over the full gene set

## Outputs

- Per-cell cell-type label predictions for each method (CellTypist, scPRINT zero-shot, openproblems logistic regression, xgboost)
- classification_benchmark_results.csv with accuracy and macro-F1 per method and per label granularity (fine-grained vs ontology-aggregated)

## How to apply

Load a multi-batch scRNA-seq dataset (e.g. open-problems human pancreas) and split train-test by batch following the openproblems benchmark preprocessing. Normalize counts and log1p-transform over the full gene set, then run CellTypist with default parameters using a matched reference database such as 'Adult_Human_PancreaticIslet' (14 cell types). Run zero-shot comparators (scPRINT at a 4000-gene context with random gene sampling, no fine-tuning) and openproblems classifiers (logistic regression and xgboost on 25 PCs) on the same split. Compute accuracy and macro-F1 per method, then re-evaluate after rolling fine-grained labels up to coarser ontology aggregations (e.g. 'endocrine pancreatic cell'), compiling results into classification_benchmark_results.csv.

## Related tools

- **CellTypist** (Marker/reference-database classifier run with default parameters ('Adult_Human_PancreaticIslet' database) as the supervised baseline in the pancreas benchmark)
- **scPRINT** (Zero-shot cell label predictor (no fine-tuning, 4000-gene context, random gene sampling) compared against the marker-based classifier) — https://github.com/cantinilab/scPRINT
- **scikit-learn** (Supplies classifiers and metrics for the openproblems baselines and accuracy/macro-F1 computation (e.g. RidgeClassifier with L2 penalty 1 elsewhere in the paper's benchmarks))

## Examples

```
from sklearn.metrics import accuracy_score, f1_score; acc = accuracy_score(y_true, y_pred); macro_f1 = f1_score(y_true, y_pred, average='macro')
```

## Evaluation signals

- classification_benchmark_results.csv contains accuracy and macro-F1 for every method at both fine-grained and ontology-aggregated ('endocrine pancreatic cell') label granularity
- CellTypist accuracy reflects its database-coverage ceiling: only 4 of 14 'Adult_Human_PancreaticIslet' cell types cover ~70% of cells, so its accuracy sits below scPRINT's zero-shot 62%
- scPRINT achieves its accuracy without training on the test batch (train-test split strictly by batch per openproblems preprocessing)
- Logistic regression and xgboost baselines are fitted on 25 PCs of count-normalized, log1p-transformed data, matching the openproblems setup

## Limitations

- CellTypist performance is bounded by reference-database coverage — in the pancreas benchmark only 4 of 14 database cell types covered ~70% of cells, capping its achievable accuracy
- No perfect ground truths exist for label granularity: results change substantially when fine-grained labels are rolled up to coarser ontology terms
- Zero-shot prediction depends on the pretraining label ontology; scPRINT uses a hierarchical classification loss during multi-task pretraining to support label prediction

## Evidence

- [abstract] scPRINT shows competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
- [methods] Hierarchical classification loss supports label prediction in scPRINT pretraining: "Due to the hierarchical structure of the prediction, we also created a hierarchical loss."
- [intro] Zero-shot abilities arise from learning a cell model: "This suggests that by learning a cell model, scPRINT gains zero-shot abilities in many tasks of cellular biology."
