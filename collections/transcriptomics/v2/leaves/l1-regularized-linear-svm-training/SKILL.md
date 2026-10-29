---
name: l1-regularized-linear-svm-training
description: 'Use when you need to validate single-cell-derived pathway modules as
  biomarkers: a small set of high-weight pathways must be converted into predictive
  features on independent bulk RNA-seq cohorts with binary disease labels.'
when_to_use_negative:
- Input is already a per-sample feature/score matrix from a validated signature and
  no module validation is needed — a simpler regression or published signature score
  suffices
- The dataset has fewer than ~15 samples, since the upstream scPAFA/MOFA module identification
  that produces the feature set requires at least 15 samples
- Labels are multi-class or multiple disease states, where the article notes interpretability
  may decrease
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: scikit-learn
  role: Provides the L1-regularized LinearSVC used to train the MMR-status classifier
    on GSVA pathway scores
- name: GSVA
  role: Computes per-sample pathway variation scores for the 30 high-weight module
    pathways on bulk expression matrices
- name: MOFA
  role: Source of the factor weights used to rank and select the top pathways that
    become classifier features
  repo: https://github.com/bioFAM/mofapy2
- name: scPAFA
  role: Upstream method whose MMR-related multicellular pathway modules supply the
    high-weight pathways for classifier training
  repo: https://github.com/ZhuoliHuang/scPAFA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-024-07238-7
    title: Uncovering disease-related multicellular pathway modules on large-scale
      single-cell transcriptomes with scPAFA
  source_content_hash: sha256:f2bfc3a4c9615d5dd1c8720e9569b2f6d4027cbb82fd936c5d9143fb5b238514
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# l1-regularized-linear-svm-training

## Summary

Trains an L1-regularized LinearSVC classifier on GSVA pathway scores of high-weight MOFA-module pathways to predict MMR/MSI status in bulk CRC RNA data, validating scPAFA-derived multicellular pathway modules.

## When to use

Use when you need to validate single-cell-derived pathway modules as biomarkers: a small set of high-weight pathways must be converted into predictive features on independent bulk RNA-seq cohorts with binary disease labels.

## When NOT to use

- Input is already a per-sample feature/score matrix from a validated signature and no module validation is needed — a simpler regression or published signature score suffices
- The dataset has fewer than ~15 samples, since the upstream scPAFA/MOFA module identification that produces the feature set requires at least 15 samples
- Labels are multi-class or multiple disease states, where the article notes interpretability may decrease

## Inputs

- Bulk RNA-seq expression matrix for GSE39582 with MMR/MSI status labels
- Bulk RNA-seq expression matrix for TCGA COAD-READ
- List of ~30 high-weight pathways (top MOFA factor weights) from MMR-related multicellular pathway modules

## Outputs

- Fitted L1-regularized LinearSVC model (C=0.2) on GSVA pathway scores
- Per-cohort AUROC values written to mmr_signature_auroc_results.csv
- Platt-scaled probability predictions for the 30% held-out GSE39582 test set and TCGA COAD-READ

## How to apply

Extract the top ~30 pathways carrying the highest MOFA factor weights from the scPAFA-identified MMR-related multicellular pathway modules. Compute GSVA (Gene Set Variation Analysis) scores for these pathways on the bulk expression matrices (GSE39582 and TCGA COAD-READ). Split GSE39582 into 70% training / 30% test partitions, then train a scikit-learn L1-regularized LinearSVC with C=0.2, chosen by 4-fold cross-validation, using MMR/MSI status labels. Convert decision values to probabilities via Platt scaling and compute AUROC on the held-out test set and on TCGA COAD-READ as an independent external validation. The L1 penalty performs embedded feature selection, keeping the classifier sparse and aligned with the small module-derived feature set.

## Related tools

- **scikit-learn** (Provides the L1-regularized LinearSVC used to train the MMR-status classifier on GSVA pathway scores)
- **GSVA** (Computes per-sample pathway variation scores for the 30 high-weight module pathways on bulk expression matrices)
- **MOFA** (Source of the factor weights used to rank and select the top pathways that become classifier features) — https://github.com/bioFAM/mofapy2
- **scPAFA** (Upstream method whose MMR-related multicellular pathway modules supply the high-weight pathways for classifier training) — https://github.com/ZhuoliHuang/scPAFA

## Examples

```
from sklearn.svm import LinearSVC; clf = LinearSVC(penalty='l1', C=0.2, dual=False); clf.fit(X_train_gsva, y_train); probs = _sigmoid(clf.decision_function(X_test_gsva))
```

## Evaluation signals

- Held-out 30% GSE39582 test AUROC must match the reported ~0.91 shown in Fig. 3d-g
- Independent TCGA COAD-READ test AUROC must also match the reported ~0.91
- Best C value from 4-fold CV should select C=0.2; large deviations indicate feature-set or label problems
- mmr_signature_auroc_results.csv must contain one AUROC per cohort with both values near 0.91
- Feature matrix dimensionality should equal the number of extracted high-weight pathways (30) per sample

## Limitations

- Module pathways and cell types within a module are data-driven associations, not biological causal effects, so classifier features are correlational
- Pseudobulk aggregation of the PAS matrix upstream leads to information loss since PAS of multiple cells is aggregated
- MOFA is a linear model; interpretability is achieved by sacrificing information content in each factor
- The upstream feature extraction requires at least 15 samples (not cells) for MOFA-based module identification

## Evidence

- [results] An L1-regularized SVC classifier built on GSVA scores of high-weight MMR-module pathways (C=0.2 from 4-fold CV) achieved AUROC 0.91: [evidence span withheld — anchor status: not_found]
- [discussion] modules can be dissected to extract features for training classifiers: "These modules are consistent with previous reports in biological interpretation and can further be dissected to extract features for training classifiers."
- [methods] MOFA model is trained to capture variance in PAS among different samples: "MOFA model is trained to capture variance in PAS among different samples"
- [discussion] scPAFA requires studies to include at least 15 samples: "Most importantly, scPAFA requires studies to include at least 15 samples (not cells), which is the requirement of using the MOFA model for identifying multicellular pathway modules."
- [readme] Step4 downstream analysis identifies multicellular pathway modules with pathway weights across cell types: "disease-related multicellular pathway modules (latent factor and its corresponding weights of pathways across cell types) can be identified by statistical analysis"
