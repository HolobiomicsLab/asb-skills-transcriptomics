---
name: svm-classifier-training
description: Use when you have per-sample cell-type abundance estimates (e.g., from
  DCNet deconvolution of TCGA bulk RNA-seq) and need to classify samples into discrete
  subtypes such as LUAD vs LUSC.
when_to_use_negative:
- The input is still raw bulk gene expression and has not yet been deconvolved into
  cell-type abundances by DCNet — run DCNet first to obtain the feature space.
- The goal is survival stratification or risk modeling from TCGA cell landscapes (the
  paper uses consensus clustering and Kaplan-Meier analysis for that), not discrete
  subtype classification.
- The cell-type abundances show no significant difference between the groups being
  classified, leaving the SVM without discriminative features.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DCNet
  role: Infers the 434-cell-type abundance landscape from bulk RNA-seq that supplies
    the classifier's features
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Framework in which the pretrained DCNet model is run to produce per-patient
    cell-type abundances
- name: Python
  role: Language used to train and evaluate the SVM on the significant cell-type features
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:8f326507d240d7991704e18123f8db350c43917ae0f31e7256bcab8807858a36
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# svm-classifier-training

## Summary

Trains a support vector machine (SVM) classifier on DCNet-inferred cell-type abundance features that are significantly different between LUAD and LUSC patients, to predict lung cancer subtype; the paper reports ROC AUC ≈ 0.87 and precision 0.975 on held-out TCGA test samples.

## When to use

Use when you have per-sample cell-type abundance estimates (e.g., from DCNet deconvolution of TCGA bulk RNA-seq) and need to classify samples into discrete subtypes such as LUAD vs LUSC.

## When NOT to use

- The input is still raw bulk gene expression and has not yet been deconvolved into cell-type abundances by DCNet — run DCNet first to obtain the feature space.
- The goal is survival stratification or risk modeling from TCGA cell landscapes (the paper uses consensus clustering and Kaplan-Meier analysis for that), not discrete subtype classification.
- The cell-type abundances show no significant difference between the groups being classified, leaving the SVM without discriminative features.

## Inputs

- Cell-type abundance matrix from DCNet inference (rows = patients, columns = inferred cell types)
- Sample subtype labels (LUAD vs LUSC) for TCGA patients
- TCGA gene expression matrix (rows = human gene symbols, columns = samples), log2(x+1) transformed

## Outputs

- Trained SVM subtype classifier on significantly different cell-type features
- Test-set predictions with ROC AUC and precision metrics
- ROC curve plot (Figure 6d equivalent)

## How to apply

First run the pretrained DCNet model (mxnet; adam optimizer, relu activation, L2loss, learning rate 1e-4, 600 iterations, 256 samples per batch, CPU) on the log2(x+1)-transformed TCGA expression matrix (rows = human gene symbols, columns = samples) to infer cell-type abundances per patient. Statistically test each cell type's abundance between LUAD and LUSC patients and retain only the significantly different cell types as classifier features. Split patients into training and test sets at an 80%/20% ratio, train an SVM in Python on the significant cell-type features, and evaluate ROC AUC and precision on the held-out test set. Correct application reproduces the paper's benchmark: ROC AUC ≈ 0.87 and SVM precision of 0.975.

## Related tools

- **DCNet** (Infers the 434-cell-type abundance landscape from bulk RNA-seq that supplies the classifier's features) — https://github.com/xindd/DCNet
- **mxnet** (Framework in which the pretrained DCNet model is run to produce per-patient cell-type abundances)
- **Python** (Language used to train and evaluate the SVM on the significant cell-type features)

## Examples

```
from sklearn.svm import SVC; clf = SVC(probability=True); clf.fit(X_train, y_train); preds = clf.predict(X_test)
```

## Evaluation signals

- Held-out test-set ROC AUC approximates the reported 0.87 (within reasonable tolerance).
- SVM precision on the test set approximates the reported 0.975.
- Feature set contains only cell types whose abundances are significantly different between LUAD and LUSC.
- Train/test split respects the 80%/20% patient-level ratio.
- Input expression matrix was log2(x+1) transformed before DCNet inference, with rows as human gene symbols.

## Limitations

- No random seed or reproducibility controls are reported for model training, so exact AUC may vary across runs.
- Only source code is deposited (github.com/xindd/DCNet); the pretrained parameters file net_M434_adam_T600_DCNetBnoise-D1.params is hosted separately in the DCNet-Use repository.
- Features depend on DCNet's embedded set of 434 cell types and 9078 marker genes; cell types absent from that mapping cannot serve as features.
- Classifier performance was validated on TCGA LUAD/LUSC only; transfer to other cohorts or subtypes is untested in this paper.

## Evidence
