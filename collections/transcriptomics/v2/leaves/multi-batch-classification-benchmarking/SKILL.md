---
name: multi-batch-classification-benchmarking
description: Use when comparing a pretrained zero-shot label predictor (e.g. scPRINT)
  against supervised classifiers (CellTypist, openproblems logistic regression/xgboost)
  on multi-batch single-cell data where the test set comes from held-out batches.
when_to_use_negative:
- The goal is gene network inference benchmarking (use scPRINT's BenGRN/GRnnData suite
  instead of label prediction metrics).
- Fine-tuning on the target dataset is allowed — the headline comparison here rests
  on scPRINT being a zero-shot predictor that does not train on the test dataset.
- The dataset has a single batch, so a train-test split by batch (and the batch-effect
  challenge) cannot be constructed.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scPRINT
  role: zero-shot cell label predictor run on held-out batches with random gene sampling
    at a 4000-gene context
  repo: github.com/cantinilab/scPRINT
- name: CellTypist
  role: marker-based supervised classifier baseline using the Adult_Human_PancreaticIslet
    database
- name: scikit-learn
  role: provides metrics and linear classifiers (e.g. RidgeClassifier) used in the
    scPRINT/benchmarking ecosystem
- name: Python
  role: implementation environment for the benchmark pipeline and results compilation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:48ce649e02a35b2c5b1b8b527db1b2811a349f3208ef816bd7378611ee1b97fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellTypist
  - Python
  - scPRINT
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# multi-batch-classification-benchmarking

## Summary

Benchmark cell-type label prediction methods head-to-head on a multi-batch scRNAseq dataset by splitting train-test by batch, running zero-shot and supervised classifiers, and computing accuracy and macro-F1 at multiple label granularities.

## When to use

Use when comparing a pretrained zero-shot label predictor (e.g. scPRINT) against supervised classifiers (CellTypist, openproblems logistic regression/xgboost) on multi-batch single-cell data where the test set comes from held-out batches.

## When NOT to use

- The goal is gene network inference benchmarking (use scPRINT's BenGRN/GRnnData suite instead of label prediction metrics).
- Fine-tuning on the target dataset is allowed — the headline comparison here rests on scPRINT being a zero-shot predictor that does not train on the test dataset.
- The dataset has a single batch, so a train-test split by batch (and the batch-effect challenge) cannot be constructed.

## Inputs

- multi-batch pancreas scRNAseq dataset (count matrix with batch and cell-type annotations, e.g. .h5ad)
- CellTypist 'Adult_Human_PancreaticIslet' reference database (14 cell types)
- pretrained scPRINT checkpoint (available from huggingface.co/jkobject/scPRINT)
- count-normalized, log1p-transformed expression matrix for PCA-based baselines

## Outputs

- classification_benchmark_results.csv with accuracy and macro-F1 per method and label granularity
- per-method predicted cell type labels for the held-out test batches
- coarse-grained ('endocrine pancreatic cell') metric re-evaluation

## How to apply

Load a multi-batch pancreas scRNAseq dataset and split train-test by batch following the openproblems benchmark preprocessing. Run the pretrained scPRINT model zero-shot (no fine-tuning) using its random sampling of genes at a context of 4000 genes to predict cell labels, exploiting the hierarchical classification loss built into its multi-task pretraining. Run CellTypist with default parameters on the 'Adult_Human_PancreaticIslet' database (14 cell types) over normalized, log1p-transformed counts, and run openproblems logistic regression and xgboost on 25 principal components of count-normalized, log1p-transformed expression. Compute accuracy and macro-F1 for every method, then re-evaluate after rolling fine-grained labels up to the coarser 'endocrine pancreatic cell' ontology aggregation, and compile per-method, per-granularity metrics into classification_benchmark_results.csv.

## Related tools

- **scPRINT** (zero-shot cell label predictor run on held-out batches with random gene sampling at a 4000-gene context) — github.com/cantinilab/scPRINT
- **CellTypist** (marker-based supervised classifier baseline using the Adult_Human_PancreaticIslet database)
- **scikit-learn** (provides metrics and linear classifiers (e.g. RidgeClassifier) used in the scPRINT/benchmarking ecosystem)
- **Python** (implementation environment for the benchmark pipeline and results compilation)

## Evaluation signals

- classification_benchmark_results.csv contains accuracy and macro-F1 for each method at both label granularities
- CellTypist ceiling is reflected in its accuracy: only 4 of 14 database cell types cover ~70% of cells, capping its achievable accuracy relative to scPRINT
- scPRINT reaches ~62% classification accuracy on the pancreas benchmark despite training on no test data
- train-test split is by batch, so no cells from the same batch appear in both training and test sets
- macro-F1 is averaged across classes, so rare-type performance is penalized if coarse/fine label mapping is wrong

## Limitations

- CellTypist performance is bounded by database coverage — only 4 of its 14 'Adult_Human_PancreaticIslet' cell types cover ~70% of the pancreas test cells
- Fine-grained vs coarse label granularity changes the ranking and must be reported separately; results should be re-evaluated after rolling labels up to 'endocrine pancreatic cell'
- Zero-shot prediction is limited by the pretrained model's known cell vocabulary and its random sampling of 4000 genes as context, not by any dataset-specific tuning

## Evidence

- [abstract] competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
- [methods] hierarchical classification loss: "Due to the hierarchical structure of the prediction, we also created a hierarchical loss."
- [other] multi-batch pancreas benchmark workflow: [evidence span withheld — anchor status: not_found]
- [other] scPRINT accuracy result: [evidence span withheld — anchor status: not_found]
