---
name: macro-f1-accuracy-evaluation
description: Use when comparing cell-type label predictors on a single-cell classification
  benchmark and you need comparable accuracy and macro-F1 per method across fine-grained
  and coarse ontology label levels.
when_to_use_negative:
- The task is gene network inference benchmarking rather than cell label classification
  — use the article's BenGRN/GRnnData benchmarking suite instead
- The evaluated predictor was trained or fine-tuned on the test dataset, which invalidates
  the zero-shot comparison framing of this benchmark
- The label ontology differs fundamentally from the pancreas benchmark's (e.g. tissue-agnostic
  or spatial labels), so CellTypist database ceilings and coarse-label roll-ups would
  not be meaningful
tools:
- name: scPRINT
  role: Zero-shot cell label predictor whose predictions are scored at 4000-gene context
    with random gene sampling; no fine-tuning on the test data
  repo: https://github.com/cantinilab/scPRINT
- name: CellTypist
  role: Marker-based reference classifier run with default parameters on the 'Adult_Human_PancreaticIslet'
    database (14 cell types) over normalized, log1p-transformed counts
- name: scikit-learn
  role: Provides the RidgeClassifier used for head selection in scPRINT and the accuracy/macro-F1
    metric computations for the benchmark
- name: Python
  role: Execution environment for running all classifiers, openproblems baselines,
    and metric compilation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:760c698ff33c05438fe8fd27998f8e5947b283aa6a4e8fcd17fca1cbc8346292
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/scPRINT
  tools:
  - CellTypist
  - Python
  - scPRINT
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# macro-f1-accuracy-evaluation

## Summary

A protocol for evaluating cell-type label prediction methods (e.g. zero-shot scPRINT vs CellTypist vs supervised openproblems classifiers) by computing accuracy and macro-F1 (average F1 across classes) at multiple label granularities on a batch-split scRNAseq benchmark.

## When to use

Use when comparing cell-type label predictors on a single-cell classification benchmark and you need comparable accuracy and macro-F1 per method across fine-grained and coarse ontology label levels.

## When NOT to use

- The task is gene network inference benchmarking rather than cell label classification — use the article's BenGRN/GRnnData benchmarking suite instead
- The evaluated predictor was trained or fine-tuned on the test dataset, which invalidates the zero-shot comparison framing of this benchmark
- The label ontology differs fundamentally from the pancreas benchmark's (e.g. tissue-agnostic or spatial labels), so CellTypist database ceilings and coarse-label roll-ups would not be meaningful

## Inputs

- per-method cell-type label predictions on the open-problems pancreas test batch (scPRINT zero-shot at 4000-gene context, CellTypist 'Adult_Human_PancreaticIslet', logistic regression and xgboost on 25 PCs)
- ground-truth cell-type labels for the test cells at fine-grained and coarse 'endocrine pancreatic cell' ontology levels
- count-normalized, log1p-transformed expression matrix used by the baseline classifiers

## Outputs

- classification_benchmark_results.csv with accuracy and macro-F1 for each method at each label granularity
- per-method accuracy and macro-F1 scores (zero-shot scPRINT reported at ~62% accuracy, largely outperforming CellTypist)

## How to apply

After each method has produced predictions on the held-out test batch of the open-problems human pancreas dataset, compute accuracy and macro-F1 (average F1 across classes) for every method's predictions with scikit-learn. Evaluate predictions at two granularities: the original fine-grained labels, and after rolling labels up to the coarser 'endocrine pancreatic cell' ontology aggregation, because label vocabulary size strongly affects scores (CellTypist's 'Adult_Human_PancreaticIslet' database covers only 14 cell types, 4 of which cover ~70% of cells). Compile all per-method, per-granularity metrics into a single results table (classification_benchmark_results.csv). Interpret zero-shot scPRINT's ~62% accuracy relative to CellTypist's database-coverage ceiling rather than in absolute terms.

## Related tools

- **scPRINT** (Zero-shot cell label predictor whose predictions are scored at 4000-gene context with random gene sampling; no fine-tuning on the test data) — https://github.com/cantinilab/scPRINT
- **CellTypist** (Marker-based reference classifier run with default parameters on the 'Adult_Human_PancreaticIslet' database (14 cell types) over normalized, log1p-transformed counts)
- **scikit-learn** (Provides the RidgeClassifier used for head selection in scPRINT and the accuracy/macro-F1 metric computations for the benchmark)
- **Python** (Execution environment for running all classifiers, openproblems baselines, and metric compilation)

## Examples

```
from sklearn.metrics import accuracy_score, f1_score; acc = accuracy_score(y_true, y_pred); mf1 = f1_score(y_true, y_pred, average='macro')
```

## Evaluation signals

- The results table contains both accuracy and macro-F1 for every method and for both label granularities (fine-grained and coarse 'endocrine pancreatic cell' aggregation)
- CellTypist's database-coverage ceiling is reflected: only 4 of its 14 'Adult_Human_PancreaticIslet' cell types cover ~70% of cells, capping its achievable accuracy relative to scPRINT's zero-shot predictions
- Zero-shot scPRINT reaches approximately 62% classification accuracy and largely outperforms the marker-based CellTypist classifier
- Metrics were re-computed after rolling fine-grained labels up to the coarser ontology aggregation, and the ranking changes are consistent across granularity
- scPRINT predictions were produced without any training on the test dataset (train-test split done by batch per the openproblems preprocessing)

## Limitations

- Label vocabulary coverage dominates achievable accuracy: CellTypist is limited by its 14-cell-type database, so cross-method comparison must account for each method's label space
- Scores are granularity-dependent; conclusions at fine-grained level may not hold after rolling labels up to coarser ontology terms
- No perfect ground truths exist for cell label prediction benchmarks in general, so accuracy and macro-F1 are reference-dependent measures

## Evidence

- [abstract] competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
- [methods] Due to the hierarchical structure of the prediction, we also created a hierarchical loss.: "Due to the hierarchical structure of the prediction, we also created a hierarchical loss."
- [methods] we use a linear classifier RidgeClassifier from scikit-learn130 (with an L2 penalty set to 1, a positivity constraint on the coefficients: "we use a linear classifier RidgeClassifier from scikit-learn130 (with an L2 penalty set to 1, a positivity constraint on the coefficients"
- [other] Compute accuracy and macro-F1 (average F1 across classes) for every method's predictions.: [evidence span withheld — anchor status: not_found]
