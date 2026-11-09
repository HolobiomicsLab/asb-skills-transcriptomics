---
name: gene-absence-presence-matrix-modeling
description: Use when you have binary gene cluster absence/presence matrices plus
  literature-based lifestyle labels and need to train lifestyle classifiers or predict
  lifestyles of genomes from species with no representative in the training set.
when_to_use_negative:
- Genomes do not meet the CheckM quality thresholds (>90% completeness, <5% contamination)
  and would inject noise into the feature matrix
- No lifestyle metadata labels are available for any training genomes, so the supervised
  random forest cannot be fit
- The goal is only to detect genes present in >90% of genomes (core-genome analysis)
  rather than to classify lifestyles from cluster distributions
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: scikit-learn
  role: Provides chi2 feature selection (top 1000 features) and the random forest
    classifier used to predict lifestyles from the absence/presence matrix
- name: bacLIFE
  role: Generates the gene cluster absence/presence matrices and lifestyle-prediction
    workflow in which the random forest model is embedded
  repo: https://github.com/Carrion-lab/bacLIFE
- name: Python
  role: Python-based system used to organize and execute the workflow steps including
    the machine learning lifestyle prediction
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:6c0c4345714d9f3c74f4eaab7acb46b909ba5b52d2eed7098c1d6cc90128e1d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - Python
  - bacLIFE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# gene-absence-presence-matrix-modeling

## Summary

Train and validate random forest lifestyle classifiers on gene cluster absence/presence matrices from Burkholderia/Paraburkholderia and Pseudomonas genomes, using chi2 feature selection and tree-vote confidence to predict lifestyles even for species entirely excluded from training.

## When to use

Use when you have binary gene cluster absence/presence matrices plus literature-based lifestyle labels and need to train lifestyle classifiers or predict lifestyles of genomes from species with no representative in the training set.

## When NOT to use

- Genomes do not meet the CheckM quality thresholds (>90% completeness, <5% contamination) and would inject noise into the feature matrix
- No lifestyle metadata labels are available for any training genomes, so the supervised random forest cannot be fit
- The goal is only to detect genes present in >90% of genomes (core-genome analysis) rather than to classify lifestyles from cluster distributions

## Inputs

- Gene cluster absence/presence matrix (binary feature table per genome)
- Literature-based lifestyle metadata labels for training genomes (Supplementary Datasets 10 binary classifiers)
- Absence/presence matrix rows for held-out genomes of the target species

## Outputs

- Per-genome predicted lifestyle labels with tree-vote probability confidence values (e.g., 90% of trees voting for the same class)
- leave_one_species_out_predictions.csv compiling held-out predictions and confidences
- Top 1000 chi2-selected gene cluster features per lifestyle classifier

## How to apply

Build the gene cluster absence/presence matrix with bacLIFE (linclust clustering at >90% identity, 80% coverage, MCL grouping) from genomes passing CheckM quality filters (>90% completeness, <5% contamination), and pair it with lifestyle metadata from literature. Apply chi2 feature selection in scikit-learn to retain the top 1000 gene cluster features, then train a random forest classifier with n_estimators=50 under a 5-fold cross-validation scheme (70% training, 30% test). For leave-one-species-out validation (e.g., B. vietnamiensis, P. fluorescens, P. cichorii), exclude all genomes of the target species from training and predict the held-out genomes. Record the probability confidence as the fraction of forest trees voting for each class, accepting a lifestyle only when confidence exceeds 0.8. Compile per-genome predictions and confidences into a CSV such as leave_one_species_out_predictions.csv.

## Related tools

- **scikit-learn** (Provides chi2 feature selection (top 1000 features) and the random forest classifier used to predict lifestyles from the absence/presence matrix)
- **bacLIFE** (Generates the gene cluster absence/presence matrices and lifestyle-prediction workflow in which the random forest model is embedded) — https://github.com/Carrion-lab/bacLIFE
- **Python** (Python-based system used to organize and execute the workflow steps including the machine learning lifestyle prediction)

## Examples

```
from sklearn.feature_selection import SelectKBest, chi2; from sklearn.ensemble import RandomForestClassifier; X_new = SelectKBest(chi2, k=1000).fit_transform(X, y); rf = RandomForestClassifier(n_estimators=50).fit(X_new, y)
```

## Evaluation signals

- Leave-one-species-out reference counts must match: 40/40 held-out B. vietnamiensis genomes classified as opportunistic human pathogen with 90% of trees voting for that class, 92/124 P. fluorescens predicted environmental, and 30/35 P. cichorii predicted plant pathogen at >=70% tree votes
- Chi2 feature selection must retain exactly the top 1000 gene cluster features per lifestyle classifier
- Probability confidence equals the fraction of the 50 forest trees voting per class, and a lifestyle is only accepted when this value is >0.8
- Output CSV must contain one row per held-out genome with predicted lifestyle and confidence columns

## Limitations

- Prediction quality degrades for species with no training representative: only 92/124 P. fluorescens and 30/35 P. cichorii held-out genomes were recovered at the reported tree-vote thresholds
- Lifestyle labels are literature-based metadata, so classifier performance is bounded by metadata accuracy
- A lifestyle is only assigned when the random forest confidence exceeds 0.8, leaving low-confidence genomes without a lifestyle call

## Evidence

- [methods] The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test): "The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test)"
- [methods] A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8: "A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8"
- [intro] By machine learning of the gene cluster distributions, we first tested the predictability of the different lifestyles and pinpointed the candidate genes: "By machine learning of the gene cluster distributions, we first tested the predictability of the different lifestyles and pinpointed the candidate genes"
- [readme] Employs a machine learning model to forecast bacterial lifestyle or other specified metadata: "Employs a machine learning model to forecast bacterial lifestyle or other specified metadata"
- [methods] cluster all sequences that are >90% identical with a minimum coverage of 80% (parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8): "cluster all sequences that are >90% identical with a minimum coverage of 80% (parameters --min-seq-id 0.9 --cov-mode 0 -c 0.8)"
