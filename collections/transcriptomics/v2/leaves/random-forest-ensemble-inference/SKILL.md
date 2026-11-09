---
name: random-forest-ensemble-inference
description: Use when a small labelled gene set (e.g. 136 TSGs and 76 OGs from CGC)
  must be extended to unlabelled genes using somatic mutation-derived features, and
  consensus across ensemble fold models is needed to reduce overfitting.
when_to_use_negative:
- Input is already a classified driver-gene list or you only need mutation rates —
  MutSigCV-style rate estimation is the appropriate comparison tool, not ensemble
  re-classification
- Somatic mutation samples exceed the 2000-mutation filter used to build the feature
  matrix, or the goal is tissue-specific prediction from subsets with fewer than 1000
  samples
- Labelled training genes are not available from CGC — the random forest requires
  TSG/OG labels to train per-fold models
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
tools:
- name: cTaG
  role: Pan-cancer random forest model trained on 37 ratio-metric/entropy features
    to classify unlabelled genes as TSGs or OGs via cross-fold consensus
  repo: https://github.com/RamanLab/cTaG
- name: MutSigCV
  role: Calculates mutation rates used to analyse mutation frequencies of genes predicted
    by the ensemble model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:bf8f1e101dd7d3ba43bda3594b2cc6de421609d57f3b59e0e508bbcd000ec4af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - MutSigCV
  - cTaG
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# random-forest-ensemble-inference

## Summary

Apply a random-forest ensemble (cTaG pan-cancer model) to classify unlabelled genes as tumour suppressor genes (TSGs) or oncogenes (OGs) using 37 ratio-metric and entropy mutation features, then consolidate per-fold model predictions into a consensus set of candidate new driver genes.

## When to use

Use when a small labelled gene set (e.g. 136 TSGs and 76 OGs from CGC) must be extended to unlabelled genes using somatic mutation-derived features, and consensus across ensemble fold models is needed to reduce overfitting.

## When NOT to use

- Input is already a classified driver-gene list or you only need mutation rates — MutSigCV-style rate estimation is the appropriate comparison tool, not ensemble re-classification
- Somatic mutation samples exceed the 2000-mutation filter used to build the feature matrix, or the goal is tissue-specific prediction from subsets with fewer than 1000 samples
- Labelled training genes are not available from CGC — the random forest requires TSG/OG labels to train per-fold models

## Inputs

- Pre-processed feature matrix FeatureMat.tsv (rows = genes/transcripts, 37 ratio-metric and entropy features)
- Random forest models built for each cross-validation fold
- CGC-derived gene labels (TSG/OG/Unlabelled) from cgc_genes list
- COSMIC v79 somatic coding mutation data filtered to samples with fewer than 2000 mutations

## Outputs

- pancancer_predictions.tsv: consensus genes with TSG/OG labels and per-class probabilities
- Consensus set of predicted new driver genes (e.g. CD36, ZNF750, ARHGAP35 as TSGs; TAB3 as OG)

## How to apply

Load the pre-processed 37-column feature matrix (FeatureMat.tsv, rows = genes/transcripts, columns = ratio-metric and entropy features). Train random forest classifiers per cross-validation fold on 80% of the 136 TSGs and 76 OGs, estimating stable hyper-parameters via fivefold cross-validation plus multiple random iterations to avoid overfitting. Apply each of the five fold models to all genes not used for training to obtain per-class TSG/OG probabilities. Consolidate per-model predictions across folds (consolidateModelPredictions.py-style aggregation), retaining genes predicted by more than 4 of the 5 models and restricting to the top 5 percentile to form the consensus set. Assign each consensus gene a TSG or OG label with per-class probabilities and write the table (e.g. pancancer_predictions.tsv).

## Related tools

- **cTaG** (Pan-cancer random forest model trained on 37 ratio-metric/entropy features to classify unlabelled genes as TSGs or OGs via cross-fold consensus) — https://github.com/RamanLab/cTaG
- **MutSigCV** (Calculates mutation rates used to analyse mutation frequencies of genes predicted by the ensemble model)

## Examples

```
python code/consolidateModelPredictions.py  # aggregate per-fold random forest TSG/OG probabilities into pancancer_predictions.tsv consensus (>4/5 models, top 5 percentile)
```

## Evaluation signals

- Known drivers ARID1A, TP53 and RB1 appear as high-probability TSGs in the consensus predictions
- TAB3 appears as an OG and CD36, ZNF750, ARHGAP35 as TSGs among consensus unlabelled-gene predictions
- Every consensus gene is predicted by more than 4 of the 5 fold models and falls in the top 5 percentile of predictions
- Feature matrix has exactly 37 columns (ratio-metric and entropy features) with rows matching genes/transcripts
- Overfitting is controlled: stable hyper-parameters confirmed via evaluation of multiple random iterations (evalRandIter output)

## Limitations

- Binary classification only — cTaG identifies TSGs and OGs but cannot model other driver classes; the authors note this as a limitation
- Feature space is limited to missense/frameshift entropy and frequency features; these could not be computed for external comparison data (Tokheim et al.), limiting overlap
- Consensus thresholds (>4 of 5 models, top 5 percentile) are tuned to the small CGC training set (136 TSGs, 76 OGs); predictions are candidates for experimental screening, not validated drivers
- Davoli et al. test set results were not published, so test-set performance could not be benchmarked

## Evidence

- [results] We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation.: "We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation."
- [results] All genes not used for training were classified and consensus formed.: "All genes that were not used for training the models were classified into TSGs and OGs."
- [abstract] Potential new driver genes predicted by cTaG.: "potential driver genes predicted are CD36, ZNF750 and ARHGAP35 as TSGs and TAB3 as an oncogene."
- [readme] README on random forest and consensus prediction pipeline.: "Random forest is used to build multiple classifiers for each fold (Block A) and overfitting is avoided by identification of stable hyper-parameters (Block B). Consensus across models is used for"
- [readme] README on feature matrix construction.: "The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively."
