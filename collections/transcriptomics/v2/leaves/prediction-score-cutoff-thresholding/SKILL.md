---
name: prediction-score-cutoff-thresholding
description: Use when converting LncCat/CatBoost prediction probabilities into binary
  lncRNA vs PCT labels, or when sweeping cutoffs (e.g. 0.1-0.9) to check that reported
  MCC/F1 benchmarks hold at the fixed 0.5 cutoff.
when_to_use_negative:
- Input is already a binary lncRNA/PCT label file rather than continuous prediction
  scores — thresholding is not applicable.
- The goal is to compare whole coding-potential methods (CPAT, CNCI, PLEK, CPC2, LncFinder,
  CPPred, mRNN, RNAsamba) rather than to re-derive LncCat's own binary calls from
  its scores.
- Evaluating on sORF test datasets where a different, unreported cutoff calibration
  may apply and the 0.5 baseline is not established.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: LncCat
  role: Source of the prediction scores that are thresholded to make lncRNA vs PCT
    calls
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Underlying classifier whose class probabilities are compared against the 0.5
    cutoff
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:2ba1ab2777462418d16e281de15c0d179c2c608d826894a1bda9aabe04f6d58d
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# prediction-score-cutoff-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts LncCat (CatBoost) prediction scores into binary lncRNA vs PCT calls by thresholding at a fixed cutoff of 0.5 (score >= 0.5 -> lncRNA, < 0.5 -> PCT), and supports cutoff sweeps to verify reported MCC/F1 benchmarks are tied to that fixed cutoff.

## When to use

Use when converting LncCat/CatBoost prediction probabilities into binary lncRNA vs PCT labels, or when sweeping cutoffs (e.g. 0.1-0.9) to check that reported MCC/F1 benchmarks hold at the fixed 0.5 cutoff.

## When NOT to use

- Input is already a binary lncRNA/PCT label file rather than continuous prediction scores — thresholding is not applicable.
- The goal is to compare whole coding-potential methods (CPAT, CNCI, PLEK, CPC2, LncFinder, CPPred, mRNN, RNAsamba) rather than to re-derive LncCat's own binary calls from its scores.
- Evaluating on sORF test datasets where a different, unreported cutoff calibration may apply and the 0.5 baseline is not established.

## Inputs

- Per-transcript LncCat (CatBoost) prediction probability scores for the five species test sets (human, mouse, zebrafish, wheat, chicken) from the LncCat repository datasets
- Ground-truth class labels (lncRNA/PCT) for each transcript, e.g. from the RefSeq/GENCODE/Ensembl-derived test sets

## Outputs

- Binary lncRNA/PCT classification per transcript at the fixed 0.5 cutoff
- Table of MCC and F1 per species per cutoff (5 species x 9 cutoffs, 0.1-0.9), e.g. cutoff_sweep_results.csv

## How to apply

Obtain per-transcript prediction scores from the LncCat (CatBoost) model for each species test set (human, mouse, zebrafish, wheat, chicken) using the LncCat repository datasets. Apply the article's decision rule: prediction score >= 0.5 labels the transcript lncRNA (positive class), < 0.5 labels it PCT (negative class). To validate that the reported benchmarks are cutoff-dependent, re-threshold the same scores at cutoffs from 0.1 to 0.9 in steps of 0.1 and recompute TP/TN/FP/FN at each cutoff with lncRNAs as the positive class. Compute MCC and F1 per species per cutoff (5 species x 9 cutoffs) and tabulate them (e.g. cutoff_sweep_results.csv). Correctness is judged by whether MCC at cutoff 0.5 reproduces the paper's values: human 0.9503, mouse 0.9219, zebrafish 0.8591 (Table 5 reports 0.8521), wheat 0.8672, chicken 0.9047, and whether MCC/F1 optima fall in the 0.4-0.6 range.

## Related tools

- **LncCat** (Source of the prediction scores that are thresholded to make lncRNA vs PCT calls) — https://github.com/a525076133/LncCat
- **CatBoost** (Underlying classifier whose class probabilities are compared against the 0.5 cutoff) — https://github.com/a525076133/LncCat

## Examples

```
scores = model.predict_proba(X_test)[:,1]; y_pred = ["lncRNA" if s >= 0.5 else "PCT" for s in scores]; mcc = matthews_corrcoef(y_true, y_pred)
```

## Evaluation signals

- At cutoff 0.5, recomputed MCC must match the published values: human 0.9503, mouse 0.9219, zebrafish 0.8591 (Table 5: 0.8521), wheat 0.8672, chicken 0.9047.
- Human F1 at cutoff 0.5 should reproduce 0.9742 and mouse F1 0.9487 from the paper's benchmark tables.
- MCC and F1 optima across the 0.1-0.9 sweep must fall in the 0.4-0.6 cutoff window for all five species.
- The output table must contain the full 5 species x 9 cutoffs grid with no missing cells, and all metrics must lie in [-1, 1] (MCC) and [0, 1] (F1).

## Limitations

- CatBoost hyperparameters and software versions are not reported in the paper, so re-trained scores may not exactly reproduce the published ones.
- The fixed 0.5 cutoff is only validated on the five species test sets; cutoff sensitivity on the sORF test datasets is not characterized by the paper.
- No public code repository ships the full LncCat model itself (only the web server at http://cczubio.top/lnccat/), limiting score regeneration.

## Evidence
