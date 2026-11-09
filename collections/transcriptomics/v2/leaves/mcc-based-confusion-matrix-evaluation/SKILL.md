---
name: mcc-based-confusion-matrix-evaluation
description: Use when a binary sequence classifier (e.g. lncRNA vs protein-coding
  transcript) produces probability scores on a test set and you must compute MCC and
  companion metrics against published benchmark values.
when_to_use_negative:
- Input is raw multi-class or regression output rather than binary lncRNA/PCT predictions
  with scores comparable to a 0.5 cutoff
- Predictions come from a model not trained on the same CD-HIT-80%-filtered, >=200
  nt, 70/10/20-split data, so direct MCC comparison against the published table is
  invalid
- You only need ranking quality (AUC) and class prevalence prevents interpreting fixed-cutoff
  confusion-matrix metrics meaningfully
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: LncCat
  role: Source of predicted probability scores that are binarized at 0.5 and evaluated
    with MCC and companion metrics
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosting classifier whose test-set predictions feed the confusion-matrix
    computation
  repo: https://github.com/a525076133/LncCat
- name: RNAsamba
  role: Competing method whose test-set classifications provide baseline MCC values
    for comparison
  repo: http://github.com/apcamargo/RNAsamba
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:1ffd31b010096c85014271c3f9d1e616d7f08464e3bc117b68c0f50c6ee53329
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
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mcc-based-confusion-matrix-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate lncRNA/PCT classifier predictions on held-out test sets by binarizing predicted scores at a fixed 0.5 cutoff and computing a confusion-matrix-derived panel of seven metrics (ACC, SEN, SPE, F1, PRE, MCC, AUC), with MCC as the headline score for cross-method and cross-species benchmark comparison.

## When to use

Use when a binary sequence classifier (e.g. lncRNA vs protein-coding transcript) produces probability scores on a test set and you must compute MCC and companion metrics against published benchmark values.

## When NOT to use

- Input is raw multi-class or regression output rather than binary lncRNA/PCT predictions with scores comparable to a 0.5 cutoff
- Predictions come from a model not trained on the same CD-HIT-80%-filtered, >=200 nt, 70/10/20-split data, so direct MCC comparison against the published table is invalid
- You only need ranking quality (AUC) and class prevalence prevents interpreting fixed-cutoff confusion-matrix metrics meaningfully

## Inputs

- Per-transcript predicted probability scores for the species test set (output of CatBoost/LncCat model)
- Ground-truth binary labels (lncRNA vs PCT) for the 20% test split
- Optional: raw score probabilities for AUC computation

## Outputs

- Per-species metrics table (lnccat_species_metrics.csv) with ACC, SEN, SPE, F1, PRE, MCC, AUC
- Confusion matrix (TP/FP/TN/FN) per species dataset
- Improvement percentages of MCC over each of the eight competing methods

## How to apply

Binarize each test-set prediction with the fixed cutoff of 0.5: score >= 0.5 is predicted lncRNA, < 0.5 is predicted PCT. From the resulting confusion matrix compute ACC, SEN (recall), SPE, PRE, F1, and MCC (Matthew's Correlation Coefficient), and compute AUC from the raw score distribution. Compute metrics per species dataset (human, mouse, zebrafish, wheat, chicken) rather than pooled, so results can be compared cell-by-cell with published baselines such as PLEK, CPAT, CNCI, CPC2, LncFinder, CPPred, mRNN, and RNAsamba. Judge correctness by checking reproduced MCC against published values: 0.9503 (human), 0.9219 (mouse), 0.8591 (zebrafish), 0.8672 (wheat), 0.9047 (chicken), with expected human F1 of 0.9742. MCC is preferred as the headline metric because it is robust to class imbalance in the 70/10/20-split, CD-HIT-80%-filtered datasets.

## Related tools

- **LncCat** (Source of predicted probability scores that are binarized at 0.5 and evaluated with MCC and companion metrics) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosting classifier whose test-set predictions feed the confusion-matrix computation) — https://github.com/a525076133/LncCat
- **RNAsamba** (Competing method whose test-set classifications provide baseline MCC values for comparison) — http://github.com/apcamargo/RNAsamba

## Examples

```
from sklearn.metrics import matthews_corrcoef, f1_score, roc_auc_score; y_pred = (scores >= 0.5).astype(int); print(matthews_corrcoef(y_true, y_pred))
```

## Evaluation signals

- Reproduced human-dataset MCC equals 0.9503 and F1 equals 0.9742 within rounding
- Reproduced MCC values fall within the published improvement ranges over competitors (e.g. 1.90%~7.82% on human, 6.11%~21.50% on zebrafish)
- All five species MCCs land at 0.9503, 0.9219, 0.8591, 0.8672, 0.9047 for human, mouse, zebrafish, wheat, chicken respectively
- Binarization uses exactly the 0.5 cutoff (score >= 0.5 = lncRNA, < 0.5 = PCT), matching the article's stated threshold
- Output CSV contains all seven metrics (ACC, SEN, SPE, F1, PRE, MCC, AUC) for each of the five species

## Limitations

- The 0.5 cutoff is fixed rather than tuned per species, which may understate performance that a threshold sweep would reveal
- Cross-species transfer is poor: human, mouse, and zebrafish models score MCC of only 0.67, 0.63, and 0.66 on wheat, so same-species benchmark values cannot be generalized
- CatBoost hyperparameter tuning settings and software versions are not reported, limiting exact reproducibility of the scores being evaluated

## Evidence
