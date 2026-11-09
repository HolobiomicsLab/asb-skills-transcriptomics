---
name: ckg-versus-gene-expression-feature-comparison
description: Decision support for whether reference-free CKG features should replace
  gene expression features when classifying cell types in scRNA-seq data, particularly
  when the reference genome may be incomplete or insufficient.
when_to_use_negative:
- Your goal is the CKG generation pipeline itself (k-mer matrix, CKG lookup table,
  cell matrix) rather than the downstream classification comparison
- Your data are bulk RNA-seq or non-transcriptomic reads where k-mer abundance profiles
  per single cell are not available
- You require a single best classifier rather than a feature-representation comparison
  across four classifiers
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: keras
  role: Implements the MLP classifier used in the intra-dataset CKG-vs-GE comparison
  repo: github.com/digi2002/scSimClassify
- name: sklearn
  role: Provides random forest, gradient boosting, and SVM classifiers plus accuracy
    and weighted-F1 metrics
  repo: github.com/digi2002/scSimClassify
- name: pandas
  role: Loads feature matrices and labels, and assembles the aggregated comparison
    table
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: Numeric aggregation of classification scores into the comparison table
  repo: github.com/digi2002/scSimClassify
- name: simhash
  role: Generates the CKG fingerprints that serve as the reference-free feature set
    being benchmarked
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:f126ddb7951d85c54a4750ec9f8540f66a7e5cb832ab2f8ac48235ebdd4961f2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/digi2002/scSimClassify
  tools:
  - Python
  - keras
  - numpy
  - pandas
  - simhash
  - sklearn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# ckg-versus-gene-expression-feature-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark protocol comparing compressed k-mer group (CKG) features — simhash-derived k-mer clusters used by scSimClassify — against conventional gene expression (GE) features for scRNA-seq cell type classification, using MLP, random forest, gradient boosting, and SVM classifiers evaluated by accuracy and weighted-F1 across multiple datasets.

## When to use

Decision support for whether reference-free CKG features should replace gene expression features when classifying cell types in scRNA-seq data, particularly when the reference genome may be incomplete or insufficient.

## When NOT to use

- Your goal is the CKG generation pipeline itself (k-mer matrix, CKG lookup table, cell matrix) rather than the downstream classification comparison
- Your data are bulk RNA-seq or non-transcriptomic reads where k-mer abundance profiles per single cell are not available
- You require a single best classifier rather than a feature-representation comparison across four classifiers

## Inputs

- Gene expression feature matrices with cell type labels for scRNA-seq datasets (Chuang, Karaayvaz, PBMC3k, Lee)
- Eight CKG feature variation matrices per dataset produced by the scSimClassify cell matrix generation module
- Train/test split definitions for each dataset and feature set

## Outputs

- classification_results.csv aggregating accuracy and weighted-F1 for every classifier x feature-set x dataset combination
- Majority-case comparison of CKG vs GE scores confirming or refuting CKG superiority

## How to apply

Load gene expression feature matrices with cell labels plus the eight CKG feature variations for each scRNA-seq dataset (e.g., Chuang, Karaayvaz, PBMC3k, Lee) using pandas. Split each dataset into train/test partitions for both the GE features and each CKG variation. Train four classifiers — an MLP implemented with keras, and random forest, gradient boosting, and SVM from sklearn — on each feature set, predict held-out cell type labels, and compute accuracy and weighted-F1 with sklearn.metrics. Aggregate all scores into a single comparison table (classification_results.csv) with pandas/numpy, then count how many experimental cases CKG beats GE; the paper's claim holds when CKG wins in the majority of cases.

## Related tools

- **keras** (Implements the MLP classifier used in the intra-dataset CKG-vs-GE comparison) — github.com/digi2002/scSimClassify
- **sklearn** (Provides random forest, gradient boosting, and SVM classifiers plus accuracy and weighted-F1 metrics) — github.com/digi2002/scSimClassify
- **pandas** (Loads feature matrices and labels, and assembles the aggregated comparison table) — github.com/digi2002/scSimClassify
- **numpy** (Numeric aggregation of classification scores into the comparison table) — github.com/digi2002/scSimClassify
- **simhash** (Generates the CKG fingerprints that serve as the reference-free feature set being benchmarked) — github.com/digi2002/scSimClassify

## Examples

```
import pandas as pd; from sklearn.ensemble import RandomForestClassifier; from sklearn.metrics import accuracy_score, f1_score; acc = accuracy_score(y_test, RandomForestClassifier().fit(X_train, y_train).predict(X_test))
```

## Evaluation signals

- classification_results.csv exists and contains one row per dataset x feature-set (GE + eight CKG variations) x classifier (MLP, RF, GBM, SVM) with both accuracy and weighted-F1 columns
- Every cell in the table carries a valid score in [0, 1] for both metrics
- Comparing CKG rows against the GE row per dataset-classifier pair, CKG wins in the majority of cases, matching the paper's abstract claim
- Each of the four classification methods runs on both keras (MLP) and sklearn (RF, GBM, SVM) implementations without label leakage between train and test splits

## Limitations

- The comparison is intra-dataset; the paper's majority-case superiority claim was established on four specific scRNA-seq datasets (Chuang, Karaayvaz, PBMC3k, Lee) and may not transfer to other datasets
- The evaluation covers only four classification methods; other classifier families are untested in this comparison
- The README gives no fixed train/test ratio or hyperparameters — 'Please modify the path to use the modules' — so reproduction depends on user-specified splits

## Evidence
