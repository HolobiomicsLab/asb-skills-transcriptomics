---
name: independent-cohort-validation
description: Use when a classifier trained on one cohort's features must be shown
  to generalize — e.g., an L1 LinearSVC on GSVA scores of MMR-module pathways needs
  AUROC confirmation on both a 30% held-out GSE39582 split and the independent TCGA
  COAD-READ cohort.
when_to_use_negative:
- Only one cohort exists and no external expression matrix is available — a cross-validation
  AUROC alone does not constitute independent-cohort validation
- The feature pipeline (pathway set, GSVA parameters, preprocessing) differs between
  the training and validation cohorts, breaking feature comparability
- The goal is to retrain or refit the model on the second cohort — that is transfer
  learning, not independent validation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: scikit-learn
  role: Provides the L1-regularized LinearSVC (C=0.2) and Platt-scaling conversion
    of decision values to probabilities for prediction on the independent cohort
- name: GSVA
  role: Computes the pathway-score features on the independent TCGA COAD-READ expression
    matrix using the same 30 high-weight pathways as in training
- name: MOFA
  role: Supplies the factor weights used to select the top 30 pathways whose GSVA
    scores are the validation features
  repo: https://github.com/bioFAM/mofapy2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:24cdf1b480114c16b57ee0c3a7a8fc6f1ff6d76e1237418b68632cb58e1cb279
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioFAM/mofapy2
  tools:
  - GSVA
  - MOFA
  - Python
  - scPAFA
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# independent-cohort-validation

## Summary

Validates a trained prediction model by testing it on a completely independent cohort (e.g., TCGA COAD-READ) in addition to a held-out split of the training cohort (e.g., GSE39582), converting decision values to probabilities and computing per-cohort AUROC.

## When to use

Use when a classifier trained on one cohort's features must be shown to generalize — e.g., an L1 LinearSVC on GSVA scores of MMR-module pathways needs AUROC confirmation on both a 30% held-out GSE39582 split and the independent TCGA COAD-READ cohort.

## When NOT to use

- Only one cohort exists and no external expression matrix is available — a cross-validation AUROC alone does not constitute independent-cohort validation
- The feature pipeline (pathway set, GSVA parameters, preprocessing) differs between the training and validation cohorts, breaking feature comparability
- The goal is to retrain or refit the model on the second cohort — that is transfer learning, not independent validation

## Inputs

- Fitted classifier (L1-regularized LinearSVC, C=0.2) trained on 70% GSE39582 GSVA-score matrix
- GSVA pathway-score matrix for the same 30 high-weight pathways computed on the independent TCGA COAD-READ bulk RNA expression matrix
- Held-out 30% GSE39582 test-split GSVA scores
- Disease/MSI status labels for each cohort

## Outputs

- Per-cohort AUROC values (held-out GSE39582 test AUROC and TCGA COAD-READ independent-test AUROC)
- Results table, e.g., mmr_signature_auroc_results.csv

## How to apply

First train the model only on the training partition (e.g., 70% of GSE39582 with an L1-regularized LinearSVC, C=0.2 selected by 4-fold CV) and confirm performance on the 30% held-out test split. Then apply the identical fitted pipeline — without any retraining or threshold refitting — to the expression matrix of a second, fully independent cohort (TCGA COAD-READ) after computing the same features (GSVA scores of the 30 highest-weight MOFA pathways). Convert the SVC's raw decision values to probabilities via Platt scaling so AUROC is comparable across cohorts. Compute AUROC separately for each cohort (held-out test set and independent cohort) and require both to reproduce the reported ~0.91 (Fig. 3d-g) before writing results, e.g., to mmr_signature_auroc_results.csv. Matching performance on the external cohort is what validates that the scPAFA-identified multicellular pathway modules carry a generalizable disease signature rather than cohort-specific signal.

## Related tools

- **scikit-learn** (Provides the L1-regularized LinearSVC (C=0.2) and Platt-scaling conversion of decision values to probabilities for prediction on the independent cohort)
- **GSVA** (Computes the pathway-score features on the independent TCGA COAD-READ expression matrix using the same 30 high-weight pathways as in training)
- **MOFA** (Supplies the factor weights used to select the top 30 pathways whose GSVA scores are the validation features) — https://github.com/bioFAM/mofapy2

## Examples

```
from sklearn.svm import LinearSVC; from sklearn.calibration import CalibratedClassifierCV; clf = CalibratedClassifierCV(LinearSVC(C=0.2, penalty='l1', dual=False), method='sigmoid').fit(X_train_gsva, y_train); proba_tcga = clf.predict_proba(X_tcga_gsva)[:,1]
```

## Evaluation signals

- Held-out GSE39582 test-split AUROC matches the reported ~0.91 from Fig. 3d-g
- Independent TCGA COAD-READ test AUROC also matches the reported ~0.91
- The exact same 30-pathway feature set and fitted model are applied to the external cohort with no retraining or parameter re-tuning
- Per-cohort AUROC values are written to mmr_signature_auroc_results.csv with one row per cohort
- SVC decision values are converted to probabilities (Platt scaling) before AUROC computation

## Limitations

- Validation is cohort-dependent: a signature matching GSE39582 and TCGA COAD-READ may still fail on cohorts with different platforms, batch effects, or case mixes
- Matching AUROC on two cohorts does not establish biological causality — the pathway-module associations are data-driven, not causal
- The pathway features are supervised selections; if the pathway set is redundant or misaligned with biology, apparent generalization may be unstable

## Evidence

- [methods] Split GSE39582 samples into 70% training and 30% test partitions.: [evidence span withheld — anchor status: not_found]
- [results] Independent TCGA validation of AUROC 0.91: [evidence span withheld — anchor status: not_found]
- [discussion] Dissect modules to extract features for training classifiers: "These modules are consistent with previous reports in biological interpretation and can further be dissected to extract features for training classifiers."
- [full_text] Taskcard workflow: train LinearSVC and predict on TCGA: [evidence span withheld — anchor status: not_found]
- [full_text] Feature computation on both cohorts: [evidence span withheld — anchor status: not_found]
