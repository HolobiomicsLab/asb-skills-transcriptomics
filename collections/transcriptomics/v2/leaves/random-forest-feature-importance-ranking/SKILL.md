---
name: random-forest-feature-importance-ranking
description: Use when a small, class-imbalanced labelled gene set (e.g. 136 TSGs vs
  76 OGs) must be ranked by which mutation-type features (entropy, frequency, ratio-metric)
  drive random forest classification, avoiding single-model importance instability.
when_to_use_negative:
- The goal is predicting new driver genes on unlabelled genes rather than interpreting
  which features drive classification — use the consensus prediction pipeline (consolidateModelPredictions.py)
  instead.
- The input data lacks the entropy/frequency features (e.g. Tokheim et al. data where
  missense and frameshift entropy and frequency could not be calculated), so the 37-feature
  ranking cannot be reproduced.
- A background-mutation-rate-based driver prediction (e.g. MutSigCV) is intended;
  this skill addresses feature-based classification, which is explicitly not biased
  towards high mutation-rate genes.
edam_operation: http://edamontology.org/operation_3763
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3474
tools:
- name: cTaG
  role: Pan-cancer random forest framework whose TSG_OG_Feat_final.py loads the feature
    matrices, trains the fold models, and supports the feature-importance ranking
  repo: https://github.com/RamanLab/cTaG
- name: RandomForest
  role: Classifier trained per cross-validation fold; its feature importance measures
    are ranked and averaged to form the consensus ranking
  repo: https://github.com/RamanLab/cTaG
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:11272fc7fb235a5e826307968ae26039e7febb7fe33cc1d8ff83f12703457f14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - RandomForest
  - cTaG
  edam_operation: http://edamontology.org/operation_3763
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# random-forest-feature-importance-ranking

## Summary

Trains a random forest classifier per fivefold cross-validation fold on a 37-feature gene-by-feature matrix (136 TSGs, 76 OGs), extracts per-fold feature importances, converts them to ranks, and averages ranks across folds to obtain a consensus importance ranking of mutation-type features for TSG vs OG classification.

## When to use

Use when a small, class-imbalanced labelled gene set (e.g. 136 TSGs vs 76 OGs) must be ranked by which mutation-type features (entropy, frequency, ratio-metric) drive random forest classification, avoiding single-model importance instability.

## When NOT to use

- The goal is predicting new driver genes on unlabelled genes rather than interpreting which features drive classification — use the consensus prediction pipeline (consolidateModelPredictions.py) instead.
- The input data lacks the entropy/frequency features (e.g. Tokheim et al. data where missense and frameshift entropy and frequency could not be calculated), so the 37-feature ranking cannot be reproduced.
- A background-mutation-rate-based driver prediction (e.g. MutSigCV) is intended; this skill addresses feature-based classification, which is explicitly not biased towards high mutation-rate genes.

## Inputs

- Gene-by-feature matrix with 37 features per gene (FeatureMat.tsv) built from COSMIC (v79) somatic coding mutation data
- CGC-derived class labels: 136 TSGs and 76 OGs used for training (list at RamanLab/IdentifyTSGOG/data/cgc_genes)
- Stable random forest hyper-parameter set estimated via fivefold CV plus multiple random iterations

## Outputs

- Consensus feature importance ranking table (feature_importance_ranking.csv) with average per-feature rank across the five fold models
- Per-fold feature importance ranks for all 37 features
- Trained random forest models for each cross-validation fold (output/RandomForest)

## How to apply

Load the pre-processed gene-by-feature matrix (FeatureMat.tsv, 37 features per gene) together with CGC-derived TSG/OG labels for the 136 TSGs and 76 OGs. Train a random forest classifier for each of the five cross-validation folds, using the stable hyper-parameter set obtained via fivefold cross-validation followed by multiple random iterations to avoid overfitting on the small training set. Extract the feature importance measure from each fold's trained model, convert the 37 importances into per-fold feature ranks (1 = most important), and average each feature's rank across all five fold models. Save the averaged ranking as a consensus table (feature_importance_ranking.csv) and compare feature-value distributions between TSGs and OGs for top features to interpret them biologically. Rationale: averaging per-feature ranks across fold models yields a consensus ranking that is robust to single-model variance, which the paper used to show nonsense mutations are among the most important features.

## Related tools

- **cTaG** (Pan-cancer random forest framework whose TSG_OG_Feat_final.py loads the feature matrices, trains the fold models, and supports the feature-importance ranking) — https://github.com/RamanLab/cTaG
- **RandomForest** (Classifier trained per cross-validation fold; its feature importance measures are ranked and averaged to form the consensus ranking) — https://github.com/RamanLab/cTaG

## Examples

```
python code/TSG_OG_Feat_final.py  # loads data/FeatureMat/FeatureMat.tsv + CGC labels, trains the five CV-fold random forests, and emits per-fold feature importances for rank averaging
```

## Evaluation signals

- Nonsense-, LOF-, frameshift- and missense-related features (e.g. Nonsense entropy, High Frameshift Frequency, High missense frequency) occupy the top ranks in the averaged ranking.
- Exactly 37 features are ranked per fold, and each fold's importance vector converts to a complete permutation of ranks 1–37.
- Top ranks are dominated by LOF- and missense-related features, matching the paper's reported top-18 set (Nonsense entropy, Compound/benign, Damaging/kb, Compound/kB, Damaging/LoFI, HiFI/benign, etc.).
- Number of random iterations is validated via output/evalRandIter to confirm the stable hyper-parameter set was reached before ranking.
- Feature-value distributions of top-ranked features differ visibly between TSG and OG training-set genes.

## Limitations

- Only 136 TSGs and 76 OGs are available for training, so overfitting is a risk; stable hyper-parameters must be estimated via fivefold CV with multiple random iterations before trusting the ranking.
- Features from certain external datasets cannot be reproduced — missense and frameshift entropy/frequency could not be calculated for the Tokheim et al. data, limiting comparisons.
- Averaged feature importance ranks indicate association with the TSG/OG classes, not causal importance; binary classification into TSG/OG is itself a stated limitation of cTaG.

## Evidence

- [results] To identify features important for the classification of TSGs and OGs, we calculated the average rank of each feature across all models.: "To identify features important for the classification of TSGs and OGs, we calculated the average rank of each feature across all models."
- [results] We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation.: "We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation."
- [results] We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting: "We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting"
- [readme] The processed feature matrix can be found [here](https://github.com/RamanLab/IdentifyTSGOG/blob/master/data/FeatureMat/FeatureMat.tsv).: "The processed feature matrix can be found [here](https://github.com/RamanLab/IdentifyTSGOG/blob/master/data/FeatureMat/FeatureMat.tsv)."
- [readme] The genes were labelled as TSG or OG based on labels given by Cancer Gene Census (CGC), and the rest were marked Unlabelled.: "The genes were labelled as TSG or OG based on labels given by Cancer Gene Census (CGC), and the rest were marked Unlabelled."
