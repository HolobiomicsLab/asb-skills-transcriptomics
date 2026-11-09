---
name: benchmark-metric-reproduction-across-species
description: Use when replicating a published lncRNA-vs-PCT classifier's benchmark
  table on multi-species transcript datasets to check reported ACC, SEN, SPE, F1,
  PRE, MCC, and AUC values.
when_to_use_negative:
- Evaluating on sORF-enriched test sets (human/mouse/zebrafish sORF datasets) — those
  use a separate protocol with different reported improvement ranges
- Transcript sets not filtered with CD-HIT at 80% similarity or containing ncRNAs
  shorter than 200 nt — metric values will not be comparable to the published table
- Cross-species prediction scenarios — the article reports same-species models perform
  poorly on wheat (MCC ~0.63-0.67), so cross-species transfers are out of scope for
  this reproduction
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
tools:
- name: LncCat
  role: Model under reproduction; provides feature extraction, CatBoost classifier,
    and benchmark datasets
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosting classifier trained on the five-type feature matrices for
    each species
- name: RNAsamba
  role: Competing method benchmarked on the same five species datasets for comparison
  repo: http://github.com/apcamargo/RNAsamba
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:7e8b8f82835548479d138847662b7c49b0a5a6bc55cfd1ce09a08239123c93df
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
  - RNAsamba
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# benchmark-metric-reproduction-across-species

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce published classification benchmarks of LncCat, a CatBoost-based lncRNA identifier with ORF-attention features, on five species datasets (human, mouse, zebrafish, wheat, chicken) and verify the reported MCC values against the eight competing methods.

## When to use

Use when replicating a published lncRNA-vs-PCT classifier's benchmark table on multi-species transcript datasets to check reported ACC, SEN, SPE, F1, PRE, MCC, and AUC values.

## When NOT to use

- Evaluating on sORF-enriched test sets (human/mouse/zebrafish sORF datasets) — those use a separate protocol with different reported improvement ranges
- Transcript sets not filtered with CD-HIT at 80% similarity or containing ncRNAs shorter than 200 nt — metric values will not be comparable to the published table
- Cross-species prediction scenarios — the article reports same-species models perform poorly on wheat (MCC ~0.63-0.67), so cross-species transfers are out of scope for this reproduction

## Inputs

- Five species lncRNA/PCT transcript FASTA datasets (human, mouse, zebrafish, wheat, chicken) with 70/10/20 train/validation/test split
- Precomputed five-type feature matrices (codon-related, GC-related, sequence-related, peptide-related, ORF-attention)
- Trained CatBoost model from the LncCat repository

## Outputs

- Per-species metrics table (lnccat_species_metrics.csv) with ACC, SEN, SPE, F1, PRE, MCC, AUC
- Reproduced MCC values for comparison: 0.9503 (human), 0.9219 (mouse), 0.8591 (zebrafish), 0.8672 (wheat), 0.9047 (chicken)

## How to apply

Load the five species lncRNA/PCT training, validation, and test sets (70/10/20 split, CD-HIT filtered at 80% similarity, transcripts >=200 nt) from the released LncCat repository (github.com/a525076133/LncCat). Encode each transcript with the five feature types: codon-related (stop codon count/frequency, Fickett score), GC-related (GC1/GC2/GC3 and frame scores), sequence-related (length, k-mer, CTD, hexamer score), peptide-related (Mw, PI, Gravy, instability index), and ORF-attention features extracted with Orfipy using Standard Code transl_table=1 (start codons ATG/TTG/CTG). Train the CatBoost classifier per species on the training features, predict on the 20% held-out test set, and apply the fixed decision cutoff of 0.5 (score >= 0.5 predicted as lncRNA, < 0.5 as PCT). Compute the seven metrics per species and compare reproduced MCC values (0.9503 human, 0.9219 mouse, 0.8591 zebrafish, 0.8672 wheat, 0.9047 chicken) against the published values and the reported improvement ranges over the eight competing methods.

## Related tools

- **LncCat** (Model under reproduction; provides feature extraction, CatBoost classifier, and benchmark datasets) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosting classifier trained on the five-type feature matrices for each species)
- **RNAsamba** (Competing method benchmarked on the same five species datasets for comparison) — http://github.com/apcamargo/RNAsamba

## Examples

```
python reproduce_lnccat_benchmark.py --species human mouse zebrafish wheat chicken --cutoff 0.5 --output lnccat_species_metrics.csv
```

## Evaluation signals

- Reproduced human MCC within tolerance of 0.9503 (published F1 0.9742; next-best PLEK at MCC 0.9313)
- Reproduced mouse MCC ~0.9219, zebrafish ~0.8591, wheat ~0.8672, chicken ~0.9047
- Improvement ranges over the eight competing methods fall within reported windows (human 1.90-7.82%, mouse 1.49-17.63%, zebrafish 6.11-21.50%)
- Predictions use the fixed cutoff of 0.5 (>=0.5 lncRNA, <0.5 PCT) — verify no threshold tuning was applied
- Output CSV contains all seven metrics (ACC, SEN, SPE, F1, PRE, MCC, AUC) for each of the five species

## Limitations

- CatBoost hyperparameter tuning settings are not reported, and no CatBoost/BERT versions are specified, limiting exact reproducibility
- Web server results are returned only via user email rather than in the interface
- Species may lack annotations, making it impossible to build models for all species

## Evidence

- [abstract] Reported MCC on five species: "LncCat achieves 0.9503, 0.9219, 0.8591, 0.8672, and 0.9047 on the human, mouse, zebrafish, wheat, and chicken datasets"
- [methods] Dataset construction thresholds: "the open-source program CD-HIT [47] is used to reduce the redundancy of the datasets by a threshold of 80%"
- [methods] Train/val/test split: "the dataset is divided into training, validation, and test datasets, which account for 70%, 10%, and 20% of the total, respectively"
- [methods] Decision cutoff: "the cutoff value is set at 0.5, which means that the prediction score is greater than or equal to 0.5 for lncRNA and less than 0.5 for PCT"
- [readme] Improvement ranges over competing methods: "which improves by 1.90%~7.82%, 1.49%~17.63%, 6.11%~21.50%, 3.02%~51.64% and 5.35%~26.90%, respectively"
