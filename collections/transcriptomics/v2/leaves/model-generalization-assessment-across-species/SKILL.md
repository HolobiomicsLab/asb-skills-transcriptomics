---
name: model-generalization-assessment-across-species
description: Use when you have per-species trained models and test datasets and need
  to quantify whether models transfer across related species (e.g., animal-to-animal)
  but degrade on distant taxa (e.g., animal-to-plant).
when_to_use_negative:
- The goal is same-species benchmarking only — the diagonal entries of the matrix
  duplicate standard held-out test evaluation (e.g., human 0.9503, mouse 0.9219)
- Species lack annotations or sufficient data to train species-specific models, making
  a cross-species matrix impossible to populate (noted as a limitation in the article
  discussion)
- Comparing feature-engineering-based classifiers (like LncCat) with deep-learning
  predictors (e.g., RNAsamba, mRNN) on general coding-potential tasks rather than
  assessing transfer of a fixed set of trained models
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3308
tools:
- name: LncCat
  role: Source of the five trained per-species CatBoost models, feature computation
    code, and five species benchmark datasets used for cross-species evaluation
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosted classifier underlying each species-specific LncCat model
    being evaluated for transfer
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:d16f04e59dae700be734377437de59197a3275846040163c402730324ce36fe7
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# model-generalization-assessment-across-species

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematically evaluates trained species-specific classifiers (e.g., LncCat CatBoost models for human, mouse, zebrafish, wheat, chicken) by applying each model to every other species' test dataset, filling a cross-species MCC matrix to reveal homology-dependent transfer.

## When to use

Use when you have per-species trained models and test datasets and need to quantify whether models transfer across related species (e.g., animal-to-animal) but degrade on distant taxa (e.g., animal-to-plant).

## When NOT to use

- The goal is same-species benchmarking only — the diagonal entries of the matrix duplicate standard held-out test evaluation (e.g., human 0.9503, mouse 0.9219)
- Species lack annotations or sufficient data to train species-specific models, making a cross-species matrix impossible to populate (noted as a limitation in the article discussion)
- Comparing feature-engineering-based classifiers (like LncCat) with deep-learning predictors (e.g., RNAsamba, mRNN) on general coding-potential tasks rather than assessing transfer of a fixed set of trained models

## Inputs

- Five species test datasets as transcript sequences (human, mouse, zebrafish, wheat, chicken; FASTA)
- Five trained LncCat CatBoost models (one per species)
- Pre-trained BERT model for ORF peptide sequence encoding

## Outputs

- cross_species_mcc_matrix.csv containing the 5x5 model-dataset MCC matrix
- Interpretation of transfer pattern (e.g., human model MCC 0.93 on mouse, 0.90 on zebrafish; 0.67 on wheat)

## How to apply

Load the five species test datasets (human, mouse, zebrafish, wheat, chicken) and the five corresponding trained LncCat (CatBoost) models from the LncCat repository. For each transcript in each evaluation dataset, compute the five feature types (codon-related, GC-related, sequence-related, peptide-related, ORF-attention), extracting ORFs with Orfipy using the Standard Code (transl_table=1) so start codons ATG/TTG/CTG and stop codons TAA/TAG/TGA are recognized. Predict with each species model on each dataset using a prediction-score cutoff of 0.5 (score >= 0.5 labeled lncRNA, < 0.5 labeled PCT, with lncRNA as the positive class), then compute MCC for each model-dataset pair to fill a 5x5 matrix whose diagonal holds the same-species results. Interpret the off-diagonal pattern: models trained on homologous animal species (human, mouse, zebrafish) transfer well to each other but degrade on the plant species (wheat).

## Related tools

- **LncCat** (Source of the five trained per-species CatBoost models, feature computation code, and five species benchmark datasets used for cross-species evaluation) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosted classifier underlying each species-specific LncCat model being evaluated for transfer) — https://github.com/a525076133/LncCat

## Examples

```
python cross_species_eval.py --models models/{human,mouse,zebrafish,wheat,chicken}.cbm --datasets data/{human,mouse,zebrafish,wheat,chicken}.fasta --orfipy_args 'transl_table=1' --cutoff 0.5 --out cross_species_mcc_matrix.csv
```

## Evaluation signals

- Diagonal MCC values of the 5x5 matrix must match reported same-species performances: human 0.9503, mouse 0.9219, zebrafish 0.8591, wheat 0.8672, chicken 0.9047
- Off-diagonal wheat column for animal models (human, mouse, zebrafish) should fall near 0.63-0.67 MCC (0.67, 0.63, 0.66 respectively)
- Mutual transfer among homologous animal models should be high, e.g., human model MCC 0.93 on mouse and 0.90 on zebrafish
- All predictions use the fixed cutoff of 0.5 (score >= 0.5 = lncRNA, < 0.5 = PCT); changing the cutoff invalidates comparability with reported MCC values
- ORF extraction must use Orfipy with Standard Code transl_table=1 including ATG/TTG/CTG start codons; alternative genetic codes or ORF finders will alter feature vectors and MCCs

## Limitations

- Animal models perform poorly on the plant species (wheat), so cross-species transfer claims are restricted to homologous taxa
- Species may lack annotations, and it is impossible to build models for all species, limiting the matrix to well-annotated organisms
- CatBoost hyperparameter tuning settings and the versions of CatBoost and BERT are not reported, which hinders exact reproducibility of the transferred models
- The published transfer matrix was computed with a fixed 0.5 cutoff; performance under other thresholds is uncharacterized

## Evidence

- [methods] Human, mouse, and zebrafish models perform well on each other's datasets but poorly on wheat, with MCC of 0.67, 0.63, and 0.66: [evidence span withheld — anchor status: not_found]
