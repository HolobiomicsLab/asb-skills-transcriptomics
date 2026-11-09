---
name: cancer-driver-gene-classification
description: Use when somatic mutation data (e.g. COSMIC MAF-style coding mutations)
  must be converted into per-gene mutation-type features and genes classified as TSG
  vs OG, especially to nominate low-frequency candidate drivers among unlabelled genes
  without background-mutation-rate bias.
when_to_use_negative:
- The input is already a processed feature table or you only need mutation-rate-based
  driver detection (MutSigCV-style background-rate modelling) rather than TSG/OG functional
  classification
- You need tissue-specific predictions from raw per-tissue mutation subsets — use
  the tissue-specific variant workflow (TSG_OG_tissueAnalysis.py) on tissues with
  greater than 1000 samples instead of the pan-cancer model
- Missense/frameshift entropy and frequency features cannot be computed from your
  mutation data (as with Tokheim et al. data), since the classifier depends on these
  37 features
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: cTaG
  role: Random forest pan-cancer classifier that builds per-fold TSG/OG models on
    37 mutation-type features and predicts new driver genes from unlabelled genes
  repo: https://github.com/RamanLab/cTaG
- name: RandomForest
  role: Classification algorithm trained on 80% of 136 TSGs and 76 OGs per CV fold
    with hyper-parameters stabilised over multiple random iterations
  repo: https://github.com/RamanLab/cTaG
- name: MutSigCV
  role: Used to calculate mutation rates for post-hoc analysis of mutation frequencies
    of cTaG-predicted genes (not part of classification itself)
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:5fad9624438be0d18d30649c19208255e3ce39450c9a9f6bfb8c301a09c8bf04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - MutSigCV
  - RandomForest
  - cTaG
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cancer-driver-gene-classification

## Summary

A pan-cancer random forest workflow (cTaG) that classifies genes as tumour suppressor genes (TSGs) or oncogenes (OGs) using 37 ratio-metric and entropy features derived from COSMIC somatic mutation data, and uses consensus across cross-validation fold models to predict new driver genes among unlabelled genes.

## When to use

Use when somatic mutation data (e.g. COSMIC MAF-style coding mutations) must be converted into per-gene mutation-type features and genes classified as TSG vs OG, especially to nominate low-frequency candidate drivers among unlabelled genes without background-mutation-rate bias.

## When NOT to use

- The input is already a processed feature table or you only need mutation-rate-based driver detection (MutSigCV-style background-rate modelling) rather than TSG/OG functional classification
- You need tissue-specific predictions from raw per-tissue mutation subsets — use the tissue-specific variant workflow (TSG_OG_tissueAnalysis.py) on tissues with greater than 1000 samples instead of the pan-cancer model
- Missense/frameshift entropy and frequency features cannot be computed from your mutation data (as with Tokheim et al. data), since the classifier depends on these 37 features

## Inputs

- COSMIC (v79) somatic coding mutation data filtered to samples with fewer than 2000 mutations
- Cancer Gene Census (CGC) TSG/OG gene labels (136 TSGs, 76 OGs; list in RamanLab/IdentifyTSGOG/data/cgc_genes)
- Processed 37-feature gene-by-feature matrix (FeatureMat.tsv)
- Trained random forest models for each of the five cross-validation folds

## Outputs

- pancancer_predictions.tsv: consensus TSG/OG labels with per-class probabilities for unlabelled genes
- feature_importance_ranking.csv: average per-feature rank across the five fold models
- Per-fold random forest models with stable hyper-parameter set from repeated random iterations

## How to apply

Build a gene-by-feature matrix (rows = genes/transcripts, 37 columns of ratio-metric and entropy features such as nonsense entropy, frameshift frequency and Damaging/kb) from COSMIC v79 somatic coding mutations restricted to samples with fewer than 2000 mutations, and label genes as TSG (136) or OG (76) using Cancer Gene Census. Train random forest classifiers per fivefold cross-validation fold on an 80% subset, estimating a stable hyper-parameter set via multiple random iterations to avoid overfitting the small training set. Extract per-fold feature importances and average their ranks across the five fold models to identify mutation types important for classification (top ranks dominated by LOF- and missense-related features such as Nonsense entropy and High missense frequency). Apply each of the five fold models to all unlabelled genes, consolidate per-model predictions (consolidateModelPredictions.py-style), and retain the consensus set of genes predicted by more than 4 of 5 models within the top 5 percentile of predictions, writing TSG/OG labels with per-class probabilities to pancancer_predictions.tsv.

## Related tools

- **cTaG** (Random forest pan-cancer classifier that builds per-fold TSG/OG models on 37 mutation-type features and predicts new driver genes from unlabelled genes) — https://github.com/RamanLab/cTaG
- **RandomForest** (Classification algorithm trained on 80% of 136 TSGs and 76 OGs per CV fold with hyper-parameters stabilised over multiple random iterations) — https://github.com/RamanLab/cTaG
- **MutSigCV** (Used to calculate mutation rates for post-hoc analysis of mutation frequencies of cTaG-predicted genes (not part of classification itself))

## Examples

```
python code/consolidateModelPredictions.py  # consolidate per-fold RF predictions into pancancer_predictions.tsv
```

## Evaluation signals

- Known TSG drivers ARID1A, TP53 and RB1 appear among high-probability TSG predictions, and TAB3 appears as an OG in the consensus set
- Consensus predictions restricted to genes predicted by more than 4 of the 5 fold models and within the top 5 percentile of predictions
- Feature importance ranking is dominated by LOF- and missense-related features (e.g. Nonsense entropy, High missense frequency, High Frameshift Frequency) in the top 18 average ranks
- Feature matrix has 37 columns per gene and training uses exactly 136 TSGs and 76 OGs from CGC
- cTaG recall for TSGs shows improvement relative to mutation-rate-based baselines, and CGC-listed drivers not used in training appear among predictions

## Limitations

- Binary TSG/OG classification cannot represent dual-function or intermediate driver classes, a stated limitation of cTaG
- Test-set performance could not be compared with Davoli et al. because their test set results were not published
- Missense and frameshift entropy/frequency features could not be computed for the Tokheim et al. comparison data, limiting overlap
- Small training set (136 TSGs, 76 OGs) requires multiple random iterations of cross-validation to avoid overfitting; predictions depend on stable hyper-parameter estimation

## Evidence

- [readme] The pan-cancer model is generated using random forest. Cancer Gene Census (CGC) is used to label genes as TSG or OG. The pan-cancer model is trained on these genes.: "The pan-cancer model is generated using random forest. Cancer Gene Census (CGC) is used to label genes as TSG or OG. The pan-cancer model is trained on these genes."
- [results] We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation.: "We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation."
- [results] To identify features important for the classification of TSGs and OGs, we calculated the average rank of each feature across all models.: "To identify features important for the classification of TSGs and OGs, we calculated the average rank of each feature across all models."
- [abstract] potential driver genes predicted are CD36, ZNF750 and ARHGAP35 as TSGs and TAB3 as an oncogene.: "potential driver genes predicted are CD36, ZNF750 and ARHGAP35 as TSGs and TAB3 as an oncogene."
- [methods] Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations.: "Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations."
