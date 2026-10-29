---
name: somatic-mutation-feature-matrix-modeling
description: Use when you have COSMIC somatic coding mutation data and need to classify
  genes into TSG/OG categories from mutation-type features, or to predict new driver
  genes among unlabelled genes.
when_to_use_negative:
- The goal is mutation-rate-based driver discovery (e.g. MutSigCV-style background
  mutation rate modelling) rather than mutation-type feature classification
- Input is tissue-specific data for a tissue with 1000 or fewer samples (the analysis
  only used tissues with >1000 samples, e.g. breast, kidney, large_intestine)
- No somatic mutation annotations are available (e.g. expression or methylation data),
  since entropy and ratio-metric features cannot be computed
edam_operation: http://edamontology.org/operation_0394
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: cTaG
  role: Pan-cancer random-forest classifier that consumes the 37-feature matrix and
    CGC labels to produce TSG/OG predictions
  repo: https://github.com/RamanLab/cTaG
- name: Random Forest
  role: scikit-learn classifier trained per CV fold with fivefold cross-validated
    hyper-parameters (n_estimator, max_features, max_depth, criterion)
- name: MutSigCV
  role: Used post-hoc to analyse mutation frequencies of predicted genes via calculated
    mutation rates, not for model training
provenance:
  source_task_ids:
  - task_004
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:1d7324258f7bc59723086d662506187a7952d238fc9aa19d6a59340727cbbe02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - MutSigCV
  - Random Forest
  - cTaG
  edam_operation: http://edamontology.org/operation_0394
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# somatic-mutation-feature-matrix-modeling

## Summary

Build pan-cancer random-forest classifiers (cTaG) on a 37-feature ratio-metric/entropy feature matrix derived from COSMIC somatic mutation data to classify genes as TSGs or OGs, using CGC labels and stable-hyper-parameter estimation to avoid overfitting the small training set.

## When to use

Use when you have COSMIC somatic coding mutation data and need to classify genes into TSG/OG categories from mutation-type features, or to predict new driver genes among unlabelled genes.

## When NOT to use

- The goal is mutation-rate-based driver discovery (e.g. MutSigCV-style background mutation rate modelling) rather than mutation-type feature classification
- Input is tissue-specific data for a tissue with 1000 or fewer samples (the analysis only used tissues with >1000 samples, e.g. breast, kidney, large_intestine)
- No somatic mutation annotations are available (e.g. expression or methylation data), since entropy and ratio-metric features cannot be computed

## Inputs

- COSMIC v79 somatic coding mutation data (samples with <2000 mutations)
- Cancer Gene Census (CGC) TSG/OG gene labels (136 TSGs, 76-77 OGs)
- Pre-processed 37-feature matrix FeatureMat.tsv (github.com/RamanLab/IdentifyTSGOG/blob/master/data/FeatureMat/FeatureMat.tsv)
- List of unlabelled genes to classify

## Outputs

- Trained random forest models for each of the five CV folds
- Per-fold TSG/OG accuracy, precision, recall and F1 on training and test partitions (avg ± std)
- Consensus pan-cancer predictions of new TSGs/OGs with per-class probabilities (pancancer_predictions.tsv)

## How to apply

Generate a 37-column feature matrix (rows = genes/transcripts) from COSMIC v79 somatic coding mutations restricted to samples with fewer than 2000 mutations, computing ratio-metric plus missense/frameshift entropy and frequency features. Label genes using Cancer Gene Census annotations (136 TSGs and 76-77 OGs) and mark the remainder Unlabelled. Train scikit-learn random forest classifiers per fold on a random 80% subset of labelled genes, running fivefold cross-validation to estimate hyper-parameters (n_estimator, max_features, max_depth, criterion) and repeating over multiple random iterations to obtain a stable hyper-parameter set that avoids overfitting the small gene set. Apply each of the five fold models to all unlabelled genes and form a consensus retaining only genes predicted by more than 4 of 5 models within the top 5 percentile. Compare against a balanced bagging baseline: if train accuracy (~0.94) far exceeds test accuracy (~0.69-0.73), the hyper-parameter tuning and iteration procedure was not properly applied.

## Related tools

- **cTaG** (Pan-cancer random-forest classifier that consumes the 37-feature matrix and CGC labels to produce TSG/OG predictions) — https://github.com/RamanLab/cTaG
- **Random Forest** (scikit-learn classifier trained per CV fold with fivefold cross-validated hyper-parameters (n_estimator, max_features, max_depth, criterion))
- **MutSigCV** (Used post-hoc to analyse mutation frequencies of predicted genes via calculated mutation rates, not for model training)

## Examples

```
python code/TSG_OG_Feat_final.py  # loads FeatureMat.tsv + CGC labels, builds fivefold random forest models
```

## Evaluation signals

- Test-set accuracies match reported values within one SD: TSG ~0.83 ± 0.02 and OG ~0.76 ± 0.03
- A balanced bagging baseline trained on the same matrix should overfit (train OG 0.93 ± 0.05, TSG 0.94 ± 0.04; test OG 0.69 ± 0.06, TSG 0.73 ± 0.06), confirming the stable-hyper-parameter iteration step is needed; cTaG AUROC 0.76 ± 0.07 vs baseline 0.54 ± 0.07
- Consensus predictions retain only genes predicted by more than 4 of the 5 fold models and within the top 5 percentile of predictions
- Known drivers ARID1A, TP53 and RB1 emerge as high-probability TSGs and TAB3 as an OG in the consensus predictions
- Feature matrix schema: rows = genes/transcripts, exactly 37 feature columns (ratio-metric, entropy and mutation-frequency features)

## Limitations

- Only 136 TSGs and 76 OGs are available for training, so overfitting is a constant risk requiring the multiple-random-iteration hyper-parameter procedure
- cTaG uses binary TSG/OG classification, a stated limitation of the model
- Missense and frameshift entropy/frequency features could not be computed for Tokheim et al. comparison data, limiting overlap
- Comparison on a test set with Davoli et al. was not possible as their test-set results have not been published

## Evidence

- [readme] The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively.: "The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively."
- [readme] We overcome overfitting due to the small set of genes to train by estimating stable hyper-parameter set using multiple random iterations.: "We overcome overfitting due to the small set of genes to train by estimating stable hyper-parameter set using multiple random iterations."
- [results] We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation.: "We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation."
- [abstract] Some known cancer driver genes predicted by cTaG as TSGs with high probability are ARID1A, TP53, and RB1.: "Some known cancer driver genes predicted by cTaG as TSGs with high probability are ARID1A, TP53, and RB1."
- [methods] Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations.: "Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations."
