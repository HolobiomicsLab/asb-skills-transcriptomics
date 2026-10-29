---
name: classification-metrics-evaluation
description: Use when a trained cell-type classifier (e.g., a Keras neural network
  on CKG or gene-expression features) must be scored against known labels, or when
  comparing classifiers across dataset pairs.
when_to_use_negative:
- Input is raw gene expression or k-mer data that has not yet been classified — score
  computation presupposes predicted labels and ground-truth labels already exist
- The task requires unsupervised clustering quality measures rather than supervised
  label prediction against known cell labels
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: sklearn
  role: Provides sklearn.metrics utilities for computing accuracy and weighted F1
    score, plus label handling for predicted and true cell-type labels
  repo: github.com/digi2002/scSimClassify
- name: pandas
  role: Writes the computed accuracy and weighted F1 values for all dataset pairs
    to a results CSV
  repo: github.com/digi2002/scSimClassify
- name: keras
  role: Neural network classifier whose predictions are the input to the metric evaluation
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:71e9d9948cc2354c653c0bd3a6e548167d18a48e703e57c77cca371aca66fe30
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/digi2002/scSimClassify
  tools:
  - keras
  - numpy
  - pandas
  - sklearn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# classification-metrics-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates scRNA-seq cell-type classifiers by computing accuracy and weighted F1 score against known cell labels with sklearn.metrics, and comparing feature representations (CKG vs gene expression) or cross-dataset transfer outcomes.

## When to use

Use when a trained cell-type classifier (e.g., a Keras neural network on CKG or gene-expression features) must be scored against known labels, or when comparing classifiers across dataset pairs.

## When NOT to use

- Input is raw gene expression or k-mer data that has not yet been classified — score computation presupposes predicted labels and ground-truth labels already exist
- The task requires unsupervised clustering quality measures rather than supervised label prediction against known cell labels

## Inputs

- Predicted cell-type labels for target-dataset cells from the trained scSimClassify model
- Ground-truth cell labels for the target dataset (e.g., Karaayvaz, PBMC3k, or Lee cells)
- One or more dataset pairs (training dataset, target dataset), e.g., Chuang→Karaayvaz and PBMC3k→Lee

## Outputs

- Accuracy and weighted F1 score per experimental condition (e.g., cross_dataset_classification_results.csv written with pandas)
- Comparison of CKG-feature model scores against gene-expression-feature baseline scores

## How to apply

Take the predicted cell-type labels from the scSimClassify classifier and the target dataset's known cell labels, then compute accuracy and weighted F1 score using sklearn.metrics. Weighted F1 is used because cell-type classes can be imbalanced across scRNA-seq datasets. For inter-dataset experiments (e.g., train on Chuang's dataset, predict on Karaayvaz's dataset, or the PBMC3k/Lee pair), compute metrics separately for each dataset pair. Write the accuracy and weighted F1 values for all experimental conditions to a results CSV (e.g., cross_dataset_classification_results.csv) with pandas so CKG-feature and gene-expression-feature baselines can be compared directly.

## Related tools

- **sklearn** (Provides sklearn.metrics utilities for computing accuracy and weighted F1 score, plus label handling for predicted and true cell-type labels) — github.com/digi2002/scSimClassify
- **pandas** (Writes the computed accuracy and weighted F1 values for all dataset pairs to a results CSV) — github.com/digi2002/scSimClassify
- **keras** (Neural network classifier whose predictions are the input to the metric evaluation) — github.com/digi2002/scSimClassify

## Examples

```
from sklearn.metrics import accuracy_score, f1_score; acc = accuracy_score(y_true, y_pred); wf1 = f1_score(y_true, y_pred, average='weighted'); results.to_csv('cross_dataset_classification_results.csv')
```

## Evaluation signals

- Accuracy and weighted F1 are both finite values in [0, 1] for each dataset pair
- CKG-feature model accuracy and weighted F1 meet or exceed the gene-expression-feature baseline, consistent with the paper's finding that CKG features perform better in the majority of cases
- Results CSV contains one accuracy and one weighted F1 entry per dataset pair (e.g., Chuang→Karaayvaz and PBMC3k→Lee)
- Predicted label set matches the target dataset's known cell-label set (label space consistency via sklearn label handling)

## Limitations

- CKG features outperform gene expression features only 'in the majority of experimental cases', so the CKG-vs-baseline comparison may not hold for every dataset pair
- Weighted F1 is sensitive to cell-type class imbalance; rare cell types may contribute little to the weighted score

## Evidence
