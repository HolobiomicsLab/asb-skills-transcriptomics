---
name: confusion-matrix-metric-computation
description: Use when binary classification predictions (lncRNA vs PCT) with raw scores
  need to be converted into confusion-matrix metrics such as MCC, F1, PRE, SEN, SPE,
  ACC, and AUC for benchmark comparison against published LncCat values.
when_to_use_negative:
- When raw prediction scores are not available and only final class labels exist,
  since AUC and any cutoff sweep require scores
- When the task is multi-class or regression evaluation — this skill is defined for
  binary lncRNA vs PCT classification only
- When evaluating cross-species transfer performance of separately trained species
  models, which requires a different protocol than same-split test evaluation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0080
tools:
- name: LncCat
  role: CatBoost-based classifier producing the prediction scores that are thresholded
    at 0.5 to build the confusion matrix
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosting model used to construct the prediction model whose test-set
    outputs feed the metric computation
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:33e1f2608a07068e12de30b0ec122767e6336c8d1bbc70b0adab75bb40321662
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# confusion-matrix-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute PRE, SEN, SPE, ACC, F1, MCC, and AUC from TP/TN/FP/FN counts obtained by thresholding LncCat prediction scores at the fixed 0.5 cutoff, treating lncRNAs as the positive class and PCTs as the negative class, to benchmark lncRNA identification across species and sORF datasets.

## When to use

Use when binary classification predictions (lncRNA vs PCT) with raw scores need to be converted into confusion-matrix metrics such as MCC, F1, PRE, SEN, SPE, ACC, and AUC for benchmark comparison against published LncCat values.

## When NOT to use

- When raw prediction scores are not available and only final class labels exist, since AUC and any cutoff sweep require scores
- When the task is multi-class or regression evaluation — this skill is defined for binary lncRNA vs PCT classification only
- When evaluating cross-species transfer performance of separately trained species models, which requires a different protocol than same-split test evaluation

## Inputs

- LncCat (CatBoost) raw prediction scores for the five species test sets (human, mouse, zebrafish, wheat, chicken) from the LncCat repository deposit
- Prediction scores for the integrated long-ORF test dataset
- Prediction scores for the sORF-integrated test datasets (human 639 lncRNAs/641 PCTs, mouse 993 lncRNAs/846 PCTs, zebrafish 497 lncRNAs/387 PCTs)
- Ground-truth class labels (lncRNA = positive, PCT = negative)

## Outputs

- Confusion-matrix counts (TP, TN, FP, FN) per test set at the 0.5 cutoff
- Metrics table with PRE, SEN, SPE, ACC, F1, MCC, and AUC per species/test set (e.g., integrated_model_evaluation.csv)
- Optionally, a cutoff sweep tabulating MCC and F1 at cutoffs 0.1-0.9 in steps of 0.1 (cutoff_sweep_results.csv)

## How to apply

First threshold prediction scores at the fixed cutoff of 0.5: scores >= 0.5 are labeled lncRNA (positive) and scores < 0.5 are labeled PCT (negative). Derive TP/TN/FP/FN counts with lncRNAs as the positive class, then compute PRE, SEN, SPE, ACC, F1, MCC, and AUC on each test set. Apply this per species test set (human, mouse, zebrafish, wheat, chicken), to the integrated long-ORF test set, and to the sORF test sets. Compare the resulting metrics against the article's Table benchmarks: at cutoff 0.5 the per-species MCCs are 0.9503 (human), 0.9219 (mouse), 0.8591 (zebrafish), 0.8672 (wheat), and 0.9047 (chicken); the integrated long-ORF MCC should be 0.9236 (F1 0.9537, Table 14) and the sORF-integrated MCC 0.8346 (F1 0.9245, Table 15).

## Related tools

- **LncCat** (CatBoost-based classifier producing the prediction scores that are thresholded at 0.5 to build the confusion matrix) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosting model used to construct the prediction model whose test-set outputs feed the metric computation) — https://github.com/a525076133/LncCat

## Examples

```
preds = model.predict_proba(X_test)[:,1]; yhat = (preds >= 0.5).astype(int); tp = ((yhat==1)&(y==1)).sum(); tn = ((yhat==0)&(y==0)).sum(); fp = ((yhat==1)&(y==0)).sum(); fn = ((yhat==0)&(y==1)).sum(); mcc = (tp*tn-fp*fn)/np.sqrt((tp+fp)*(tp+fn)*(tn+fp)*(tn+fn))
```

## Evaluation signals

- MCC at cutoff 0.5 must match Table 5 values: human 0.9503, mouse 0.9219, zebrafish 0.8591, wheat 0.8672, chicken 0.9047
- Integrated long-ORF test MCC must match the reported 0.9236 (F1 0.9537, Table 14)
- sORF-integrated test MCC must match the reported 0.8346 (F1 0.9245, Table 15)
- A cutoff sweep from 0.1 to 0.9 must show MCC/F1 optima occurring between 0.4 and 0.6, with cutoff 0.5 reproducing the benchmark values
- Human dataset F1 should reach 0.9742, exceeding the runner-up PLEK (F1 0.9645)

## Limitations

- CatBoost hyperparameter tuning settings and software versions are not reported, so exact score reproduction may not be possible
- Species may lack annotations, making it impossible to build models and compute metrics for all species
- Metric values depend on the 70/10/20 train/validation/test split and CD-HIT 80% redundancy reduction of the source datasets

## Evidence

- [readme] sORF benchmark improvements: [evidence span withheld — anchor status: external_doc]
