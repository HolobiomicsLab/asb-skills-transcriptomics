---
name: cell-type-label-prediction
description: Use when annotating cell types in scRNA-seq data without training on
  the test dataset, or when benchmarking a zero-shot label predictor against supervised
  classifiers like CellTypist on a multi-batch atlas such as the open-problems pancreas
  dataset.
when_to_use_negative:
- The task is gene network inference rather than label prediction — scPRINT's headline
  benchmarking targets GN inference, and this zero-shot annotation skill does not
  produce gene networks
- A labeled training split for the target atlas is available and a supervised model
  can be fine-tuned on it — the zero-shot setup is specifically for predictors that
  do not train on the test dataset
- The test data is spatial omics or non-single-cell assay data, which the scPRINT
  preprocessing explicitly drops
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPRINT
  role: Pretrained foundation model used as the zero-shot cell label predictor with
    random gene sampling at a 4000-gene context
  repo: https://github.com/cantinilab/scPRINT
- name: CellTypist
  role: Supervised marker-based classifier baseline run with the 'Adult_Human_PancreaticIslet'
    database on normalized log1p counts
- name: scikit-learn
  role: Provides the RidgeClassifier-style linear machinery underlying benchmark classifiers
    and metric computation (accuracy, macro-F1)
- name: Python
  role: Execution environment for the scPRINT inference and openproblems benchmark
    pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-58699-1
    title: 'scPRINT: pre-training on 50 million cells allows robust gene network predictions'
  source_content_hash: sha256:16be9b888d6336c13703212cae6a6a760f7531009ec84059dc761596e4e76ed6
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# cell-type-label-prediction

## Summary

Zero-shot cell-type label prediction using the pretrained scPRINT foundation model (random gene sampling, 4000-gene context), benchmarked against supervised classifiers such as CellTypist and openproblems logistic regression/XGBoost using accuracy and macro-F1.

## When to use

Use when annotating cell types in scRNA-seq data without training on the test dataset, or when benchmarking a zero-shot label predictor against supervised classifiers like CellTypist on a multi-batch atlas such as the open-problems pancreas dataset.

## When NOT to use

- The task is gene network inference rather than label prediction — scPRINT's headline benchmarking targets GN inference, and this zero-shot annotation skill does not produce gene networks
- A labeled training split for the target atlas is available and a supervised model can be fine-tuned on it — the zero-shot setup is specifically for predictors that do not train on the test dataset
- The test data is spatial omics or non-single-cell assay data, which the scPRINT preprocessing explicitly drops

## Inputs

- Multi-batch single-cell RNA-seq dataset with cell-type labels (e.g. open-problems human pancreas atlas, .h5ad/AnnData)
- Pretrained scPRINT model checkpoint (e.g. from https://huggingface.co/jkobject/scPRINT)
- CellTypist cell-type reference database (e.g. 'Adult_Human_PancreaticIslet', 14 cell types)
- Count-normalized, log1p-transformed expression matrix for the supervised baselines

## Outputs

- Per-cell predicted cell-type labels from each method (scPRINT zero-shot, CellTypist, openproblems LR and XGBoost)
- Per-method, per-granularity metrics table (accuracy and macro-F1, e.g. classification_benchmark_results.csv)
- Metrics re-evaluated at coarse ontology granularity ('endocrine pancreatic cell' aggregation)

## How to apply

Load the multi-batch scRNAseq atlas and split train-test by batch (openproblems benchmark preprocessing). Run the pretrained scPRINT model zero-shot — no fine-tuning — using its random sampling of genes approach at a context of 4000 genes to predict cell type labels. Run CellTypist with default parameters against the 'Adult_Human_PancreaticIslet' database (14 cell types) on normalized, log1p-transformed counts over the full gene set, and run openproblems logistic regression and xgboost classifiers on the 25 principal components of the count-normalized, log1p-transformed expression. Compute accuracy and macro-F1 (average F1 across classes) per method, then re-evaluate all predictions after rolling fine-grained labels up to the coarser 'endocrine pancreatic cell' ontology aggregation. Note that scPRINT's multi-task pretraining includes a classification task with a hierarchical loss due to the hierarchical structure of the prediction, which underpins its zero-shot label prediction ability.

## Related tools

- **scPRINT** (Pretrained foundation model used as the zero-shot cell label predictor with random gene sampling at a 4000-gene context) — https://github.com/cantinilab/scPRINT
- **CellTypist** (Supervised marker-based classifier baseline run with the 'Adult_Human_PancreaticIslet' database on normalized log1p counts)
- **scikit-learn** (Provides the RidgeClassifier-style linear machinery underlying benchmark classifiers and metric computation (accuracy, macro-F1))
- **Python** (Execution environment for the scPRINT inference and openproblems benchmark pipeline)

## Evaluation signals

- Results table contains both accuracy and macro-F1 for every method at both fine-grained and coarse ('endocrine pancreatic cell') label granularities
- scPRINT predictions are produced with no fine-tuning on the test dataset — training split is disjoint from the test split by batch, per openproblems preprocessing
- scPRINT zero-shot accuracy (~62% on the pancreas benchmark) should exceed the marker-based CellTypist baseline
- CellTypist accuracy should reflect its ceiling: only 4 of 14 database cell types cover ~70% of cells, limiting the labels it can assign
- Openproblems baselines (logistic regression, xgboost) are fit on 25 principal components of count-normalized, log1p-transformed expression, not raw counts

## Limitations

- CellTypist is ceiling-limited by its reference database coverage (4 of 14 'Adult_Human_PancreaticIslet' cell types covering ~70% of cells), which depresses its apparent accuracy relative to a zero-shot predictor
- Zero-shot performance depends on the pretrained checkpoint; the README notes the scPRINT-2 next-generation model has been released at github.com/cantinilab/scPRINT-2, so version drift affects reproducibility of published numbers
- Fine-grained labels can be dominated by ontology granularity; re-evaluating after rolling labels up to the coarser aggregation can change method rankings

## Evidence

- [abstract] competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction: "competitive zero-shot abilities in denoising, batch effect correction, and cell label prediction"
- [methods] hierarchical loss for classification task: "Due to the hierarchical structure of the prediction, we also created a hierarchical loss."
- [methods] weighted random sampling of training data: "we use weighted random sampling on our training data based on the different class values we have to predict. We use a factor of 50"
- [readme] scPRINT-2 release notice (version drift caveat): "🎊 The scPRINT-2 model has now been released:"
- [abstract] pre-training corpus: "pre-trained on more than 50 million cells from the cellxgene database"
