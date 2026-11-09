---
name: supervised-classifier-training-and-evaluation
description: Use when cell-type labels are available for scRNA-seq data and you need
  to benchmark classification performance of alternative feature sets (e.g., CKG vs
  gene expression) across multiple classifiers and train/test splits.
when_to_use_negative:
- Input data lacks cell-type labels, so supervised training is impossible (scSimClassify's
  classification module requires labeled cells)
- The goal is unsupervised cell clustering or label-free annotation rather than benchmarking
  labeled classification accuracy
- Features are already CKG cell matrices and the question is upstream k-mer/CKG generation,
  not classifier training
edam_operation: http://edamontology.org/operation_3923
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: keras
  role: Implements the MLP classifier trained on GE and CKG feature sets
  repo: github.com/digi2002/scSimClassify
- name: sklearn
  role: Provides random forest, gradient boosting, SVM classifiers and accuracy/weighted-F1
    metrics
  repo: github.com/digi2002/scSimClassify
- name: pandas
  role: Loads feature matrices and labels and aggregates scores into the comparison
    table
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: Numeric aggregation of classifier scores into the results table
  repo: github.com/digi2002/scSimClassify
- name: simhash
  role: Produces the CKG features (compressed k-mer groups) that serve as one side
    of the classifier comparison
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:64c94e450d1a7575070195cbf8001934c471e27e7a14049cf484517456521b11
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
  edam_operation: http://edamontology.org/operation_3923
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# supervised-classifier-training-and-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains four supervised classifiers (keras MLP, sklearn random forest, gradient boosting, and SVM) on alternative feature representations of scRNA-seq cells and scores them with accuracy and weighted-F1 to compare compressed k-mer group (CKG) features against gene expression (GE) features.

## When to use

Use when cell-type labels are available for scRNA-seq data and you need to benchmark classification performance of alternative feature sets (e.g., CKG vs gene expression) across multiple classifiers and train/test splits.

## When NOT to use

- Input data lacks cell-type labels, so supervised training is impossible (scSimClassify's classification module requires labeled cells)
- The goal is unsupervised cell clustering or label-free annotation rather than benchmarking labeled classification accuracy
- Features are already CKG cell matrices and the question is upstream k-mer/CKG generation, not classifier training

## Inputs

- Gene expression feature matrices with cell labels (pandas DataFrames) for Chuang, Karaayvaz, PBMC3k, and Lee scRNA-seq datasets
- Eight CKG feature variation matrices per dataset, produced from k-mer matrices via the simhash-based CKG look-up table
- Train/test split configuration for each feature set

## Outputs

- classification_results.csv: a single comparison table of accuracy and weighted-F1 scores for MLP, RF, GBM, and SVM across GE and CKG feature sets on all datasets
- A majority-case comparison statement of CKG vs GE classification performance

## How to apply

Load the gene expression feature matrices, cell labels, and CKG feature variations (derived via simhash compression of k-mer vectors) with pandas for each dataset (e.g., Chuang, Karaayvaz, PBMC3k, Lee). Split each dataset into train and test partitions for both the GE features and each of the eight CKG feature variations. Train the four classification methods — a keras MLP plus sklearn random forest, gradient boosting, and SVM — on each feature set, predict held-out labels, and compute accuracy and weighted-F1 with sklearn.metrics. Aggregate all scores into a single pandas table (classification_results.csv), then count the cases where CKG outperforms GE to test the paper's claim that CKG wins 'in the majority of experimental cases'. Rationale: CKGs are derived from raw reads without reference-genome alignment, so winning this comparison supports scSimClassify as an alternative when the reference genome is incomplete.

## Related tools

- **keras** (Implements the MLP classifier trained on GE and CKG feature sets) — github.com/digi2002/scSimClassify
- **sklearn** (Provides random forest, gradient boosting, SVM classifiers and accuracy/weighted-F1 metrics) — github.com/digi2002/scSimClassify
- **pandas** (Loads feature matrices and labels and aggregates scores into the comparison table) — github.com/digi2002/scSimClassify
- **numpy** (Numeric aggregation of classifier scores into the results table) — github.com/digi2002/scSimClassify
- **simhash** (Produces the CKG features (compressed k-mer groups) that serve as one side of the classifier comparison) — github.com/digi2002/scSimClassify

## Examples

```
import pandas as pd; from sklearn.ensemble import RandomForestClassifier; from sklearn.metrics import accuracy_score, f1_score; clf = RandomForestClassifier().fit(X_train, y_train); acc = accuracy_score(y_test, clf.predict(X_test))
```

## Evaluation signals

- classification_results.csv contains one accuracy and one weighted-F1 entry per (dataset, classifier, feature-set) combination with no missing cells
- For every dataset, all four classifiers (MLP, RF, GBM, SVM) run to completion on both GE and each of the eight CKG variations
- The majority-count check over reported cases confirms CKG accuracy exceeds GE accuracy in more than half of the cases
- Test-set predictions are made strictly on held-out splits, so scores reflect generalization rather than training-set fit
- Weighted-F1 values stay within [0, 1] and are consistent with the reported accuracy for the same run

## Limitations

- Conclusion is comparative, not absolute: CKG superiority holds 'in the majority of experimental cases', so individual dataset/classifier pairs may still favor GE features
- Results depend on the CKG generation step (informative k-mer selection and simhash fingerprinting); poor CKG tables propagate into degraded classification
- Requires python 3.6 and the pinned package set (simhash, keras, sklearn, pandas, numpy, Bio); repository usage requires manually modifying paths to use the modules

## Evidence
