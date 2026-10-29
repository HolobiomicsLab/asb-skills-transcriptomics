---
name: binary-random-forest-classifier-training
description: Use when you have a binary gene cluster absence/presence matrix plus
  lifestyle metadata labels per genome and want to quantify lifestyle predictability
  (AUC) or assign lifestyles to unlabelled genomes.
when_to_use_negative:
- Input is already a table of lifestyle labels or confidence values rather than a
  binary gene cluster absence/presence feature matrix
- The goal is de novo genome annotation or gene clustering — use bacLIFE's clustering
  module (Prokka, linclust, MCL) first before classification
- Genomes lack literature-derived lifestyle labels and no supervised training set
  can be assembled
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: chi2 feature selection (top 1000 features) and random forest classifier training
    with 5-fold cross-validation and AUC scoring
- name: bacLIFE
  role: workflow wrapper whose lifestyle-prediction module employs the machine learning
    model to forecast bacterial lifestyle from gene cluster distributions
  repo: https://github.com/Carrion-lab/bacLIFE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:207e4f8fa8e4f2474690949b4577d0414f0f321fa27509d65f66047cb32ae6a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - bacLIFE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Binary random forest classifier training for lifestyle prediction

## Summary

Train per-lifestyle binary random forest classifiers on gene cluster absence/presence matrices to test how predictable bacterial lifestyles (environmental, opportunistic animal pathogen, plant pathogen) are, as done in bacLIFE for Burkholderia/Paraburkholderia and Pseudomonas genomes.

## When to use

Use when you have a binary gene cluster absence/presence matrix plus lifestyle metadata labels per genome and want to quantify lifestyle predictability (AUC) or assign lifestyles to unlabelled genomes.

## When NOT to use

- Input is already a table of lifestyle labels or confidence values rather than a binary gene cluster absence/presence feature matrix
- The goal is de novo genome annotation or gene clustering — use bacLIFE's clustering module (Prokka, linclust, MCL) first before classification
- Genomes lack literature-derived lifestyle labels and no supervised training set can be assembled

## Inputs

- Gene cluster absence/presence matrix (binary features per genome, from linclust+MCL clustering)
- Lifestyle metadata labels per genome (environmental, opportunistic animal pathogen, plant pathogen) assigned by literature review
- Genus grouping (Burkholderia/Paraburkholderia vs Pseudomonas) for per-genus evaluation

## Outputs

- Per-lifestyle, per-genus mean cross-validated AUC values (e.g. lifestyle_prediction_auc.csv)
- Trained binary random forest classifiers per lifestyle with predicted probability/confidence values per genome

## How to apply

Load the gene cluster absence/presence matrix and the lifestyle labels assigned by literature review for each genome. For each lifestyle, run chi2 feature selection in scikit-learn and retain the top 1000 important features, then train a binary random forest classifier with n_estimators = 50 under a 5-fold cross-validation scheme (70% training, 30% test). Compute the AUC for each of the 5 cross-validated tests and average them to obtain a mean AUC per lifestyle and genus. For assigning lifestyles to new genomes, select a lifestyle only if the random forest adjudication has a probability confidence value >0.8. The high AUCs justify downstream use of the same feature space to pinpoint candidate lifestyle-associated genes.

## Related tools

- **scikit-learn** (chi2 feature selection (top 1000 features) and random forest classifier training with 5-fold cross-validation and AUC scoring)
- **bacLIFE** (workflow wrapper whose lifestyle-prediction module employs the machine learning model to forecast bacterial lifestyle from gene cluster distributions) — https://github.com/Carrion-lab/bacLIFE

## Examples

```
from sklearn.ensemble import RandomForestClassifier; from sklearn.feature_selection import SelectKBest, chi2; X_new = SelectKBest(chi2, k=1000).fit_transform(X, y); clf = RandomForestClassifier(n_estimators=50).fit(X_new, y)
```

## Evaluation signals

- Each Pseudomonas lifestyle mean AUC must fall between 0.96 and 0.99 (environmental, opportunistic animal pathogen, plant pathogen)
- Each Burkholderia/Paraburkholderia lifestyle mean AUC must equal 1.00
- Feature selection retains exactly the top 1000 features per lifestyle by chi2 score before training
- Cross-validation follows the 5-fold, 70% training / 30% test scheme, and the reported value is the mean of the 5 per-fold AUCs
- Assigned lifestyles for new genomes carry a random forest probability confidence >0.8; genomes below this threshold receive no lifestyle call

## Limitations

- Near-perfect AUCs reflect within-genus predictability on literature-curated labels; generalization to other genera or label sources is not demonstrated here
- Binary classifiers are trained separately per lifestyle, so multi-class assignment requires adjudication of per-lifestyle probabilities with the >0.8 confidence cutoff
- Features are limited to the top 1000 chi2-selected gene clusters, so rare discriminative genes outside this set are not considered

## Evidence

- [methods] The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test): "The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test)"
- [methods] confidence threshold for lifestyle assignment: "A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8"
- [methods] supplementary dataset of lifestyle labels: "binary classifiers for each lifestyle assigned based on metadata from literature (Supplementary Datasets 10)"
- [readme] README lifestyle prediction module: "Employs a machine learning model to forecast bacterial lifestyle or other specified metadata"
