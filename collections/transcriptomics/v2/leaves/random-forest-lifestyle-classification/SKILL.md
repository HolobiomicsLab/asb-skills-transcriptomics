---
name: random-forest-lifestyle-classification
description: Use when gene cluster absence/presence matrices paired with literature-based
  lifestyle metadata are available and the goal is to classify held-out genomes' lifestyle,
  including leave-one-species-out validation for novel species.
when_to_use_negative:
- Input is raw genome assemblies or annotations rather than a gene cluster absence/presence
  matrix — the clustering module must run first
- The target lifestyle has too few labeled genomes to train a binary classifier reliably
- The prediction target is identification of lifestyle-associated genes (pLAGs) rather
  than lifestyle classification of genomes
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: bacLIFE
  role: Workflow that generates the gene cluster absence/presence matrices consumed
    by the lifestyle-prediction machine learning module
  repo: https://github.com/Carrion-lab/bacLIFE
- name: Python
  role: Language for the Snakemake-based module organizing and executing the classification
    pipeline rules
- name: scikit-learn
  role: Provides chi2 feature selection (SelectKBest top 1000 features) and the random
    forest classifier (n_estimators=50)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-46302-y
    title: 'bacLIFE: a user-friendly computational workflow for genome analysis and
      prediction of lifestyle-associated genes in bacteria'
  source_content_hash: sha256:63f050b807899cff4085b177f0408c9f0be9857940cc40d2b3a38fe2e18cc58a
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# random-forest-lifestyle-classification

## Summary

Train scikit-learn random forest classifiers on gene cluster absence/presence matrices to predict bacterial lifestyle (e.g., plant pathogen, environmental, opportunistic human pathogen), as done in bacLIFE for Burkholderia/Paraburkholderia and Pseudomonas genomes.

## When to use

Use when gene cluster absence/presence matrices paired with literature-based lifestyle metadata are available and the goal is to classify held-out genomes' lifestyle, including leave-one-species-out validation for novel species.

## When NOT to use

- Input is raw genome assemblies or annotations rather than a gene cluster absence/presence matrix — the clustering module must run first
- The target lifestyle has too few labeled genomes to train a binary classifier reliably
- The prediction target is identification of lifestyle-associated genes (pLAGs) rather than lifestyle classification of genomes

## Inputs

- Gene cluster absence/presence matrix (binary features per genome)
- Literature-based lifestyle metadata labels for each genome
- List of genomes of the held-out target species (e.g., B. vietnamiensis, P. fluorescens, P. cichorii)

## Outputs

- Per-genome predicted lifestyle labels with tree-vote probability confidence values (leave_one_species_out_predictions.csv)
- Trained random forest classifier per lifestyle (n_estimators=50, chi2 top-1000 features)
- Counts of held-out genomes predicted with their true lifestyle at reported tree-vote thresholds

## How to apply

Load the binary gene cluster absence/presence matrix together with lifestyle labels derived from literature metadata for the Burkholderia/Paraburkholderia and Pseudomonas datasets. Apply chi2 feature selection in scikit-learn to retain the top 1000 gene cluster features per lifestyle classifier, then train a random forest with n_estimators=50; the article evaluates features via a 5-fold cross-validation scheme (70% training, 30% test). For leave-one-species-out validation, exclude all genomes of the target species (e.g., B. vietnamiensis, P. fluorescens, P. cichorii) from training and predict the held-out genomes. Record the probability confidence, i.e., the fraction of forest trees voting for each class, and assign a lifestyle when confidence exceeds 0.8 (>=0.70 tree votes is used as a reporting threshold in the validation counts). Compile per-genome predictions and confidences into leave_one_species_out_predictions.csv.

## Related tools

- **bacLIFE** (Workflow that generates the gene cluster absence/presence matrices consumed by the lifestyle-prediction machine learning module) — https://github.com/Carrion-lab/bacLIFE
- **Python** (Language for the Snakemake-based module organizing and executing the classification pipeline rules)
- **scikit-learn** (Provides chi2 feature selection (SelectKBest top 1000 features) and the random forest classifier (n_estimators=50))

## Examples

```
from sklearn.feature_selection import SelectKBest, chi2; from sklearn.ensemble import RandomForestClassifier; X_sel = SelectKBest(chi2, k=1000).fit_transform(X, y); clf = RandomForestClassifier(n_estimators=50).fit(X_sel, y)
```

## Evaluation signals

- All 40 held-out B. vietnamiensis genomes classified as opportunistic human pathogen with 90% tree votes
- 92/124 held-out P. fluorescens genomes predicted as environmental lifestyle
- 30/35 held-out P. cichorii genomes predicted as plant pathogen at >=70% tree votes
- Assigned lifestyles must meet the >0.8 probability confidence threshold; confidences match Supplementary Datasets 2
- Predictions and confidence values are compiled into leave_one_species_out_predictions.csv with per-genome rows

## Limitations

- Held-out genomes are only assigned a lifestyle when tree-vote confidence exceeds 0.8, so low-confidence genomes remain unclassified
- Leave-one-species-out performance varies by species (e.g., only 92/124 P. fluorescens genomes recovered at the reported threshold)
- Model is trained on literature-based lifestyle labels for Burkholderia/Paraburkholderia and Pseudomonas; transfer to other genera is unvalidated in this context

## Evidence

- [methods] chi2 top-1000 selection and CV design: "The top 1000 important features are selected and used in a 5-fold cross-validation scheme (70% training, 30% test)"
- [methods] confidence threshold for lifestyle assignment: "A lifestyle is selected if the random forest adjudication has a probability confidence value of >0.8"
- [results] leave-one-species-out B. vietnamiensis result: [evidence span withheld — anchor status: not_found]
- [intro] machine learning on gene cluster distributions: "By machine learning of the gene cluster distributions, we first tested the predictability of the different lifestyles and pinpointed the candidate genes"
