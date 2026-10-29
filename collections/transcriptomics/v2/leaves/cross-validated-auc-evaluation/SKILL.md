---
name: cross-validated-auc-evaluation
description: Use when you have a binary gene cluster absence/presence matrix with
  literature-assigned lifestyle labels and need to quantify how predictable each lifestyle
  is via cross-validated AUC.
when_to_use_negative:
- Input is already a per-lifestyle AUC table or the prediction model has already been
  evaluated — no retraining needed
- The goal is experimental validation of individual lifestyle-associated genes (use
  site-directed mutagenesis and plant bioassays instead)
- Genomes lack literature-based lifestyle labels, so no supervised target exists for
  training the binary classifiers
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Chi2 feature selection (top 1000 features) and binary random forest classifier
    training with 5-fold cross-validation and AUC scoring
- name: bacLIFE
  role: Workflow providing the lifestyle prediction module that employs the machine
    learning model on the gene cluster absence/presence data
  repo: https://github.com/Carrion-lab/bacLIFE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:d034cb0ef3765c1c2af36e1af7959fff06d3566d66fc7a34985cf9f42e9a6b94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - bacLIFE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-validated-auc-evaluation

## Summary

Trains binary random forest classifiers (n_estimators=50) on chi2-selected gene cluster absence/presence features to predict bacterial lifestyles, and reports mean AUC over 5-fold cross-validation per lifestyle and genus, as in the bacLIFE paper.

## When to use

Use when you have a binary gene cluster absence/presence matrix with literature-assigned lifestyle labels and need to quantify how predictable each lifestyle is via cross-validated AUC.

## When NOT to use

- Input is already a per-lifestyle AUC table or the prediction model has already been evaluated — no retraining needed
- The goal is experimental validation of individual lifestyle-associated genes (use site-directed mutagenesis and plant bioassays instead)
- Genomes lack literature-based lifestyle labels, so no supervised target exists for training the binary classifiers

## Inputs

- Gene cluster absence/presence matrix (binary features per genome)
- Lifestyle metadata labels per genome (environmental, opportunistic animal pathogen, plant pathogen) assigned by literature review
- Genus grouping of genomes (Burkholderia/Paraburkholderia, Pseudomonas)

## Outputs

- lifestyle_prediction_auc.csv with per-lifestyle, per-genus mean AUC values
- Per-fold AUC values from the 5 cross-validated tests for each lifestyle

## How to apply

Load the gene cluster absence/presence matrix and lifestyle metadata labels (environmental, opportunistic animal pathogen, plant pathogen) assigned by literature review. For each lifestyle, run chi2 feature selection in scikit-learn and retain only the top 1000 important features. Train a binary random forest classifier with n_estimators=50 under a 5-fold cross-validation scheme using a 70% training / 30% test split per fold. Compute the AUC for each of the 5 cross-validated tests and average them to get the mean AUC per lifestyle and genus (Burkholderia/Paraburkholderia and Pseudomonas). High mean AUC indicates gene cluster distributions strongly encode lifestyle, justifying downstream pLAG identification (e.g. >70% group presence with >2 log2 fold change).

## Related tools

- **scikit-learn** (Chi2 feature selection (top 1000 features) and binary random forest classifier training with 5-fold cross-validation and AUC scoring)
- **bacLIFE** (Workflow providing the lifestyle prediction module that employs the machine learning model on the gene cluster absence/presence data) — https://github.com/Carrion-lab/bacLIFE

## Examples

```
from sklearn.ensemble import RandomForestClassifier; from sklearn.feature_selection import SelectKBest, chi2; X_new = SelectKBest(chi2, k=1000).fit_transform(X, y); scores = cross_val_score(RandomForestClassifier(n_estimators=50), X_new, y, cv=5, scoring='roc_auc')
```

## Evaluation signals

- Each Pseudomonas lifestyle AUC falls between 0.96 and 0.99
- Each Burkholderia/Paraburkholderia lifestyle AUC equals 1.00
- Feature selection retains exactly the top 1000 chi2-ranked features before classifier training
- lifestyle_prediction_auc.csv contains one mean AUC per lifestyle per genus, each averaged over 5 cross-validated tests
- Classification confidence values exceeding 0.8 are required for a lifestyle assignment in the downstream prediction module

## Limitations

- AUCs were validated only on Burkholderia/Paraburkholderia and Pseudomonas datasets; performance on other genera is not established by this study
- Labels derive from literature-based metadata assignment, so label noise propagates into both training and AUC evaluation
- Near-perfect AUCs (1.00 for Burkholderia/Paraburkholderia) may reflect strong lifestyle-genome separation and could overstate generalizability to edge-case genomes

## Evidence

- [methods] top 1000 features: "The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test)"
- [readme] ml module readme: "Employs a machine learning model to forecast bacterial lifestyle or other specified metadata"
- [methods] confidence threshold: "A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8"
- [intro] cluster distributions intro: "By machine learning of the gene cluster distributions, we first tested the predictability of the different lifestyles and pinpointed the candidate genes"
