---
name: random-forest-driver-gene-classification
description: Use when classifying unlabelled cancer genes into TSGs/OGs from somatic
  mutation feature matrices (37 ratio-metric/entropy features), e.g. pan-cancer or
  tissue-specific driver gene prediction with CGC training labels.
when_to_use_negative:
- Input genes already have TSG/OG labels or the goal is only to re-validate training-set
  genes rather than classify unlabelled genes
- Somatic mutation data contains samples with ≥2000 mutations or lacks COSMIC mutation-type
  annotations needed to compute the 37 ratio-metric/entropy features
- A background-mutation-rate method is required instead — cTaG deliberately avoids
  background mutation rate modelling and is not biased towards highly mutated genes,
  so it should not be used when such bias is desired
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: cTaG
  role: Pan-cancer random-forest model (classify TSG and OG) trained on CGC labels
    to classify unlabelled genes as TSGs or OGs
  repo: https://github.com/RamanLab/cTaG
- name: IdentifyTSGOG
  role: Provides feature-matrix building code (identifyTSGOG.py, TSG_OG_tissueAnalysis.py)
    and the pre-processed FeatureMat.tsv and CGC gene lists used for training
  repo: https://github.com/RamanLab/IdentifyTSGOG
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:90c2c2c311ed51f3666528f731e095096a3035367789d262a9f61084eee4666d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - IdentifyTSGOG
  - cTaG
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# random-forest-driver-gene-classification

## Summary

Train and apply the cTaG random-forest model that classifies genes as tumour suppressor genes (TSGs) or oncogenes (OGs) using 37 ratio-metric and entropy mutation-type features from COSMIC somatic mutation data, with stable hyper-parameters estimated via fivefold cross-validation plus multiple random iterations to avoid overfitting.

## When to use

Use when classifying unlabelled cancer genes into TSGs/OGs from somatic mutation feature matrices (37 ratio-metric/entropy features), e.g. pan-cancer or tissue-specific driver gene prediction with CGC training labels.

## When NOT to use

- Input genes already have TSG/OG labels or the goal is only to re-validate training-set genes rather than classify unlabelled genes
- Somatic mutation data contains samples with ≥2000 mutations or lacks COSMIC mutation-type annotations needed to compute the 37 ratio-metric/entropy features
- A background-mutation-rate method is required instead — cTaG deliberately avoids background mutation rate modelling and is not biased towards highly mutated genes, so it should not be used when such bias is desired

## Inputs

- COSMIC (v79) somatic coding mutation data filtered to samples with fewer than 2000 mutations
- Pre-processed 37-feature matrix FeatureMat.tsv (rows = genes/transcripts; columns = ratio-metric and entropy features)
- Cancer Gene Census (CGC) labels: 136 TSGs and 76 OGs used for training
- Tissue-specific feature matrices from IdentifyTSGOG data/FeatureMat/tissues (for tissue-specific prediction)

## Outputs

- Per-fold random-forest models and consensus TSG/OG predictions for unlabelled genes (output/RandomForest)
- Consensus tissue-specific driver gene predictions with model consensus scores (output/TissueSpecificAnalyisis)
- Balanced-bagging baseline metrics comparison (balanced_bagging_metrics.csv, when benchmarking)
- Number-of-random-iterations evaluation results (output/evalRandIter)

## How to apply

Build or load the pre-processed 37-feature matrix (FeatureMat.tsv from RamanLab/IdentifyTSGOG) whose rows are genes/transcripts and columns are ratio-metric and entropy features computed from COSMIC somatic coding mutations filtered to samples with fewer than 2000 mutations. Label genes using the Cancer Gene Census list (136 TSGs and 76 OGs for training); the remainder are Unlabelled prediction targets. Train random forest models on an 80% subset of the 136 TSGs and 76 OGs per fold with fivefold cross-validation for hyper-parameter estimation, followed by multiple random iterations to obtain stable hyper-parameters and avoid overfitting on the small training set. Apply consensus across the per-fold models to classify all unlabelled genes into TSGs and OGs, recording model consensus scores. For tissue-specific prediction, reuse the fixed pan-cancer model on per-tissue feature matrices built with TSG_OG_tissueAnalysis.py for the 10 tissues with >1000 COSMIC samples (breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas, prostate).

## Related tools

- **cTaG** (Pan-cancer random-forest model (classify TSG and OG) trained on CGC labels to classify unlabelled genes as TSGs or OGs) — https://github.com/RamanLab/cTaG
- **IdentifyTSGOG** (Provides feature-matrix building code (identifyTSGOG.py, TSG_OG_tissueAnalysis.py) and the pre-processed FeatureMat.tsv and CGC gene lists used for training) — https://github.com/RamanLab/IdentifyTSGOG

## Examples

```
python code/TSG_OG_Feat_final.py  # loads FeatureMat.tsv + CGC labels and trains random-forest models with CV; python code/TSG_OG_tissueAnalysis.py  # builds tissue feature matrices and applies the fixed pan-cancer model
```

## Evaluation signals

- Fivefold CV training/test accuracies within reported variability: training OG 0.93 ± 0.05 and TSG 0.94 ± 0.04; test OG 0.69 ± 0.06 and TSG 0.73 ± 0.06
- AUROC of cTaG on held-out folds (0.76 ± 0.07) should exceed the balanced-bagging baseline (0.54 ± 0.07)
- Prediction files exist for all 10 named tissues and each consensus list contains genes labelled TSG or OG with model consensus scores
- Known drivers ARID1A, TP53 and RB1 are predicted as TSGs with high probability; candidates include CD36, ZNF750, ARHGAP35 (TSG) and TAB3 (OG)
- Random-iteration count is validated via output/evalRandIter to confirm hyper-parameter stability

## Limitations

- The training set is small (136 TSGs, 76 OGs), so overfitting is a central risk; balanced bagging with standard grid-search CV overfits (test accuracies ~0.69–0.73 vs training ~0.93–0.94), motivating the stable-hyper-parameter procedure
- Comparison with Davoli et al. on a test set was not possible because their test-set results have not been published
- Missense and frameshift entropy and frequency features could not be calculated for the Tokheim et al. data, limiting comparison overlap
- cTaG uses binary classification for TSG/OG identity and does not capture genes with mixed or dual roles, as noted in the discussion

## Evidence

- [results] We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting: "We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting"
- [results] We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation.: "We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation."
- [methods] Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations.: "Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations."
- [readme] The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively.: "The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively."
- [abstract] Some known cancer driver genes predicted by cTaG as TSGs with high probability are ARID1A, TP53, and RB1.: "Some known cancer driver genes predicted by cTaG as TSGs with high probability are ARID1A, TP53, and RB1."
