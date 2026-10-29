---
name: tissue-specific-feature-matrix-construction
description: Use when COSMIC somatic coding mutation data can be split by primary
  tissue of origin and the goal is to identify tissue-specific TSGs/OGs by reusing
  a pan-cancer cTaG model on per-tissue feature matrices.
when_to_use_negative:
- The tissue of interest has fewer than 1000 samples — the per-tissue feature matrix
  would be too sparse for reliable pan-cancer model transfer
- The input is already a pre-processed feature matrix (e.g., data/FeatureMat.tsv);
  proceed directly to cTaG prediction instead of re-building features
- Samples exceed the 2000-mutation filter cutoff, since the model was trained on COSMIC
  data filtered to samples below that threshold
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: cTaG
  role: Pan-cancer random forest model whose fixed hyper-parameters are applied to
    each tissue-specific feature matrix to classify unlabelled genes into TSGs and
    OGs
  repo: https://github.com/RamanLab/cTaG
- name: TSG_OG_tissueAnalysis.py
  role: Script in the cTaG code folder that creates tissue-specific feature matrices
    and makes tissue-specific predictions
  repo: https://github.com/RamanLab/cTaG
- name: identifyTSGOG.py
  role: Module for building the feature matrix (37 ratio-metric and entropy mutation-type
    features) from somatic mutation data
  repo: https://github.com/RamanLab/cTaG
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:fe4ab226f1b1e5edf13dd7ac5f48be935ee601049c5638570bba4f771e7f8105
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - TSG_OG_tissueAnalysis.py
  - cTaG
  - identifyTSGOG.py
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# tissue-specific-feature-matrix-construction

## Summary

Builds per-tissue mutation feature matrices (genes/transcripts as rows, 37 ratio-metric and entropy mutation-type features as columns) from COSMIC somatic coding mutations, so the fixed pan-cancer cTaG random forest model can be applied to tissue-specific driver gene prediction.

## When to use

Use when COSMIC somatic coding mutation data can be split by primary tissue of origin and the goal is to identify tissue-specific TSGs/OGs by reusing a pan-cancer cTaG model on per-tissue feature matrices.

## When NOT to use

- The tissue of interest has fewer than 1000 samples — the per-tissue feature matrix would be too sparse for reliable pan-cancer model transfer
- The input is already a pre-processed feature matrix (e.g., data/FeatureMat.tsv); proceed directly to cTaG prediction instead of re-building features
- Samples exceed the 2000-mutation filter cutoff, since the model was trained on COSMIC data filtered to samples below that threshold

## Inputs

- Filtered COSMIC somatic coding mutation data (v79) restricted to samples with fewer than 2000 mutations
- Primary tissue-of-origin annotations for each COSMIC sample
- COSMIC mutation type annotations used to compute the 37 ratio-metric and entropy features
- Per-tissue feature matrices in the IdentifyTSGOG/cTaG repository (data/FeatureMat/tissues)

## Outputs

- One per-tissue feature matrix (genes/transcripts x 37 mutation-type features) for each of the 10 retained tissues
- Tissue-specific TSG/OG predictions per tissue under output/TissueSpecificAnalyisis
- Consensus lists of tissue-specific driver gene predictions with model consensus scores per tissue

## How to apply

Load the filtered COSMIC somatic coding mutation data restricted to samples with fewer than 2000 mutations, and split samples by their primary tissue of origin. Retain only tissues with greater than 1000 samples — the 10 tissues: breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas and prostate. For each retained tissue, build a feature matrix with genes/transcripts as rows and 37 ratio-metric and entropy mutation-type features as columns using the identifyTSGOG feature-building code (TSG_OG_tissueAnalysis.py). Apply the fixed pan-cancer cTaG random forest model to each per-tissue feature matrix to classify unlabelled genes as TSG or OG, and consolidate predictions per tissue into a consensus list saved under output/TissueSpecificAnalyisis. The rationale is that the pan-cancer model captures patterns that may be lost in single-cohort analyses, while per-tissue matrices surface tissue-specific driver candidates (e.g., in large intestine and liver) missed by pan-cancer analysis.

## Related tools

- **cTaG** (Pan-cancer random forest model whose fixed hyper-parameters are applied to each tissue-specific feature matrix to classify unlabelled genes into TSGs and OGs) — https://github.com/RamanLab/cTaG
- **TSG_OG_tissueAnalysis.py** (Script in the cTaG code folder that creates tissue-specific feature matrices and makes tissue-specific predictions) — https://github.com/RamanLab/cTaG
- **identifyTSGOG.py** (Module for building the feature matrix (37 ratio-metric and entropy mutation-type features) from somatic mutation data) — https://github.com/RamanLab/cTaG

## Examples

```
python code/TSG_OG_tissueAnalysis.py
```

## Evaluation signals

- Prediction files exist for all 10 named tissues (breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas, prostate)
- Each consensus list contains genes labelled as TSG or OG with model consensus scores
- Every input sample passed the <2000 mutations filter and belongs to a tissue retained by the >1000-samples cutoff
- Each per-tissue feature matrix has exactly 37 ratio-metric/entropy feature columns with genes/transcripts as rows
- Tissue-specific candidate genes absent from the pan-cancer output (e.g., large intestine and liver candidates) are present in the corresponding per-tissue consensus list

## Limitations

- Only tissues with greater than 1000 samples can be analysed, so rare cancer types with small cohorts are excluded
- The pan-cancer cTaG model was trained on COSMIC v79 with binary TSG/OG classification, which the authors note is a limitation of the approach
- cTaG does not use background mutation rates, so very low-frequency genes in small tissues may be harder to classify reliably

## Evidence

- [other] samples restricted to fewer than 2000 mutations and split by primary tissue of origin: "Somatic coding mutation data from COSMIC was considered for samples less than 2000 mutations."
- [methods] tissue retention threshold: "Tissues with greater than 1000 samples were used."
- [readme] list of the 10 retained tissues: "These tissues are breast, central_nervous_system, cervix, endometrium, haematopoietic_and_lymphoid_tissue, kidney, large_intestine, liver, pancreas and prostate."
- [readme] feature matrix structure: "The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively."
- [results] purpose of tissue-specific analysis: "The objective of predicting genes using a subset of data specific to tumour primary tissue source was to identify genes specific to a cancer type."
- [intro] rationale for pan-cancer model transfer to tissues: "A pan-cancer model will help in identifying patterns that might be lost while studying a cohort or specific cancer type, owing to low sample sizes or mutation recurrence."
