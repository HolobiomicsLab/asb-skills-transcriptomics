---
name: tissue-specific-driver-gene-prediction
description: Use when somatic mutation data can be split by primary tissue of origin
  (tissues with >1000 samples) and the goal is to predict cancer-type-specific driver
  genes rather than pan-cancer drivers.
when_to_use_negative:
- Input samples have not been filtered to fewer than 2000 mutations or the tissue
  of interest has 1000 or fewer samples — the model and tissue split were defined
  for the filtered, >1000-sample setting.
- Goal is pan-cancer (all-cancer) driver prediction rather than tissue-specific candidates
  — use the pan-cancer cTaG pipeline (consolidateModelPredictions.py) instead.
- Input is already a gene-level feature table unrelated to the 37 ratio-metric/entropy
  mutation-type features — the fixed random forest expects this exact feature schema.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_2640
tools:
- name: cTaG
  role: Fixed pan-cancer random forest model applied to per-tissue feature matrices
    to classify unlabelled genes as TSG or OG
  repo: https://github.com/RamanLab/cTaG
- name: identifyTSGOG / TSG_OG_tissueAnalysis.py
  role: Feature-building code that creates tissue-specific 37-feature matrices (ratio-metric
    and entropy mutation-type features) and makes per-tissue predictions
  repo: https://github.com/RamanLab/IdentifyTSGOG
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:41de1954034388130a82f4d9ca1f8c697239316061d7ac096434986f3074c432
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - cTaG
  - identifyTSGOG / TSG_OG_tissueAnalysis.py
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# tissue-specific-driver-gene-prediction

## Summary

Applies the fixed pan-cancer cTaG random forest model to per-tissue feature matrices built from COSMIC somatic coding mutations to identify tissue-specific tumour suppressor genes (TSGs) and oncogenes (OGs), uncovering driver candidates missed by the pan-cancer analysis.

## When to use

Use when somatic mutation data can be split by primary tissue of origin (tissues with >1000 samples) and the goal is to predict cancer-type-specific driver genes rather than pan-cancer drivers.

## When NOT to use

- Input samples have not been filtered to fewer than 2000 mutations or the tissue of interest has 1000 or fewer samples — the model and tissue split were defined for the filtered, >1000-sample setting.
- Goal is pan-cancer (all-cancer) driver prediction rather than tissue-specific candidates — use the pan-cancer cTaG pipeline (consolidateModelPredictions.py) instead.
- Input is already a gene-level feature table unrelated to the 37 ratio-metric/entropy mutation-type features — the fixed random forest expects this exact feature schema.

## Inputs

- COSMIC somatic coding mutation data (v79), filtered to samples with fewer than 2000 mutations, annotated with primary tissue of origin
- CGC labels assigning genes as TSG or OG (from github.com/RamanLab/IdentifyTSGOG data/cgc_genes)
- Pan-cancer cTaG random forest model with stable hyper-parameters
- identifyTSGOG feature-building module producing 37-feature matrices (genes/transcripts x features)

## Outputs

- Per-tissue feature matrices (data/FeatureMat/tissues)
- Per-tissue cTaG TSG/OG predictions with consensus scores (output/TissueSpecificAnalyisis)
- Consensus list of tissue-specific driver gene predictions per tissue

## How to apply

Load the filtered COSMIC somatic coding mutation data (samples restricted to fewer than 2000 mutations) and split samples by their primary tissue of origin. Retain only tissues with greater than 1000 samples — breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas and prostate. For each retained tissue, build a feature matrix with genes/transcripts as rows and the 37 ratio-metric and entropy mutation-type features as columns using the identifyTSGOG feature-building code (TSG_OG_tissueAnalysis.py). Apply the fixed pan-cancer cTaG random forest model to each per-tissue feature matrix to classify unlabelled genes into TSGs and OGs, then consolidate model predictions per tissue into a consensus list saved under output/TissueSpecificAnalyisis. The rationale is that a pan-cancer model captures patterns lost in single-cohort studies while tissue subsetting reveals genes specific to a cancer type.

## Related tools

- **cTaG** (Fixed pan-cancer random forest model applied to per-tissue feature matrices to classify unlabelled genes as TSG or OG) — https://github.com/RamanLab/cTaG
- **identifyTSGOG / TSG_OG_tissueAnalysis.py** (Feature-building code that creates tissue-specific 37-feature matrices (ratio-metric and entropy mutation-type features) and makes per-tissue predictions) — https://github.com/RamanLab/IdentifyTSGOG

## Examples

```
python code/TSG_OG_tissueAnalysis.py
```

## Evaluation signals

- Prediction files exist for all 10 named tissues: breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas and prostate.
- Each per-tissue consensus list contains genes labelled as TSG or OG with model consensus scores.
- Per-tissue feature matrices have the expected schema: genes/transcripts as rows and 37 ratio-metric and entropy mutation-type features as columns.
- Input samples respect the <2000 mutation filter and each retained tissue exceeds the >1000 sample threshold.
- Tissue-specific candidates not reported by the pan-cancer analysis (e.g. in large intestine and liver) appear in the per-tissue outputs.

## Limitations

- Only tissues with greater than 1000 COSMIC samples could be analysed, limiting coverage to the 10 retained tissues.
- The cTaG model uses binary classification to identify TSGs and OGs, which the authors note as a limitation of the approach.
- Predictions derive from a fixed pan-cancer model rather than per-tissue retrained models, so tissue specificity comes from the input data, not adapted model weights.

## Evidence

- [results] Tissue-specific predictions: "The objective of predicting genes using a subset of data specific to tumour primary tissue source was to identify genes specific to a cancer type."
- [methods] Tissue sample threshold: "Tissues with greater than 1000 samples were used."
- [readme] Ten retained tissues: "These tissues are breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas and prostate."
- [readme] Tissue-specific analysis setup: "The mutation data from COSMIC was also used for tissue-specific analysis. The samples were divided based on their primary tissue for origin."
- [readme] Mutation filter and feature schema: "Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations."
