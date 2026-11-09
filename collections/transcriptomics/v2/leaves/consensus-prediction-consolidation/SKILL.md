---
name: consensus-prediction-consolidation
description: Use when multiple random forest classifiers (e.g. per CV fold or per
  random iteration) each label the same unlabelled genes as TSG or OG, and a single
  stable per-gene consensus prediction with a consensus score is needed.
when_to_use_negative:
- Input is already a consolidated consensus list — re-consolidating would double-count
  model agreement
- Tissue subsets with fewer than 1000 samples, which the cTaG tissue-specific analysis
  explicitly excludes
- Samples with 2000 or more mutations, which were filtered out before feature matrix
  construction
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3474
tools:
- name: cTaG
  role: Random forest ensemble whose per-model TSG/OG predictions are consolidated
    into consensus driver gene calls
  repo: https://github.com/RamanLab/cTaG
- name: IdentifyTSGOG
  role: Provides the feature matrices and consolidateModelPredictions.py used to build
    and merge per-model predictions; repository now redirects to cTaG
  repo: https://github.com/RamanLab/IdentifyTSGOG
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:c6f57552c7fd7d6c508c803bafb5ffc7bd3a99f557cd6c0ca2c8b9d69a5dcb09
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - IdentifyTSGOG
  - cTaG
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Consensus prediction consolidation

## Summary

Consolidates predictions from multiple random forest models (built per cross-validation fold and across random iterations) into a single consensus label and score per gene, as done by the cTaG pan-cancer and tissue-specific driver gene analyses. Consensus across models mitigates overfitting from the small training set (136 TSGs, 76 OGs from CGC).

## When to use

Use when multiple random forest classifiers (e.g. per CV fold or per random iteration) each label the same unlabelled genes as TSG or OG, and a single stable per-gene consensus prediction with a consensus score is needed.

## When NOT to use

- Input is already a consolidated consensus list — re-consolidating would double-count model agreement
- Tissue subsets with fewer than 1000 samples, which the cTaG tissue-specific analysis explicitly excludes
- Samples with 2000 or more mutations, which were filtered out before feature matrix construction

## Inputs

- Per-model prediction outputs from random forest classifiers trained on CGC-labelled TSGs (136) and OGs (76)
- Pan-cancer feature matrix (37 ratio-metric and entropy mutation-type features per gene/transcript) from COSMIC v79 samples with <2000 mutations
- Tissue-specific feature matrices (data/FeatureMat/tissues) for the 10 tissues with >1000 samples

## Outputs

- Consensus pan-cancer prediction list of unlabelled genes classified as TSG or OG with consensus scores (output/RandomForest)
- Per-tissue consensus prediction lists under output/TissueSpecificAnalyisis for breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas and prostate

## How to apply

Run the cTaG random forest pipeline (TSG_OG_Feat_final.py / TSG_OG_tissueAnalysis.py) which trains models on 80% subsets of 136 TSGs and 76 OGs per cross-validation fold, repeating over multiple random iterations to identify stable hyper-parameters. Collect the per-model TSG/OG predictions for every unlabelled gene (genes not in the CGC training set). Apply the consolidation code (consolidateModelPredictions.py for pan-cancer, or the tissue-specific variant under output/TissueSpecificAnalyisis) to compute, for each gene, the consensus label and a consensus score aggregating agreement across models. Save per-tissue consensus outputs separately for the 10 tissues with >1000 samples (breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas, prostate). The rationale is that consensus across independently trained models counters overfitting due to the small labelled training set.

## Related tools

- **cTaG** (Random forest ensemble whose per-model TSG/OG predictions are consolidated into consensus driver gene calls) — https://github.com/RamanLab/cTaG
- **IdentifyTSGOG** (Provides the feature matrices and consolidateModelPredictions.py used to build and merge per-model predictions; repository now redirects to cTaG) — https://github.com/RamanLab/IdentifyTSGOG

## Examples

```
python code/consolidateModelPredictions.py  # consolidate per-model random forest predictions into consensus TSG/OG calls (pan-cancer or per-tissue under output/TissueSpecificAnalyisis)
```

## Evaluation signals

- Prediction files exist for all 10 named tissues under output/TissueSpecificAnalyisis
- Each consensus list contains genes labelled as TSG or OG with model consensus scores
- Consensus predictions include known drivers (e.g. ARID1A, TP53, RB1 as high-probability TSGs) validating against CGC
- Number of random iterations for consolidation is verified as sufficient via output/evalRandIter
- No training-set CGC genes appear as 'new' predictions; only unlabelled genes are classified

## Limitations

- Binary classification into TSGs and OGs is a stated limitation of cTaG, so dual-function or context-dependent genes may be mislabelled in the consensus
- Small training set (136 TSGs and 76 OGs from CGC) means consensus scores rest on limited labelled data; stability is approximated via multiple random iterations rather than guaranteed
- Davoli et al. test-set performance could not be compared because their test-set results were not published

## Evidence

- [readme] Consensus across models is used for predicting new TSGs and OGs.: "Consensus across models is used for predicting new TSGs and OGs."
- [results] Random iterations for stable hyper-parameters: "We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting"
- [readme] Overfitting mitigation rationale: "We overcome overfitting due to the small set of genes to train by estimating stable hyper-parameter set using multiple random iterations."
- [readme] Tissue-specific consensus outputs: "└── TissueSpecificAnalyisis # Tissue specific predictions and consensus of results"
- [results] Unlabelled genes classified into TSGs and OGs: "All genes that were not used for training the models were classified into TSGs and OGs."
