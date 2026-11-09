---
name: leave-one-species-out-model-validation
description: Use when evaluating whether a gene-cluster-based lifestyle classifier
  generalizes to a species entirely absent from training, e.g. potential novel species
  with no representative genomes in databases.
when_to_use_negative:
- Input genomes belong to species already represented in the training set — use the
  standard 5-fold cross-validation scheme (70% training, 30% test) instead of species
  exclusion
- Goal is identifying lifestyle-associated genes (pLAGs) rather than testing classifier
  generalization — use the pLAG thresholds (>70% group presence, >2 log2 fold change)
  instead
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: bacLIFE
  role: Generates the gene cluster absence/presence matrices and lifestyle-prediction
    framework being validated
  repo: https://github.com/Carrion-lab/bacLIFE
- name: scikit-learn
  role: Provides chi2 feature selection (top 1000 features) and the random forest
    classifier (n_estimators=50) used for leave-one-species-out prediction
- name: Python
  role: Implementation language for the training, prediction, and confidence-compilation
    steps
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:8541f9aff31b2e66a188f72d0ef5d90573a5ae3643c0c7055c2c63de55657fa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Carrion-lab/bacLIFE
  tools:
  - Python
  - bacLIFE
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# leave-one-species-out-model-validation

## Summary

A generalization test for bacLIFE's random forest lifestyle classifiers in which every genome of one species is excluded from training and the model must predict the lifestyle of that held-out species, mimicking annotation of a novel species absent from reference databases.

## When to use

Use when evaluating whether a gene-cluster-based lifestyle classifier generalizes to a species entirely absent from training, e.g. potential novel species with no representative genomes in databases.

## When NOT to use

- Input genomes belong to species already represented in the training set — use the standard 5-fold cross-validation scheme (70% training, 30% test) instead of species exclusion
- Goal is identifying lifestyle-associated genes (pLAGs) rather than testing classifier generalization — use the pLAG thresholds (>70% group presence, >2 log2 fold change) instead

## Inputs

- Gene cluster absence/presence binary matrix for the Burkholderia/Paraburkholderia dataset
- Gene cluster absence/presence binary matrix for the Pseudomonas dataset
- Literature-based lifestyle metadata labels per genome (e.g. plant pathogen, opportunistic human pathogen, environmental)

## Outputs

- leave_one_species_out_predictions.csv with per-genome predicted lifestyles and tree-vote confidence values
- Per-species validation counts of held-out genomes predicted with their true lifestyle at reported tree-vote thresholds

## How to apply

Load the gene cluster absence/presence matrices together with the literature-based lifestyle metadata for the Burkholderia/Paraburkholderia and Pseudomonas datasets. Apply chi2 feature selection in scikit-learn to retain the top 1000 gene cluster features for each lifestyle classifier, mirroring the paper's 5-fold cross-validation scheme (70% training, 30% test). For each target species (B. vietnamiensis, P. fluorescens, P. cichorii), train a random forest with n_estimators=50 on the training set with all genomes of that species excluded. Predict each held-out genome's lifestyle and record the probability confidence, i.e. the fraction of forest trees voting for each class; the paper selects a lifestyle when confidence exceeds 0.8. Compile the per-genome predictions and confidence values into leave_one_species_out_predictions.csv.

## Related tools

- **bacLIFE** (Generates the gene cluster absence/presence matrices and lifestyle-prediction framework being validated) — https://github.com/Carrion-lab/bacLIFE
- **scikit-learn** (Provides chi2 feature selection (top 1000 features) and the random forest classifier (n_estimators=50) used for leave-one-species-out prediction)
- **Python** (Implementation language for the training, prediction, and confidence-compilation steps)

## Examples

```
from sklearn.feature_selection import SelectKBest, chi2; from sklearn.ensemble import RandomForestClassifier; X_sel = SelectKBest(chi2, k=1000).fit_transform(X_train, y_train); rf = RandomForestClassifier(n_estimators=50).fit(X_sel, y_train)
```

## Evaluation signals

- All 40 held-out B. vietnamiensis genomes must be predicted as opportunistic human pathogens with 90% of trees voting for the same class (40/40)
- 92 of 124 held-out P. fluorescens genomes must be predicted with their true environmental lifestyle
- 30 of 35 held-out P. cichorii genomes must be predicted as plant pathogens at >=70% tree votes
- Per-genome confidence values in leave_one_species_out_predictions.csv must match the counts reported in Supplementary Datasets 2

## Limitations

- Performance is species-dependent: only 92/124 P. fluorescens and 30/35 P. cichorii genomes recovered their true lifestyle, so held-out species predictions are imperfect even at moderate tree-vote thresholds
- The lifestyle selection criterion relies on a probability confidence threshold (>0.8 tree-vote fraction), below which no lifestyle is assigned
- Validation is limited to three target species from two genera (Burkholderia/Paraburkholderia and Pseudomonas) with literature-based lifestyle labels

## Evidence

- [methods] The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test): "The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test)"
- [methods] A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8: "A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8"
- [other] the random forest model trained without any B. vietnamiensis genome classified all 40 held-out B. vietnamiensis genomes as opportunistic human pathogens, with 90% of the trees voting: [evidence span withheld — anchor status: not_found]
