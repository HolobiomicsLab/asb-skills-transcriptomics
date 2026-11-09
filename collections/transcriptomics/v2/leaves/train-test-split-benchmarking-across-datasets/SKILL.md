---
name: train-test-split-benchmarking-across-datasets
description: Use when two or more feature representations must be compared for the
  same classification task across several scRNA-seq datasets (e.g., Chuang, Karaayvaz,
  PBMC3k, Lee), using held-out test partitions and metrics such as accuracy and weighted-F1.
when_to_use_negative:
- Input is already a scored comparison table or you only need a single classifier
  on a single feature set — a full cross-dataset benchmark is overkill
- The task is de novo feature construction (k-mer matrix generation, CKG generation,
  or cell matrix generation) rather than evaluating already-built feature matrices
- No held-out labels exist — train/test benchmarking requires ground-truth cell type
  labels to compute accuracy and weighted-F1
edam_operation: http://edamontology.org/operation_2990
tools:
- name: sklearn
  role: Provides train/test splitting, random forest, gradient boosting, and SVM classifiers,
    and accuracy/weighted-F1 metrics
  repo: https://github.com/digi2002/scSimClassify
- name: keras
  role: Implements the MLP classifier used as one of the four classification methods
  repo: https://github.com/digi2002/scSimClassify
- name: pandas
  role: Loads feature matrices/labels and aggregates scores into the comparison table
  repo: https://github.com/digi2002/scSimClassify
- name: numpy
  role: Supports aggregation of scores into the single comparison table
  repo: https://github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:16ef954b3290f245477b01af8091f06783508bf1057b2c0448fa5fa1eda10df8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/digi2002/scSimClassify
  repo_url: https://github.com/digi2002/scSimClassify
  tools:
  - Python
  - keras
  - numpy
  - pandas
  - sklearn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# train-test-split-benchmarking-across-datasets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks alternative feature representations (e.g., CKG vs. gene expression features) for cell type classification by running identical train/test splits across multiple scRNA-seq datasets and multiple classifiers, then aggregating accuracy and weighted-F1 into one comparison table.

## When to use

Use when two or more feature representations must be compared for the same classification task across several scRNA-seq datasets (e.g., Chuang, Karaayvaz, PBMC3k, Lee), using held-out test partitions and metrics such as accuracy and weighted-F1.

## When NOT to use

- Input is already a scored comparison table or you only need a single classifier on a single feature set — a full cross-dataset benchmark is overkill
- The task is de novo feature construction (k-mer matrix generation, CKG generation, or cell matrix generation) rather than evaluating already-built feature matrices
- No held-out labels exist — train/test benchmarking requires ground-truth cell type labels to compute accuracy and weighted-F1

## Inputs

- Gene expression feature matrices per scRNA-seq dataset (pandas-readable tables)
- Cell type labels per dataset
- Multiple CKG feature matrices per dataset (eight feature variations derived from k-mer abundances via the CKG look-up table)
- Train/test split specification for each dataset and feature set

## Outputs

- classification_results.csv — table of accuracy and weighted-F1 per dataset, per feature set, per classifier
- Win/loss comparison of CKG-feature scores versus gene-expression-feature scores across all reported cases

## How to apply

Load each dataset's gene expression feature matrix, cell labels, and every feature variation to test (here, eight CKG feature variations generated via simhash-compressed k-mer groups). For each dataset and each feature set, create train and test partitions so every feature representation is evaluated on the same split structure. Train four classifiers per feature set — an MLP built with keras, plus random forest, gradient boosting, and SVM from sklearn — predict held-out labels, and score accuracy and weighted-F1 with sklearn.metrics. Aggregate all runs into a single pandas/numpy comparison table (e.g., classification_results.csv) so per-dataset, per-classifier wins of one feature type over the other can be counted; the reference study concluded CKG features beat gene expression features in the majority of experimental cases.

## Related tools

- **sklearn** (Provides train/test splitting, random forest, gradient boosting, and SVM classifiers, and accuracy/weighted-F1 metrics) — https://github.com/digi2002/scSimClassify
- **keras** (Implements the MLP classifier used as one of the four classification methods) — https://github.com/digi2002/scSimClassify
- **pandas** (Loads feature matrices/labels and aggregates scores into the comparison table) — https://github.com/digi2002/scSimClassify
- **numpy** (Supports aggregation of scores into the single comparison table) — https://github.com/digi2002/scSimClassify

## Examples

```
from sklearn.model_selection import train_test_split; X_tr, X_te, y_tr, y_te = train_test_split(X, labels, test_size=0.3, stratify=labels)
```

## Evaluation signals

- classification_results.csv exists and contains one accuracy and one weighted-F1 score per (dataset × feature set × classifier) combination
- For every dataset, both the gene expression features and all eight CKG variations are scored with all four classifiers (MLP, RF, GBM, SVM)
- Train/test partitions for the gene expression features and each CKG variation follow the same split structure per dataset so scores are comparable
- Counting per-case wins, CKG features outperform gene expression features in the majority of experimental cases, reproducing the reported finding

## Limitations

- The comparison is intra-dataset (train and test partitions come from the same scRNA-seq dataset); cross-dataset generalization is not covered by this benchmark
- Conclusions depend on the four classifiers and datasets chosen (Chuang, Karaayvaz, PBMC3k, Lee); other datasets or classifiers may shift the majority outcome
- The repository README only ships cell labels and example gene expression features, so CKG feature matrices must be regenerated via the k-mer/CKG/cell matrix modules before benchmarking

## Evidence
