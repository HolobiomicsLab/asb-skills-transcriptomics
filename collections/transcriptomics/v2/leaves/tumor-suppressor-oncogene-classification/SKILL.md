---
name: tumor-suppressor-oncogene-classification
description: Use when classifying genes into TSG vs OG classes from a somatic mutation
  feature matrix (e.g. 37-feature COSMIC-derived pan-cancer matrix) with CGC labels,
  and per-class accuracy/precision/recall/F1 metrics are required.
when_to_use_negative:
- The goal is mutation-rate-based driver discovery (e.g. MutSigCV-style background
  mutation rate modelling) rather than feature-based TSG/OG classification — cTaG
  is explicitly designed to avoid mutation-rate bias
- Data are tissue-specific subsets with fewer than 1000 samples; the paper only applies
  tissue-specific analysis to tissues with greater than 1000 samples
- Input is raw somatic variant calls rather than the engineered feature matrix — the
  feature-construction step (identifyTSGOG.py) must run first
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3379
tools:
- name: cTaG
  role: Random-forest pan-cancer pipeline that builds per-fold classifiers, stabilises
    hyper-parameters, and produces consensus TSG/OG predictions
  repo: https://github.com/RamanLab/cTaG
- name: Random Forest
  role: scikit-learn classifier tuned via fivefold cross-validation over n_estimator,
    max_features, max_depth and criterion
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:db11023c61da997471caec9a9a9989af2e144d31128f8ff27e49168e2461951b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - Random Forest
  - cTaG
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3379
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tumor-suppressor-oncogene-classification

## Summary

Train a random-forest classifier (cTaG) that distinguishes tumour suppressor genes (TSGs) from oncogenes (OGs) using 37 ratio-metric and entropy somatic-mutation features derived from COSMIC, with fivefold cross-validation and multi-iteration hyper-parameter stabilisation to avoid overfitting the small labelled gene set.

## When to use

Use when classifying genes into TSG vs OG classes from a somatic mutation feature matrix (e.g. 37-feature COSMIC-derived pan-cancer matrix) with CGC labels, and per-class accuracy/precision/recall/F1 metrics are required.

## When NOT to use

- The goal is mutation-rate-based driver discovery (e.g. MutSigCV-style background mutation rate modelling) rather than feature-based TSG/OG classification — cTaG is explicitly designed to avoid mutation-rate bias
- Data are tissue-specific subsets with fewer than 1000 samples; the paper only applies tissue-specific analysis to tissues with greater than 1000 samples
- Input is raw somatic variant calls rather than the engineered feature matrix — the feature-construction step (identifyTSGOG.py) must run first

## Inputs

- Pre-processed 37-feature pan-cancer gene-by-feature matrix (FeatureMat.tsv) built from COSMIC v79 somatic coding mutations
- CGC-derived TSG/OG label list for training genes (136 TSGs, 76-77 OGs)
- COSMIC mutation-type annotations used to derive entropy and ratio-metric features

## Outputs

- Per-class (TSG, OG) accuracy, precision, recall and F1 on training and test partitions, as average ± standard deviation across folds
- A stable random-forest hyper-parameter set (n_estimator, max_features, max_depth, criterion)
- Consensus TSG/OG classifications with probabilities for previously unlabelled genes

## How to apply

Load the pre-processed 37-feature pan-cancer matrix (FeatureMat.tsv from the RamanLab/IdentifyTSGOG data folder) and attach Cancer Gene Census (CGC)-derived TSG/OG labels to the 136 TSG and 76-77 OG training genes. For each cross-validation fold, train scikit-learn random forest models on a random 80% subset of the labelled genes, holding out the remaining 20% as test. Perform fivefold cross-validation to estimate hyper-parameters (n_estimator, max_features, max_depth, criterion), then repeat over multiple random iterations to identify a stable hyper-parameter set — this is the paper's explicit strategy to overcome overfitting caused by the small training set. Train per-fold models with the stable hyper-parameters and compute per-class accuracy, precision, recall and F1 on both training and test partitions, consolidating into average ± standard deviation summaries. Predictions for unlabelled genes are made by consensus across the per-fold models.

## Related tools

- **cTaG** (Random-forest pan-cancer pipeline that builds per-fold classifiers, stabilises hyper-parameters, and produces consensus TSG/OG predictions) — https://github.com/RamanLab/cTaG
- **Random Forest** (scikit-learn classifier tuned via fivefold cross-validation over n_estimator, max_features, max_depth and criterion)

## Examples

```
python code/TSG_OG_Feat_final.py  # loads FeatureMat.tsv and builds the random-forest TSG/OG models per CV fold
```

## Evaluation signals

- Reproduced test accuracy must match reported values within standard deviations: 0.83 ± 0.02 for TSGs and 0.76 ± 0.03 for OGs
- Per-class test metrics should be near reported OG precision 0.79 ± 0.12 and OG recall 0.50 ± 0.19
- Hyper-parameter estimates should stabilise across multiple random iterations, as verified by the repo's output/evalRandIter checks
- Each cross-validation fold must train on ~80% of the 136 TSGs and 76 OGs with the remaining 20% held out as test
- Known drivers such as ARID1A, TP53 and RB1 should receive high TSG probability in downstream predictions

## Limitations

- Only 136 TSGs and 76-77 OGs are available for training, creating overfitting risk that requires the multi-iteration hyper-parameter stabilisation procedure
- The model uses binary TSG/OG classification, a limitation the authors note in the discussion
- Test-set performance could not be compared with Davoli et al. because their test set results have not been published
- Accuracy is modest for OGs (0.76 ± 0.03) with high OG recall variance (0.50 ± 0.19), so per-fold metrics can vary substantially

## Evidence

- [results] We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation.: "We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation."
- [results] We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting: "We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting"
- [readme] The genes were labelled as TSG or OG based on labels given by Cancer Gene Census (CGC), and the rest were marked Unlabelled.: "The genes were labelled as TSG or OG based on labels given by Cancer Gene Census (CGC), and the rest were marked Unlabelled."
- [abstract] We overcome overfitting and show that certain mutation types, such as nonsense mutations, are more important for classification.: "We overcome overfitting and show that certain mutation types, such as nonsense mutations, are more important for classification."
