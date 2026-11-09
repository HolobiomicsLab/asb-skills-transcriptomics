---
name: train-test-split-stratification
description: Use when a classifier trained on pathway-module-derived features (e.g.,
  GSVA scores of high-weight MOFA pathways) must be assessed for generalization on
  held-out samples and an independent bulk RNA cohort.
when_to_use_negative:
- The study cohort has fewer than 15 samples — scPAFA requires at least 15 samples
  (not cells) for MOFA-based multicellular pathway module identification, so no module
  features would exist to split and classify
- The input is already a fitted model or feature table and the goal is downstream
  interpretation of multicellular pathway modules rather than classifier validation
- Pathway selection is supervised — if pathway features are chosen using the full
  dataset before splitting, the held-out test AUROC is no longer an honest estimate
  of generalization
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn
  role: Provides LinearSVC (L1 regularization, C=0.2), the train/test splitting of
    GSVA scores, and Platt scaling of decision values to probabilities for AUROC computation
- name: GSVA
  role: Computes per-sample pathway variation scores for the 30 high-weight MMR-module
    pathways on GSE39582 and TCGA COAD-READ matrices — the classifier features being
    validated
- name: MOFA
  role: Supplies the factor weights used to select the top-30 pathways from the MMR-related
    multicellular pathway modules before splitting and classification
  repo: https://github.com/bioFAM/mofapy2
- name: scPAFA
  role: Produces the disease-related multicellular pathway modules whose features
    are dissected for classifier training and validation
  repo: https://github.com/ZhuoliHuang/scPAFA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:4984ffc523ce3916590241dee5de91457a0f443b41843c851ef4b8fceb8aacd8
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
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# train-test-split-stratification

## Summary

Strategy for validating disease-status classifiers built from scPAFA/MOFA module features by splitting a labeled bulk RNA cohort (GSE39582) 70/30 into training/test partitions, selecting regularization via 4-fold CV, and confirming on an independent cohort (TCGA COAD-READ).

## When to use

Use when a classifier trained on pathway-module-derived features (e.g., GSVA scores of high-weight MOFA pathways) must be assessed for generalization on held-out samples and an independent bulk RNA cohort.

## When NOT to use

- The study cohort has fewer than 15 samples — scPAFA requires at least 15 samples (not cells) for MOFA-based multicellular pathway module identification, so no module features would exist to split and classify
- The input is already a fitted model or feature table and the goal is downstream interpretation of multicellular pathway modules rather than classifier validation
- Pathway selection is supervised — if pathway features are chosen using the full dataset before splitting, the held-out test AUROC is no longer an honest estimate of generalization

## Inputs

- Bulk RNA-seq expression matrix for GSE39582 colorectal cancer samples with MMR/MSI status labels
- Bulk RNA-seq expression matrix for the independent TCGA COAD-READ cohort with MMR/MSI status labels
- GSVA pathway score matrix for the 30 highest-MOFA-weight pathways from the MMR-related multicellular pathway modules
- Training/test partition metadata (70%/30% split of GSE39582)

## Outputs

- Fitted L1-regularized LinearSVC (C=0.2) with Platt-scaled decision values
- Held-out test-set AUROC for GSE39582 (~0.91, Fig. 3d-g)
- Independent-test AUROC for TCGA COAD-READ (~0.91)
- mmr_signature_auroc_results.csv containing per-cohort AUROC values

## How to apply

Compute GSVA scores for the 30 highest-weight pathways from the MMR-related multicellular pathway modules on the GSE39582 bulk expression matrix, then split GSE39582 samples into 70% training and 30% held-out test partitions stratified by disease/MSI status labels. Train an L1-regularized LinearSVC in scikit-learn, fixing the regularization strength at C=0.2 as selected by 4-fold cross-validation on the training partition. Convert decision values to probabilities via Platt scaling and compute AUROC on the held-out test set. As independent validation, predict on the TCGA COAD-READ expression matrix without refitting, and record both per-cohort AUROC values to mmr_signature_auroc_results.csv. The rationale is that the classifier can only validate the scPAFA/MOFA modules if performance transfers across cohorts rather than overfitting a single dataset.

## Related tools

- **scikit-learn** (Provides LinearSVC (L1 regularization, C=0.2), the train/test splitting of GSVA scores, and Platt scaling of decision values to probabilities for AUROC computation)
- **GSVA** (Computes per-sample pathway variation scores for the 30 high-weight MMR-module pathways on GSE39582 and TCGA COAD-READ matrices — the classifier features being validated)
- **MOFA** (Supplies the factor weights used to select the top-30 pathways from the MMR-related multicellular pathway modules before splitting and classification) — https://github.com/bioFAM/mofapy2
- **scPAFA** (Produces the disease-related multicellular pathway modules whose features are dissected for classifier training and validation) — https://github.com/ZhuoliHuang/scPAFA

## Examples

```
from sklearn.model_selection import train_test_split; X_train, X_test, y_train, y_test = train_test_split(gsva_scores, msi_labels, train_size=0.7, stratify=msi_labels)
```

## Evaluation signals

- The held-out 30% test partition of GSE39582 yields AUROC ≈ 0.91, matching the reported value in Fig. 3d-g
- The independent TCGA COAD-READ AUROC also matches ≈ 0.91 without refitting the model, confirming cross-cohort transfer
- Regularization strength C=0.2 is the value selected by 4-fold cross-validation on the 70% training split, not tuned on test data
- Classifier features are restricted to GSVA scores of exactly the 30 pathways carrying the highest MOFA factor weights from the MMR-related modules
- mmr_signature_auroc_results.csv is written with one AUROC entry per cohort (GSE39582 test, TCGA COAD-READ)

## Limitations

- The scPAFA/MOFA feature pipeline requires studies to include at least 15 samples (not cells), limiting applicability to small cohorts
- Pathway selection is a supervised process; a redundant pathway set increases result complexity, and too few or biologically misaligned pathways cause information loss
- Pseudobulk integration of the PAS matrix leads to information loss because PAS of multiple cells is aggregated, and MOFA is a linear model — interpretability is achieved by sacrificing information content in each factor

## Evidence

- [discussion] These modules are consistent with previous reports in biological interpretation and can further be dissected to extract features for training classifiers.: "These modules are consistent with previous reports in biological interpretation and can further be dissected to extract features for training classifiers."
- [readme] MOFA model is trained to capture variance in PAS among different samples.: "In step 3, MOFA model is trained to capture variance in PAS among different samples."
- [other] Train an L1-regularized LinearSVC (C=0.2, scikit-learn) on the training-split GSVA scores using disease/MSI status labels.: [evidence span withheld — anchor status: not_found]
- [other] achieved AUROC 0.91 on the GSE39582 test set and on the independent TCGA COAD-READ test set, validating the modules: [evidence span withheld — anchor status: not_found]
