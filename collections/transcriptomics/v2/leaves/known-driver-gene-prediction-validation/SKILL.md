---
name: known-driver-gene-prediction-validation
description: Use when a trained TSG/OG random forest model exists and you need consensus
  predictions on unlabelled genes, with sanity checks against known Cancer Gene Census
  driver genes.
when_to_use_negative:
- The input is already a consolidated prediction table rather than a raw feature matrix
  plus per-fold models
- Genes of interest are already labelled TSG/OG in CGC and were used for training,
  so they are not part of the unlabelled prediction set
- The goal is a single-model probability estimate rather than a multi-fold consensus
  requiring >4/5 model agreement
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: cTaG
  role: Random-forest pan-cancer TSG/OG classifier whose per-fold models are applied
    to unlabelled genes and consolidated for consensus predictions
  repo: https://github.com/RamanLab/cTaG
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:fc039018b4838714e0e9de662bd092bc585fe6087410d3aed291f71320484139
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - cTaG
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# known-driver-gene-prediction-validation

## Summary

Validates cTaG driver-gene predictions by applying the five cross-validation random forest models to all unlabelled genes, consolidating predictions across folds, and checking that known drivers (ARID1A, TP53, RB1) appear as high-probability TSGs alongside candidate new drivers (CD36, ZNF750, ARHGAP35 as TSGs; TAB3 as OG).

## When to use

Use when a trained TSG/OG random forest model exists and you need consensus predictions on unlabelled genes, with sanity checks against known Cancer Gene Census driver genes.

## When NOT to use

- The input is already a consolidated prediction table rather than a raw feature matrix plus per-fold models
- Genes of interest are already labelled TSG/OG in CGC and were used for training, so they are not part of the unlabelled prediction set
- The goal is a single-model probability estimate rather than a multi-fold consensus requiring >4/5 model agreement

## Inputs

- Processed 37-feature matrix (FeatureMat.tsv) with rows = genes/transcripts and columns = ratio-metric and entropy features
- Random forest models built for each of the five cross-validation folds
- Training label sets: 136 TSGs and 76 OGs from Cancer Gene Census (CGC)

## Outputs

- pancancer_predictions.tsv: consensus TSG/OG labels with per-class probabilities for unlabelled genes
- Consensus set of candidate new driver genes (e.g. CD36, ZNF750, ARHGAP35 as TSGs; TAB3 as OG)

## How to apply

Load the processed 37-feature matrix (FeatureMat.tsv; rows = genes/transcripts, columns = ratio-metric and entropy features) and the random forest models built for each of the five cross-validation folds. Apply each fold model to every gene not used for training to obtain per-class TSG/OG probabilities. Consolidate per-model predictions across the five folds following consolidateModelPredictions.py-style aggregation, retaining genes predicted by more than 4 of the 5 models and restricting to the top 5 percentile of predictions. Assign each consensus gene a TSG or OG label with per-class probabilities and write the table to pancancer_predictions.tsv. Judge correctness by verifying that known drivers ARID1A, TP53 and RB1 appear as high-probability TSGs and that TAB3 appears as an OG in the consensus set.

## Related tools

- **cTaG** (Random-forest pan-cancer TSG/OG classifier whose per-fold models are applied to unlabelled genes and consolidated for consensus predictions) — https://github.com/RamanLab/cTaG

## Examples

```
python code/consolidateModelPredictions.py  # aggregate per-fold RF predictions on unlabelled genes into pancancer_predictions.tsv consensus
```

## Evaluation signals

- Known TSGs ARID1A, TP53 and RB1 appear among high-probability TSG predictions
- TAB3 appears as an OG in the consensus predictions
- Consensus set contains only genes predicted by more than 4 of the 5 fold models
- Consensus set is restricted to the top 5 percentile of predictions
- Output file pancancer_predictions.tsv contains per-class probabilities for each consensus gene

## Limitations

- cTaG uses binary classification to identify TSGs and OGs, which the authors note as a limitation
- Model is trained on only 136 TSGs and 76 OGs, requiring stable hyper-parameter estimation via multiple random iterations to avoid overfitting
- The approach, unlike background-mutation-rate methods, is not biased towards highly mutated genes, but performance on test sets of other methods (e.g. Davoli et al.) could not be compared because their test-set results were not published

## Evidence

- [results] Consolidate per-model predictions and form consensus: "All genes that were not used for training the models were classified into TSGs and OGs."
- [abstract] Known drivers recovered as TSGs: "Some known cancer driver genes predicted by cTaG as TSGs with high probability are ARID1A, TP53, and RB1."
- [abstract] New driver predictions: "potential driver genes predicted are CD36, ZNF750 and ARHGAP35 as TSGs and TAB3 as an oncogene."
- [readme] Training data and feature matrix size: "The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively."
- [results] Cross-validation model construction: "We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation."
