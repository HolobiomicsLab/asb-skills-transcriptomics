---
name: gene-presence-absence-feature-selection
description: Use when building lifestyle or metadata classifiers from a binary gene
  cluster presence/absence matrix (e.
when_to_use_negative:
- Input is not a binary gene cluster absence/presence matrix (e.g. raw gene sequences
  or annotated GFF files) — clustering into gene clusters must happen first in bacLIFE's
  clustering module
- Lifestyle labels are not literature-derived per-genome metadata, so there is no
  target variable for chi2 scoring and supervised selection
- The goal is de novo gene clustering or genome annotation rather than lifestyle prediction
  from an existing cluster matrix
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Provides chi2 feature selection (SelectKBest-style top-1000 retention) and
    the binary random forest classifiers (n_estimators = 50) used in the 5-fold cross-validation
    scheme
- name: bacLIFE
  role: Generates the gene cluster absence/presence matrix and lifestyle-prediction
    module that consumes the selected features
  repo: https://github.com/Carrion-lab/bacLIFE
- name: Python
  role: Host language for the lifestyle prediction module and chi2-based feature selection
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:c43f4aecff33b186ab7172f38a9d3f4ef6c0a0ea04955b5fee8358f8048b7387
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - Python
  - bacLIFE
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# gene-presence-absence-feature-selection

## Summary

Reduces a high-dimensional gene cluster absence/presence matrix (thousands of MCL-derived gene clusters per genome) to the top 1000 most informative gene-cluster features via chi2 feature selection before training binary random forest lifestyle classifiers. This is the feature-reduction step that makes bacLIFE's per-lifestyle lifestyle prediction tractable and performant.

## When to use

Use when building lifestyle or metadata classifiers from a binary gene cluster presence/absence matrix (e.g. Burkholderia/Paraburkholderia or Pseudomonas genomes) that has far more gene-cluster columns than needed, and you must select the most informative features before cross-validated random forest prediction.

## When NOT to use

- Input is not a binary gene cluster absence/presence matrix (e.g. raw gene sequences or annotated GFF files) — clustering into gene clusters must happen first in bacLIFE's clustering module
- Lifestyle labels are not literature-derived per-genome metadata, so there is no target variable for chi2 scoring and supervised selection
- The goal is de novo gene clustering or genome annotation rather than lifestyle prediction from an existing cluster matrix

## Inputs

- Gene cluster absence/presence matrix (binary feature matrix, genomes x MCL gene clusters) from bacLIFE
- Literature-derived lifestyle metadata labels per genome (e.g. environmental, opportunistic animal pathogen, plant pathogen, unknown)
- Trained binary random forest classifiers (scikit-learn, n_estimators = 50, one per lifestyle) for prediction on unlabeled genomes

## Outputs

- Reduced feature vector per genome restricted to the top 1000 chi2-selected gene-cluster features
- Per-lifestyle, per-genus mean cross-validated AUC values (e.g. lifestyle_prediction_auc.csv)
- Augmented lifestyle metadata table (TSV) with genome identifier, predicted lifestyle (or unknown), and confidence values for all genomes

## How to apply

Start from the gene cluster absence/presence matrix produced by bacLIFE's clustering module (linclust + MCL gene clusters over annotated genomes) together with literature-derived lifestyle labels (e.g. environmental, opportunistic animal pathogen, plant pathogen). Apply chi2 feature selection in scikit-learn to rank gene-cluster features by their association with each lifestyle and retain only the top 1000 important features. Train a binary random forest classifier per lifestyle (scikit-learn, n_estimators = 50) on the reduced feature vector in a 5-fold cross-validation scheme (70% training, 30% test). For prediction on genomes with unknown lifestyle, apply the same chi2-selected feature space and assign a lifestyle only when the random forest probability confidence exceeds 0.8, leaving genomes below the threshold unassigned. The rationale is that chi2 scoring on binary (Bernoulli) gene-cluster features isolates the clades' lifestyle-discriminative gene content and improves cross-validated AUC (0.96–0.99 for Pseudomonas lifestyles; 1.00 for Burkholderia/Paraburkholderia).

## Related tools

- **scikit-learn** (Provides chi2 feature selection (SelectKBest-style top-1000 retention) and the binary random forest classifiers (n_estimators = 50) used in the 5-fold cross-validation scheme)
- **bacLIFE** (Generates the gene cluster absence/presence matrix and lifestyle-prediction module that consumes the selected features) — https://github.com/Carrion-lab/bacLIFE
- **Python** (Host language for the lifestyle prediction module and chi2-based feature selection)

## Examples

```
from sklearn.feature_selection import SelectKBest, chi2; X_sel = SelectKBest(chi2, k=1000).fit_transform(X_lifestyle_binary_matrix, y)
```

## Evaluation signals

- Each Pseudomonas lifestyle cross-validated AUC falls between 0.96 and 0.99, and each Burkholderia/Paraburkholderia lifestyle AUC equals 1.00, matching Supplementary Fig. 2
- Feature vector length per genome is restricted to exactly 1000 gene-cluster features after chi2 selection
- Lifestyle assignment rate on unknown-lifestyle genomes is within tolerance of 137/145 (94.5%) for Burkholderia/Paraburkholderia and 740/976 (75.9%) for Pseudomonas when using the >0.8 confidence threshold
- Output TSV contains genome identifier, predicted-lifestyle, and confidence-value columns, with unassigned genomes showing intermediate split scores (e.g. 0.668 vs 0.368) rather than a confident call
- Prediction confidence values for all genomes are recorded (comparable to Supplementary Datasets 2)

## Limitations

- Genomes whose random forest adjudication confidence falls below 0.8 remain unassigned; ambiguous cases can show intermediate probabilities (e.g. 0.668 vs 0.368) that must not be forced into a call
- The chi2-selected top-1000 feature space is lifestyle-specific and tied to the reference gene clusters; transferring to new genera requires retraining and reselecting features
- Lifestyle labels are literature-derived metadata, so label quality bounds prediction quality; binary classifiers handle one lifestyle at a time

## Evidence

- [methods] chi2 selection retains top 1000 features: "The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test)"
- [methods] confidence threshold for lifestyle assignment: "A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8"
- [intro] motivation: machine learning on gene cluster distributions: "By machine learning of the gene cluster distributions, we first tested the predictability of the different lifestyles and pinpointed the candidate genes"
- [readme] clustering module produces the feature matrix: "Predicts, clusters and annotates the genes of every input genome"
