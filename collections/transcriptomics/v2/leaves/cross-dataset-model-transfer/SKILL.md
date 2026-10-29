---
name: cross-dataset-model-transfer
description: Use when a trained scRNA-seq cell-type classifier must be transferred
  to a second dataset with different cells but overlapping cell-type labels, and prediction
  accuracy/F1 must be quantified against the target dataset's known labels.
when_to_use_negative:
- The goal is within-dataset classification (train and test cells come from the same
  dataset), which is the standard classification module use case rather than inter-dataset
  transfer
- The two datasets do not share comparable cell types or label definitions, so no
  meaningful cross-dataset ground truth exists for evaluation
- Input is raw reads/k-mer data where the CKG generation modules (k-mer matrix, CKG
  look-up, cell matrix generation) have not yet been run
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: keras
  role: Builds and trains the neural-network classifier on the training dataset's
    CKG features, reused to predict target-dataset cell types
  repo: github.com/digi2002/scSimClassify
- name: simhash
  role: Generates the CKG look-up table used to transform both datasets' k-mer abundance
    profiles into shared CKG features
  repo: github.com/digi2002/scSimClassify
- name: sklearn
  role: Label handling for cell types and computation of accuracy and weighted F1
    via sklearn.metrics
  repo: github.com/digi2002/scSimClassify
- name: pandas
  role: Loads the feature matrices/labels and writes cross_dataset_classification_results.csv
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: Array handling for expression and CKG feature matrices
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:cd57708a2691ed6e361e41d3395e7fba56179a5c82ac88bdde2a8a59d3abb9ed
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
  - simhash
  - sklearn
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cross-dataset-model-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a scSimClassify cell-type classifier on one scRNA-seq dataset and apply it to classify shared cell types in an independent dataset (e.g., train on Chuang, predict Karaayvaz), using simhash-derived CKG features rather than raw gene expression.

## When to use

Use when a trained scRNA-seq cell-type classifier must be transferred to a second dataset with different cells but overlapping cell-type labels, and prediction accuracy/F1 must be quantified against the target dataset's known labels.

## When NOT to use

- The goal is within-dataset classification (train and test cells come from the same dataset), which is the standard classification module use case rather than inter-dataset transfer
- The two datasets do not share comparable cell types or label definitions, so no meaningful cross-dataset ground truth exists for evaluation
- Input is raw reads/k-mer data where the CKG generation modules (k-mer matrix, CKG look-up, cell matrix generation) have not yet been run

## Inputs

- Gene expression feature matrix for the training dataset (e.g., Chuang) with cell labels
- Gene expression feature matrix for the target dataset (e.g., Karaayvaz) with known cell labels for evaluation
- CKG look-up table from the CKG generation module (simhash fingerprints of informative k-mers)

## Outputs

- Predicted cell-type labels for every cell in the target dataset
- Accuracy and weighted F1 score per dataset pair (e.g., Chuang->Karaayvaz, PBMC3k->Lee)
- cross_dataset_classification_results.csv written with pandas

## How to apply

Load gene expression feature matrices and cell labels for both datasets (e.g., training set Chuang, target set Karaayvaz) with pandas and numpy. Transform expression features into compressed k-mer group (CKG) features via the simhash-based CKG look-up table, since CKGs identified by simhash contain k-mers with similar abundance profiles and outperform gene expression features in the majority of experimental cases. Train a Keras neural-network classifier on the training dataset's CKG features and labels, then apply the frozen model to the target dataset's CKG features, handling labels with sklearn utilities. Optionally repeat the train-on-one/classify-on-other procedure for the reverse pairing and for the PBMC3k/Lee pair. Compute accuracy and weighted F1 with sklearn.metrics for each cross-dataset prediction, and write both dataset pairs' results to cross_dataset_classification_results.csv with pandas.

## Related tools

- **keras** (Builds and trains the neural-network classifier on the training dataset's CKG features, reused to predict target-dataset cell types) — github.com/digi2002/scSimClassify
- **simhash** (Generates the CKG look-up table used to transform both datasets' k-mer abundance profiles into shared CKG features) — github.com/digi2002/scSimClassify
- **sklearn** (Label handling for cell types and computation of accuracy and weighted F1 via sklearn.metrics) — github.com/digi2002/scSimClassify
- **pandas** (Loads the feature matrices/labels and writes cross_dataset_classification_results.csv) — github.com/digi2002/scSimClassify
- **numpy** (Array handling for expression and CKG feature matrices) — github.com/digi2002/scSimClassify

## Examples

```
import pandas as pd, numpy as np; X_tr = pd.read_csv('chuang_ckg_features.csv'); y_tr = pd.read_csv('chuang_labels.csv'); model = keras_model(); model.fit(X_tr, y_tr); preds = model.predict(pd.read_csv('karaayvaz_ckg_features.csv')); from sklearn.metrics import accuracy_score, f1_score; acc = accuracy_score(y_true, preds); f1 = f1_score(y_true, preds, average='weighted'); pd.DataFrame([{'pair':'Chuang->Karaayvaz','accuracy':acc,'weighted_f1':f1}]).to_csv('cross_dataset_classification_results.csv', index=False)
```

## Evaluation signals

- Reported accuracy and weighted F1 for the CKG-feature model meet or exceed the gene-expression-feature baseline on the same dataset pair
- sklearn.metrics accuracy and weighted F1 are computed against the target dataset's known cell labels (not the training labels)
- cross_dataset_classification_results.csv contains rows for both dataset pairs (Chuang/Karaayvaz and PBMC3k/Lee) with accuracy and weighted F1 columns
- The same trained model object is reused for target-dataset prediction (no retraining on target data)
- Feature dimensions of train and target CKG matrices align because both use the same CKG look-up table

## Limitations

- Transfer assumes the two datasets share cell types with consistent label definitions; accuracy is only measurable where target labels exist
- scSimClassify's advantage is framed for settings where the reference genome is incomplete or insufficient; if a complete high-quality reference exists, standard alignment-based approaches may be preferable
- CKG features are derived from k-mer abundance profiles, so the intermediate k-mer matrix and CKG look-up generation steps must be completed before cross-dataset transfer is possible

## Evidence

- [other] Task card validation and output metric: [evidence span withheld — anchor status: not_found]
