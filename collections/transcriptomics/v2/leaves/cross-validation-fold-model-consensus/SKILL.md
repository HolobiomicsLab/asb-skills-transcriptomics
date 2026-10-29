---
name: cross-validation-fold-model-consensus
description: Use when classifying unlabelled genes as TSGs or OGs from a gene-by-feature
  mutation matrix (e.g. 37 ratio-metric/entropy features) where a single model would
  risk overfitting the small CGC-labelled training set.
when_to_use_negative:
- Input genes already have trusted TSG/OG labels (they are the training set, not prediction
  targets)
- The goal is a mutation-rate/background-model-based driver prediction (e.g. MutSigCV-style)
  rather than a feature-based TSG/OG classifier — cTaG deliberately avoids background
  mutation rate modelling
- Only one trained model or fold is available, since the consensus rule (>4 of 5 models
  plus top 5 percentile) cannot be applied
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: cTaG
  role: Pan-cancer random-forest pipeline whose fold models are trained per CV fold
    and whose predictions are consolidated into consensus TSG/OG calls
  repo: https://github.com/RamanLab/cTaG
- name: RandomForest
  role: Classifier trained once per cross-validation fold on 80% of the 136 TSGs and
    76 OGs to produce per-fold class probabilities
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:da0878012c5745da27f0cc7f08abffd53810a6035c4bae670a398f9162c2f8e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - RandomForest
  - cTaG
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-validation-fold-model-consensus

## Summary

Train a random forest model per cross-validation fold (on a subset of 136 TSGs and 76 OGs labelled from Cancer Gene Census) and consolidate the per-fold TSG/OG predictions on unlabelled genes into a single consensus driver-gene call, avoiding bias towards highly mutated genes.

## When to use

Use when classifying unlabelled genes as TSGs or OGs from a gene-by-feature mutation matrix (e.g. 37 ratio-metric/entropy features) where a single model would risk overfitting the small CGC-labelled training set.

## When NOT to use

- Input genes already have trusted TSG/OG labels (they are the training set, not prediction targets)
- The goal is a mutation-rate/background-model-based driver prediction (e.g. MutSigCV-style) rather than a feature-based TSG/OG classifier — cTaG deliberately avoids background mutation rate modelling
- Only one trained model or fold is available, since the consensus rule (>4 of 5 models plus top 5 percentile) cannot be applied

## Inputs

- FeatureMat.tsv — pre-processed gene/transcript-by-feature matrix with 37 ratio-metric, entropy and frequency features from COSMIC (v79/v9) somatic coding mutation data
- CGC-derived TSG/OG training labels for 136 TSGs and 76 OGs (from cgc_genes list in RamanLab/IdentifyTSGOG)
- List of unlabelled genes (genes neither TSG nor OG in CGC) to be classified

## Outputs

- pancancer_predictions.tsv — consensus TSG/OG label per unlabelled gene with per-class probabilities from the five fold models
- Random forest model files per cross-validation fold (output/RandomForest)

## How to apply

Load the pre-processed 37-feature matrix (FeatureMat.tsv, rows = genes/transcripts) and the CGC-derived labels for 136 TSGs and 76 OGs. Train a random forest classifier for each of five cross-validation folds on an 80% subset of the labelled genes, using a stable hyper-parameter set estimated via fivefold cross-validation plus multiple random iterations to avoid overfitting. Apply each fold model to all genes not used for training to obtain per-class TSG/OG probabilities per gene per fold. Aggregate the per-model predictions following the consolidateModelPredictions.py-style consolidation, then form the consensus set by retaining genes predicted by more than 4 of the 5 models and restricting to the top 5 percentile of predictions. Write the consensus labels with per-class probabilities to pancancer_predictions.tsv.

## Related tools

- **cTaG** (Pan-cancer random-forest pipeline whose fold models are trained per CV fold and whose predictions are consolidated into consensus TSG/OG calls) — https://github.com/RamanLab/cTaG
- **RandomForest** (Classifier trained once per cross-validation fold on 80% of the 136 TSGs and 76 OGs to produce per-fold class probabilities)

## Examples

```
python code/TSG_OG_Feat_final.py  # loads data/FeatureMat/FeatureMat.tsv + CGC labels, builds per-fold models; then python code/consolidateModelPredictions.py
```

## Evaluation signals

- Consensus set contains only genes predicted as TSG or OG by more than 4 of the 5 fold models AND within the top 5 percentile of predictions
- Known driver genes ARID1A, TP53 and RB1 appear as high-probability TSGs in the consensus output
- Expected new-driver calls recovered: CD36, ZNF750 and ARHGAP35 as TSGs and TAB3 as an OG among unlabelled genes
- pancancer_predictions.tsv schema includes per-gene TSG/OG label plus per-class probabilities from the consolidated fold models
- Each fold model was trained on exactly an 80% subset of the 136 TSGs and 76 OGs, with hyper-parameters identical (stable set) across folds

## Limitations

- Only 136 TSGs and 76 OGs are available for training, so overfitting is a key risk that the fold-consensus plus multiple random iterations for stable hyper-parameters attempt to mitigate
- cTaG uses binary TSG/OG classification and thus has limitations for genes with mixed or ambiguous roles
- Test-set performance could not be compared with Davoli et al. because their test-set results were not published

## Evidence

- [results] results: "We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation."
- [readme] readme: "The pan-cancer model is generated using random forest. Cancer Gene Census (CGC) is used to label genes as TSG or OG."
- [abstract] abstract: "potential driver genes predicted are CD36, ZNF750 and ARHGAP35 as TSGs and TAB3 as an oncogene."
