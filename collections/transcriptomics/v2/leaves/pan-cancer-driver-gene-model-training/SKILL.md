---
name: pan-cancer-driver-gene-model-training
description: Use when building or reproducing a pan-cancer TSG/OG classifier from
  somatic mutation features for labelled driver genes, especially when the training
  set is small (136 TSGs, 76 OGs) and overfitting is a concern.
when_to_use_negative:
- The goal is driver discovery by mutation rate/significance testing (e.g. MutSigCV-style
  background mutation rate modelling) rather than supervised TSG/OG classification
  — cTaG is deliberately not rate-based
- 'Input data is tissue-specific with fewer than 1000 samples, for which the pan-cancer
  feature matrices were not generated (only tissues with >1000 samples: breast, CNS,
  cervix, endometrium, haematopoietic, kidney, large intestine, liver, pancreas, prostate)'
- The input is already a trained model or prediction list and the task is functional
  enrichment or literature validation rather than model training
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3474
tools:
- name: cTaG
  role: Pan-cancer random forest pipeline that builds the classifier and generates
    feature matrices; code in TSG_OG_Feat_final.py, identifyTSGOG.py and consolidateModelPredictions.py
  repo: https://github.com/RamanLab/cTaG
- name: Random Forest (scikit-learn)
  role: Underlying classifier trained on 80% of 136 TSGs and 76 OGs per CV fold, with
    hyper-parameters tuned by fivefold cross-validation over multiple random iterations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:e559d2c498fd2444d2514289d7c73afa45fed00c5d4048ebcd86f70d4a5056a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - Random Forest (scikit-learn)
  - cTaG
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pan-cancer-driver-gene-model-training

## Summary

Trains the cTaG pan-cancer random forest classifier that labels genes as TSGs or OGs, using 37 ratio-metric/entropy mutation-type features and CGC-derived labels, with fivefold cross-validation plus multiple random iterations to find a stable hyper-parameter set and avoid overfitting on the small training gene set.

## When to use

Use when building or reproducing a pan-cancer TSG/OG classifier from somatic mutation features for labelled driver genes, especially when the training set is small (136 TSGs, 76 OGs) and overfitting is a concern.

## When NOT to use

- The goal is driver discovery by mutation rate/significance testing (e.g. MutSigCV-style background mutation rate modelling) rather than supervised TSG/OG classification — cTaG is deliberately not rate-based
- Input data is tissue-specific with fewer than 1000 samples, for which the pan-cancer feature matrices were not generated (only tissues with >1000 samples: breast, CNS, cervix, endometrium, haematopoietic, kidney, large intestine, liver, pancreas, prostate)
- The input is already a trained model or prediction list and the task is functional enrichment or literature validation rather than model training

## Inputs

- Pre-processed pan-cancer feature matrix (FeatureMat.tsv: genes/transcripts x 37 ratio-metric and entropy features) built from COSMIC v79 somatic coding mutation data for samples with fewer than 2000 mutations
- CGC (Cancer Gene Census) TSG/OG label lists (from github.com/RamanLab/IdentifyTSGOG/tree/master/data/cgc_genes)

## Outputs

- Trained random forest models per cross-validation fold with consensus predictions classifying unlabelled genes into TSGs and OGs
- Stable hyper-parameter set (n_estimator, max_features, max_depth, criterion) from repeated fivefold cross-validation
- Per-fold and consolidated average ± standard deviation training/test accuracy, precision, recall and F1 for TSG and OG classes
- Evaluation outputs in output/RandomForest and output/evalRandIter directories

## How to apply

Load the pre-processed 37-feature pan-cancer feature matrix (FeatureMat.tsv from the IdentifyTSGOG/cTaG repositories) and attach CGC-derived TSG/OG labels to the 136 TSGs and 76-77 OGs. Train scikit-learn random forest models on a random 80% subset of these genes per fold, holding out the remaining 20% as test. Run fivefold cross-validation to estimate hyper-parameters (n_estimator, max_features, max_depth, criterion), then repeat over multiple random iterations to identify a stable hyper-parameter set, which is the paper's strategy for avoiding overfitting given the small gene set. Evaluate per-class accuracy, precision, recall and F1 on training and test partitions and consolidate folds into average ± standard deviation summaries. Valid hyper-parameters are those giving test accuracies near 0.83 ± 0.02 for TSGs and 0.76 ± 0.03 for OGs.

## Related tools

- **cTaG** (Pan-cancer random forest pipeline that builds the classifier and generates feature matrices; code in TSG_OG_Feat_final.py, identifyTSGOG.py and consolidateModelPredictions.py) — https://github.com/RamanLab/cTaG
- **Random Forest (scikit-learn)** (Underlying classifier trained on 80% of 136 TSGs and 76 OGs per CV fold, with hyper-parameters tuned by fivefold cross-validation over multiple random iterations)

## Examples

```
python code/TSG_OG_Feat_final.py  # loads FeatureMat.tsv + CGC TSG/OG labels, runs fivefold CV random forest with random-iteration hyper-parameter search
```

## Evaluation signals

- Test-set accuracy within the reported ranges: TSG 0.83 ± 0.02 and OG 0.76 ± 0.03 across folds
- Per-class OG precision/recall in the reported ballpark (OG precision ~0.79 ± 0.12, recall ~0.50 ± 0.19)
- Hyper-parameter selection stabilises across random iterations (check output/evalRandIter) rather than varying per run, indicating overfitting is controlled
- Training/test gene counts per fold match the paper: 80% of 136 TSGs and 76-77 OGs for training, 20% held out for testing
- Known drivers such as TP53, ARID1A and RB1 are classified as TSGs with high probability by the trained model

## Limitations

- Only 136 TSGs and ~76 OGs are available for training, so overfitting is a persistent risk mitigated (not eliminated) by the stable hyper-parameter strategy
- Binary classification does not handle genes that are both TSG and OG, and the paper notes this as a limitation of cTaG
- Missing labels: all non-CGC genes are Unlabelled, so model performance on novel drivers cannot be directly verified
- Comparison with Davoli et al. on the test set was not possible because their test set results have not been published

## Evidence

- [results] We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation.: "We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation."
- [results] We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting: "We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting"
- [readme] The model is built using somatic mutation data from COSMIC (v79) from differnt cancer types. We use ratio-metric and entropy features to classify genes as TSG or OG.: "The model is built using somatic mutation data from COSMIC (v79) from differnt cancer types. We use ratio-metric and entropy features to classify genes as TSG or OG."
- [results] The cTaG model achieved test set accuracies of 0.76 ± 0.03 for OGs and 0.83 ± 0.02 for TSGs: [evidence span withheld — anchor status: not_found]
