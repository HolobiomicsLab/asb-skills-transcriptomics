---
name: class-probability-table-generation
description: Use when you have a trained random forest TSG/OG model set with per-fold
  model files and a feature matrix of unlabelled genes, and need a consolidated consensus
  table of TSG/OG labels with per-class probabilities.
when_to_use_negative:
- Input is already a consolidated prediction table or you only need to re-label genes
  already classified as TSG/OG in the training set
- You lack the per-fold random forest models (e.g. only a single merged model is available),
  since consensus requires predictions from all five fold models
- The goal is tissue-specific driver prediction with fewer than 1000 samples per tissue,
  which this pan-cancer consolidation workflow does not cover
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: cTaG
  role: Random forest pan-cancer model whose per-fold classifiers are applied to unlabelled
    genes to obtain TSG/OG class probabilities
  repo: https://github.com/RamanLab/cTaG
- name: consolidateModelPredictions.py
  role: Script in the cTaG code folder used to consolidate per-fold model predictions
    into the pan-cancer consensus result
  repo: https://github.com/RamanLab/cTaG
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:950672f0fa879dd5fe261c977b91084422101a197469329423f8e964120d543b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - cTaG
  - consolidateModelPredictions.py
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# class-probability-table-generation

## Summary

Generates a consensus TSG/OG class-probability table for unlabelled genes by applying the cTaG random forest fold models to the 37-feature ratio-metric/entropy matrix and consolidating per-model predictions with a majority-vote and top-percentile filter.

## When to use

Use when you have a trained random forest TSG/OG model set with per-fold model files and a feature matrix of unlabelled genes, and need a consolidated consensus table of TSG/OG labels with per-class probabilities.

## When NOT to use

- Input is already a consolidated prediction table or you only need to re-label genes already classified as TSG/OG in the training set
- You lack the per-fold random forest models (e.g. only a single merged model is available), since consensus requires predictions from all five fold models
- The goal is tissue-specific driver prediction with fewer than 1000 samples per tissue, which this pan-cancer consolidation workflow does not cover

## Inputs

- Processed 37-feature matrix with rows = genes/transcripts and columns = ratio-metric and entropy features (FeatureMat.tsv)
- Random forest models built for each of the five cross-validation folds
- List of unlabelled genes (genes not labelled TSG or OG by Cancer Gene Census)

## Outputs

- Consensus TSG/OG prediction table with per-class probabilities for each consensus gene (pancancer_predictions.tsv)

## How to apply

Load the processed 37-feature matrix (rows = genes/transcripts, columns = ratio-metric and entropy features) from FeatureMat.tsv and the random forest models built for each cross-validation fold. Apply each of the five fold models to all genes not used for training, obtaining per-class TSG/OG probabilities for every unlabelled gene. Aggregate the per-model predictions using the consolidateModelPredictions.py-style consolidation of the pan-cancer analysis. Form the consensus set by retaining genes predicted by more than 4 of the 5 models and restricting to the top 5 percentile of predictions. Assign each consensus gene a TSG or OG label with its per-class probabilities and write the result to pancancer_predictions.tsv.

## Related tools

- **cTaG** (Random forest pan-cancer model whose per-fold classifiers are applied to unlabelled genes to obtain TSG/OG class probabilities) — https://github.com/RamanLab/cTaG
- **consolidateModelPredictions.py** (Script in the cTaG code folder used to consolidate per-fold model predictions into the pan-cancer consensus result) — https://github.com/RamanLab/cTaG

## Examples

```
python code/consolidateModelPredictions.py --models output/RandomForest --features data/FeatureMat/FeatureMat.tsv --out output/RandomForest/pancancer_predictions.tsv
```

## Evaluation signals

- Consensus set contains only genes predicted by more than 4 of the 5 fold models and restricted to the top 5 percentile of predictions
- Known drivers ARID1A, TP53 and RB1 appear as high-probability TSGs in the consensus predictions
- TAB3 appears as an OG in the consensus predictions, and CD36, ZNF750 and ARHGAP35 appear as TSGs among the unlabelled genes
- Output table pancancer_predictions.tsv contains one row per consensus gene with a TSG/OG label and per-class probabilities
- Number of feature columns consumed equals 37 (ratio-metric and entropy features) matching FeatureMat.tsv

## Limitations

- The model uses binary classification to identify TSGs and OGs, a limitation acknowledged by the authors
- Consensus requires predictions from all five fold models; a threshold of more than 4 of 5 means genes missed by two or more folds are dropped
- The approach may be biased by the small labelled training set (136 TSGs and 76 OGs), mitigated only by stable hyper-parameter estimation via multiple random iterations

## Evidence

- [results] All genes that were not used for training the models were classified into TSGs and OGs.: "All genes that were not used for training the models were classified into TSGs and OGs."
- [other] Consolidation across five folds and consensus threshold: [evidence span withheld — anchor status: not_found]
- [other] Consensus retention rule: [evidence span withheld — anchor status: not_found]
- [abstract] Known drivers as validation signal: "Some known cancer driver genes predicted by cTaG as TSGs with high probability are ARID1A, TP53, and RB1."
- [readme] Consolidation script exists in repo: "consolidateModelPredictions.py  # Consolidate results for pan-cancer analysis"
