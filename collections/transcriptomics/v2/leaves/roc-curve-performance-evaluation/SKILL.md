---
name: roc-curve-performance-evaluation
description: Use when a binary classifier built on DCNet-inferred cell-type abundance
  features (e.g., LUAD vs LUSC prediction) must be validated on a held-out test set
  with ROC AUC, precision, and an ROC curve.
when_to_use_negative:
- The input is single-cell expression data — DCNet deconvolves bulk RNA-seq into cell
  landscapes, so abundance features and downstream subtype classification do not apply
- The goal is survival stratification or consensus clustering of TCGA patients (ConsensusCluster
  / Kaplan-Meier workflow), not subtype classification
- No significantly different cell types were identified between the two classes, leaving
  no DCNet-derived abundance features on which to train the SVM
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: DCNet
  role: Pretrained mxnet deep learning model that infers 434 cell-type abundances
    from bulk RNA-seq, providing the classification features
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Framework in which DCNet is coded and run (adam optimizer, relu, L2loss, lr
    1e-4, 600 iterations, 256-sample batches)
  repo: https://github.com/xindd/DCNet-Use
- name: Python
  role: Language used for both the DCNet model and training/evaluating the SVM classifier
    with ROC and precision metrics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:9137c664d519ed1b1a997fea4ba836c1f789960a71ecc22d17e88b7ff36f1e65
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# roc-curve-performance-evaluation

## Summary

Evaluates a binary subtype classifier — here an SVM trained on DCNet-inferred cell-type abundances that differ significantly between LUAD and LUSC — by splitting patients 80%/20% into train/test sets and reporting ROC AUC (≈0.87) and precision (0.975) on the held-out test set, with an ROC curve plot.

## When to use

Use when a binary classifier built on DCNet-inferred cell-type abundance features (e.g., LUAD vs LUSC prediction) must be validated on a held-out test set with ROC AUC, precision, and an ROC curve.

## When NOT to use

- The input is single-cell expression data — DCNet deconvolves bulk RNA-seq into cell landscapes, so abundance features and downstream subtype classification do not apply
- The goal is survival stratification or consensus clustering of TCGA patients (ConsensusCluster / Kaplan-Meier workflow), not subtype classification
- No significantly different cell types were identified between the two classes, leaving no DCNet-derived abundance features on which to train the SVM

## Inputs

- TCGA LUAD/LUSC bulk RNA-seq gene expression matrix (rows = human gene symbols, columns = samples), log2(x+1) transformed
- DCNet-inferred cell-type abundance table per patient (from index.pkl neuron-to-gene mapping and rix.pkl/relation_matrix.pkl gene-cell correspondence)
- LUAD/LUSC subtype labels for each patient

## Outputs

- ROC curve plot (Figure 6d equivalent)
- ROC AUC on the held-out test set (expected ≈ 0.87)
- SVM precision on the held-out test set (expected 0.975)

## How to apply

Run the pretrained DCNet model (mxnet, adam optimizer, relu activation, L2loss, learning rate 1e-4, 600 iterations, 256 samples per batch) on the log2(x+1)-transformed expression matrix (rows = human gene symbols, columns = samples) to infer cell-type abundances per patient, then identify the cell types whose abundances differ significantly between LUAD and LUSC. Split patients 80%/20% into training and test sets and train an SVM in Python on only the significantly different cell-type abundance features to predict subtype. On the held-out test set compute ROC AUC and precision and plot the ROC curve. Judge success by matching the reported values within tolerance: ROC AUC ≈ 0.87 and SVM precision = 0.975 on the test set.

## Related tools

- **DCNet** (Pretrained mxnet deep learning model that infers 434 cell-type abundances from bulk RNA-seq, providing the classification features) — https://github.com/xindd/DCNet
- **mxnet** (Framework in which DCNet is coded and run (adam optimizer, relu, L2loss, lr 1e-4, 600 iterations, 256-sample batches)) — https://github.com/xindd/DCNet-Use
- **Python** (Language used for both the DCNet model and training/evaluating the SVM classifier with ROC and precision metrics)

## Examples

```
from sklearn.svm import SVC; from sklearn.metrics import roc_auc_score, roc_curve; clf = SVC(); clf.fit(X_train, y_train); auc = roc_auc_score(y_test, clf.decision_function(X_test))
```

## Evaluation signals

- Held-out test-set ROC AUC is approximately 0.87 (reported value)
- SVM precision on the test set equals approximately 0.975 (reported value)
- Train/test split preserves the 80%/20% ratio described in the paper
- Classifier features are restricted to cell types whose DCNet-inferred abundances differ significantly between LUAD and LUSC
- Input expression matrix follows the required orientation (each column a sample, each row a human gene symbol) with log2(x+1) transformation applied before DCNet inference

## Limitations

- No random seed or reproducibility controls for model training are reported, so exact reproduction of AUC/precision may vary
- Trained model weights/checkpoints availability was flagged as uncertain in the article, though the README links DCNet parameters (net_M434_adam_T600_DCNetBnoise-D1.params)
- No software version numbers are reported for mxnet or Python, and training ran on CPU only with no runtime specified

## Evidence
