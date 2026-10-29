---
name: coding-potential-feature-encoding
description: Use when you have nucleotide transcript FASTA datasets (e.g., lncRNA
  vs PCT per species) and need to encode them into LncCat-style ORF-aware features
  for CatBoost classification or cross-species generalization testing.
when_to_use_negative:
- Input is already a computed feature table (encoding and feature extraction are unnecessary;
  go straight to prediction)
- Sequences are proteins or partial CDS fragments rather than full-length transcripts,
  since ORF-attention and ORF-extraction steps assume nucleotide transcripts with
  ORF structure
- Goal is to apply an off-the-shelf predictor like CPAT/CNCI/PLEK/RNAsamba rather
  than the CatBoost-based LncCat feature-fusion scheme
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0203
tools:
- name: LncCat
  role: Overall pipeline that combines the five feature types and provides the trained
    per-species CatBoost models used for cross-species prediction
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosted classifier that consumes the fused features and produces
    the lncRNA/PCT prediction scores
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:6aa73e4fd3b9459ef21e3b59852dbb4b16fa3366ffcec42e5a3cbcbfe3a448bf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/a525076133/LncCat
  repo_url: https://github.com/a525076133/LncCat
  tools:
  - CatBoost
  - LncCat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# coding-potential-feature-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encode transcript FASTA sequences into five fused feature types (codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features) and train/predict lncRNA vs. protein-coding transcripts (PCTs) with a CatBoost classifier (LncCat), including cross-species transfer evaluation via a 5x5 MCC matrix.

## When to use

Use when you have nucleotide transcript FASTA datasets (e.g., lncRNA vs PCT per species) and need to encode them into LncCat-style ORF-aware features for CatBoost classification or cross-species generalization testing.

## When NOT to use

- Input is already a computed feature table (encoding and feature extraction are unnecessary; go straight to prediction)
- Sequences are proteins or partial CDS fragments rather than full-length transcripts, since ORF-attention and ORF-extraction steps assume nucleotide transcripts with ORF structure
- Goal is to apply an off-the-shelf predictor like CPAT/CNCI/PLEK/RNAsamba rather than the CatBoost-based LncCat feature-fusion scheme

## Inputs

- Transcript nucleotide FASTA files for five species datasets (human, mouse, zebrafish, wheat, chicken) with lncRNA/PCT labels
- Trained LncCat (CatBoost) model files per species from https://github.com/a525076133/LncCat
- Pre-trained BERT model for ORF peptide sequence encoding

## Outputs

- Feature matrix per transcript with codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features
- Per-transcript prediction scores with binary labels at cutoff 0.5 (lncRNA as positive class)
- 5x5 cross-species MCC matrix written to cross_species_mcc_matrix.csv

## How to apply

Load the five species test datasets (human, mouse, zebrafish, wheat, chicken) and their trained LncCat CatBoost models from the LncCat repository. For every transcript, extract ORFs using Orfipy with Standard Code transl_table=1 (start codons ATG/TTG/CTG, stop codons TAA/TAG/TGA), then compute the five feature types: codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features; encode ORF peptide sequences with a pre-trained BERT model for contextual vector representations. Predict with a prediction-score cutoff of 0.5 (score >= 0.5 labeled lncRNA, < 0.5 labeled PCT, with lncRNA as the positive class). For cross-species transfer, evaluate each species model on every species dataset and compute MCC for each model-dataset pair to fill a 5x5 matrix, then write cross_species_mcc_matrix.csv.

## Related tools

- **LncCat** (Overall pipeline that combines the five feature types and provides the trained per-species CatBoost models used for cross-species prediction) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosted classifier that consumes the fused features and produces the lncRNA/PCT prediction scores) — https://github.com/a525076133/LncCat

## Examples

```
from catboost import CatBoostClassifier; m = CatBoostClassifier(); m.load_model('lncCat_human.cbm'); scores = m.predict_proba(features)[:,1]; labels = ['lncRNA' if s >= 0.5 else 'PCT' for s in scores]
```

## Evaluation signals

- Diagonal (same-species) MCC values of the 5x5 matrix should match the reported performances: human 0.9503, mouse 0.9219, zebrafish 0.8591, wheat 0.8672, chicken 0.9047
- Animal models (human/mouse/zebrafish) evaluated on wheat should yield MCC values near 0.67, 0.63, and 0.66 respectively
- Cross-species pattern should show high mutual transfer among homologous animals (e.g., human model MCC 0.93 on mouse and 0.90 on zebrafish) but degraded performance on the plant (wheat) dataset
- Every transcript must receive a binary label via the cutoff rule: score >= 0.5 as lncRNA (positive class), score < 0.5 as PCT
- ORF extraction must use Standard Code transl_table=1 with start codons ATG/TTG/CTG and stop codons TAA/TAG/TGA

## Limitations

- CatBoost hyperparameter tuning settings and versions of CatBoost or BERT are not reported, limiting exact reproducibility
- Species may lack annotations, making it impossible to build models for all species (a limitation the authors raise for non-model organisms)
- Cross-species transfer degrades for distant species: animal-trained models perform poorly on the wheat (plant) dataset

## Evidence
