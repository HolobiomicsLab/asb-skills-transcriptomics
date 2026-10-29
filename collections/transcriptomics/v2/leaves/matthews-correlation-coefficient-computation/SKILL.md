---
name: matthews-correlation-coefficient-computation
description: Use when benchmarking a binary transcript classifier (lncRNA vs. protein-coding
  transcript, lncRNA as positive class) on imbalanced five-species datasets and you
  need a single balanced metric, or when quantifying cross-species model transfer.
when_to_use_negative:
- When the task requires threshold-free metrics such as ROC-AUC or PR-AUC — MCC here
  depends on the fixed 0.5 prediction-score cutoff (>= 0.5 = lncRNA, < 0.5 = PCT).
- When input is already an aggregate performance table or published matrix rather
  than per-transcript predictions — no confusion matrix can be recomputed.
- For multi-class or regression outputs; LncCat's task is binary lncRNA vs. PCT classification.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
tools:
- name: LncCat
  role: Produces the per-transcript prediction scores whose dichotomized confusion
    matrix yields the MCC values
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Underlying classifier of LncCat whose test-set predictions (cutoff 0.5) are
    evaluated by MCC
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:a68a0c1982e4fe14c8cc30ba2fd4ba6c9d8cd1e6d5bf6a21762e49e124839ac0
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# matthews-correlation-coefficient-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute the Matthews Correlation Coefficient (MCC) to evaluate binary lncRNA vs. PCT classification performance in LncCat, both on same-species test sets and to fill a 5x5 cross-species transfer matrix where predictions use a 0.5 score cutoff (score >= 0.5 labeled lncRNA, < 0.5 labeled PCT).

## When to use

Use when benchmarking a binary transcript classifier (lncRNA vs. protein-coding transcript, lncRNA as positive class) on imbalanced five-species datasets and you need a single balanced metric, or when quantifying cross-species model transfer.

## When NOT to use

- When the task requires threshold-free metrics such as ROC-AUC or PR-AUC — MCC here depends on the fixed 0.5 prediction-score cutoff (>= 0.5 = lncRNA, < 0.5 = PCT).
- When input is already an aggregate performance table or published matrix rather than per-transcript predictions — no confusion matrix can be recomputed.
- For multi-class or regression outputs; LncCat's task is binary lncRNA vs. PCT classification.

## Inputs

- Trained LncCat (CatBoost) model per species from https://github.com/a525076133/LncCat
- Test/evaluation transcript FASTA datasets for human, mouse, zebrafish, wheat, and chicken
- Per-transcript feature matrices (codon-related, GC-related, sequence-related, peptide-related, ORF-attention)
- Prediction scores per transcript with lncRNA as the positive class

## Outputs

- MCC value per model-dataset pair (e.g., same-species: human 0.9503, mouse 0.9219, zebrafish 0.8591, wheat 0.8672, chicken 0.9047)
- 5x5 cross-species MCC matrix written to cross_species_mcc_matrix.csv

## How to apply

For each model-dataset pair, apply the trained CatBoost (LncCat) model to the evaluation FASTA-derived feature matrix (codon-related, GC-related, sequence-related, peptide-related, and ORF-attention features extracted with Orfipy using Standard Code transl_table=1), then dichotomize prediction scores at the cutoff 0.5: score >= 0.5 is labeled lncRNA (positive class), < 0.5 is PCT. From the resulting confusion matrix compute MCC = (TP*TN - FP*FN) / sqrt((TP+FP)(TP+FN)(TN+FP)(TN+FN)), which is balanced even when lncRNA and PCT class sizes differ. Compute MCC for all 5x5 combinations of the five species models (human, mouse, zebrafish, wheat, chicken) and the five test datasets; the diagonal gives same-species performance and off-diagonal entries quantify cross-species transfer. Write the result to cross_species_mcc_matrix.csv and validate against reported values.

## Related tools

- **LncCat** (Produces the per-transcript prediction scores whose dichotomized confusion matrix yields the MCC values) — https://github.com/a525076133/LncCat
- **CatBoost** (Underlying classifier of LncCat whose test-set predictions (cutoff 0.5) are evaluated by MCC) — https://github.com/a525076133/LncCat

## Examples

```
python compute_mcc.py --predictions scores.csv --cutoff 0.5 --positive-class lncRNA --out cross_species_mcc_matrix.csv
```

## Evaluation signals

- Diagonal (same-species) MCC values should reproduce reported performance: human 0.9503, mouse 0.9219, zebrafish 0.8591, wheat 0.8672, chicken 0.9047.
- Cross-species animal transfer should be high (e.g., human model MCC 0.93 on mouse, 0.90 on zebrafish).
- Animal models on the wheat dataset should fall near MCC 0.63–0.67 (reported: 0.67, 0.63, 0.66 for human, mouse, zebrafish models).
- cross_species_mcc_matrix.csv must be a symmetric-in-interpretation 5x5 matrix with rows = species models and columns = evaluation datasets.
- MCC values must lie in [-1, 1] and match values recomputed from the confusion matrix obtained with the 0.5 cutoff.

## Limitations

- MCC depends on the fixed 0.5 cutoff; different thresholds change values and are not explored in the article.
- Cross-species transfer degrades sharply for plant data: human, mouse, and zebrafish models reach only MCC 0.67, 0.63, and 0.66 on wheat.
- CatBoost hyperparameter settings and CatBoost/BERT versions are not reported, limiting exact reproduction of the reported MCC values.

## Evidence

- [methods] Cross-species wheat degradation: [evidence span withheld — anchor status: not_found]
