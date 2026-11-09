---
name: cross-species-lncrna-model-transfer-evaluation
description: Use when you have per-species trained lncRNA classifiers and independent
  FASTA test sets from multiple species, and need to measure homologous vs. divergent
  transfer (e.g., animal vs. plant) via an MCC matrix.
when_to_use_negative:
- You only need same-species benchmark performance (diagonal comparison against CPAT,
  CNCI, PLEK, CPC2, LncFinder, CPPred, mRNN, RNAsamba) rather than transfer analysis
- No species-specific trained models exist for your taxa — the article notes 'Species
  may lack annotations, and it is impossible to build models for all species.'
- Evaluating on sORF test datasets (human/mouse/zebrafish), which is a separate LncCat
  evaluation task
edam_operation: http://edamontology.org/operation_0267
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: LncCat
  role: Source of the five species-trained CatBoost models, feature extraction code,
    and test datasets used in the cross-species matrix
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosted classifier backbone whose per-species models are cross-applied
    to foreign-species datasets
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:603fb1adfc4bd30c65e0043beef132906b72149d245212ea4c39eef53f709812
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
  edam_operation: http://edamontology.org/operation_0267
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-species-lncrna-model-transfer-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how well species-specific LncCat (CatBoost) lncRNA classifiers transfer across the five-species benchmark (human, mouse, zebrafish, wheat, chicken) by scoring each trained model against every other species' test set and filling a 5x5 Matthew's Correlation Coefficient (MCC) matrix.

## When to use

Use when you have per-species trained lncRNA classifiers and independent FASTA test sets from multiple species, and need to measure homologous vs. divergent transfer (e.g., animal vs. plant) via an MCC matrix.

## When NOT to use

- You only need same-species benchmark performance (diagonal comparison against CPAT, CNCI, PLEK, CPC2, LncFinder, CPPred, mRNN, RNAsamba) rather than transfer analysis
- No species-specific trained models exist for your taxa — the article notes 'Species may lack annotations, and it is impossible to build models for all species.'
- Evaluating on sORF test datasets (human/mouse/zebrafish), which is a separate LncCat evaluation task

## Inputs

- Five species test datasets (human, mouse, zebrafish, wheat, chicken) as FASTA transcript sequences from the LncCat repository
- Five trained LncCat (CatBoost) models, one per species
- Orfipy with Standard Code transl_table=1 for ORF extraction
- Pre-trained BERT model for ORF-attention peptide encoding

## Outputs

- cross_species_mcc_matrix.csv: 5x5 MCC matrix, rows = species models, columns = evaluation datasets
- Diagonal entries: same-species MCC (human 0.9503, mouse 0.9219, zebrafish 0.8591, wheat 0.8672, chicken 0.9047)
- Off-diagonal animal-to-wheat MCC values in the 0.63-0.67 range

## How to apply

Load the five species test datasets and the five trained LncCat/CatBoost models from https://github.com/a525076133/LncCat. For every transcript in each evaluation set, compute the five feature types (codon-related, GC-related, sequence-related, peptide-related, ORF-attention), extracting ORFs with Orfipy using Standard Code transl_table=1 (start codons ATG/TTG/CTG, stops TAA/TAG/TGA). Predict with each model at the fixed cutoff of 0.5 (score >= 0.5 => lncRNA, positive class; < 0.5 => PCT). Compute MCC for every model-dataset pair to populate the 5x5 matrix whose diagonal holds same-species results. Rationale: the article's expectation is strong mutual transfer among homologous animals (human/mouse/zebrafish) and degraded transfer to the plant species, so deviations from this pattern flag pipeline errors.

## Related tools

- **LncCat** (Source of the five species-trained CatBoost models, feature extraction code, and test datasets used in the cross-species matrix) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosted classifier backbone whose per-species models are cross-applied to foreign-species datasets) — https://github.com/a525076133/LncCat

## Examples

```
orfipy transcripts.fa --table 1 --pep orfs.pep --min 1 && python -c "from sklearn.metrics import matthews_corrcoef; print(matthews_corrcoef(y_true, model.predict(X)))"
```

## Evaluation signals

- Diagonal MCC values match published same-species results: human 0.9503, mouse 0.9219, zebrafish 0.8591, wheat 0.8672, chicken 0.9047
- Human model yields MCC ~0.93 on mouse and ~0.90 on zebrafish (strong animal-to-animal transfer)
- Animal models on wheat fall in the 0.63-0.67 MCC band (human 0.67, mouse 0.63, zebrafish 0.66)
- Matrix is complete (all 25 model-dataset pairs scored) and written to cross_species_mcc_matrix.csv
- Predictions use the fixed 0.5 cutoff with lncRNA as the positive class; re-running with the same models/datasets reproduces the matrix deterministically

## Limitations

- Transfer to phylogenetically distant species (wheat) degrades substantially, so cross-applied predictions cannot be trusted across kingdom boundaries
- CatBoost and BERT versions and hyperparameter tuning settings are not reported, limiting exact reproducibility
- Species lacking annotations cannot have models built, so the transfer matrix cannot be extended to all species

## Evidence

- [methods] Cross-species models perform best on same-species datasets and poorly on wheat (MCC 0.67, 0.63, 0.66): [evidence span withheld — anchor status: not_found]
