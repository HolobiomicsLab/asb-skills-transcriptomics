---
name: multi-species-benchmark-reproduction
description: Use when validating or benchmarking a lncRNA-vs-PCT classifier (e.g.,
  LncCat/CatBoost) whose published MCC/F1 results on human, mouse, zebrafish, wheat,
  and chicken test sets must be reproduced at a stated prediction-score cutoff, or
  when a cutoff sensitivity sweep is needed.
when_to_use_negative:
- When the goal is to train a new lncRNA classifier rather than reproduce or stress-test
  an existing one's reported benchmark numbers
- When the model under test does not expose continuous prediction scores, since a
  cutoff sweep requires thresholdable scores rather than hard labels
- When evaluating on sORF test datasets only — this skill targets the five long-ORF
  species test sets; sORF benchmarks (human 639 lncRNAs/641 PCTs, mouse 993/846, zebrafish
  497/387) are separate comparisons
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3512
tools:
- name: LncCat
  role: Source of the CatBoost-based lncRNA/PCT prediction scores being benchmarked
    and of the five species test-set datasets
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosted classifier underlying LncCat whose probability outputs are
    thresholded at the 0.5 cutoff and swept for sensitivity analysis
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:e7d2f46b5ac40715781fb5b925de880456a5423e918fd65acb82b3d5a1218449
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
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multi-species-benchmark-reproduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the LncCat benchmark across the five species test sets (human, mouse, zebrafish, wheat, chicken) by thresholding CatBoost prediction scores at the fixed 0.5 cutoff and verifying reported MCC/F1 values, optionally extended by a cutoff sweep from 0.1 to 0.9 to test cutoff sensitivity.

## When to use

Use when validating or benchmarking a lncRNA-vs-PCT classifier (e.g., LncCat/CatBoost) whose published MCC/F1 results on human, mouse, zebrafish, wheat, and chicken test sets must be reproduced at a stated prediction-score cutoff, or when a cutoff sensitivity sweep is needed.

## When NOT to use

- When the goal is to train a new lncRNA classifier rather than reproduce or stress-test an existing one's reported benchmark numbers
- When the model under test does not expose continuous prediction scores, since a cutoff sweep requires thresholdable scores rather than hard labels
- When evaluating on sORF test datasets only — this skill targets the five long-ORF species test sets; sORF benchmarks (human 639 lncRNAs/641 PCTs, mouse 993/846, zebrafish 497/387) are separate comparisons

## Inputs

- LncCat (CatBoost) prediction score files for the five species test sets (human, mouse, zebrafish, wheat, chicken) from https://github.com/a525076133/LncCat
- Ground-truth class labels (lncRNA vs PCT) for each transcript in the test sets
- Reported baseline MCC/F1 values from Table 5 (e.g., human MCC 0.9503, human F1 0.9742)

## Outputs

- cutoff_sweep_results.csv with MCC and F1 per species per cutoff (5 species x 9 cutoffs from 0.1 to 0.9)
- Reproduced MCC/F1 table at the fixed 0.5 cutoff for all five species
- Comparison report against the published benchmark values

## How to apply

Load the LncCat (CatBoost) prediction scores for the five species test sets from the LncCat repository datasets (https://github.com/a525076133/LncCat). For each species, apply the paper's fixed cutoff of 0.5 — prediction score >= 0.5 is labeled lncRNA (positive class), < 0.5 is labeled PCT (negative) — and compute MCC and F1 from TP/TN/FP/FN counts, confirming they match the published values (human 0.9503, mouse 0.9219, zebrafish 0.8591, wheat 0.8672, chicken 0.9047). Then sweep cutoffs from 0.1 to 0.9 in steps of 0.1, recomputing MCC and F1 per species per cutoff (5 species x 9 cutoffs), and write the results to cutoff_sweep_results.csv. The reproduction is validated if MCC/F1 optima fall between cutoffs 0.4 and 0.6 and the cutoff-0.5 values match Table 5; note the paper's findings cite 0.8591 for zebrafish in the abstract while one card records 0.8521 in the workflow, so reconcile against the source table.

## Related tools

- **LncCat** (Source of the CatBoost-based lncRNA/PCT prediction scores being benchmarked and of the five species test-set datasets) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosted classifier underlying LncCat whose probability outputs are thresholded at the 0.5 cutoff and swept for sensitivity analysis) — https://github.com/a525076133/LncCat

## Examples

```
import pandas as pd; df = pd.read_csv('lnc_cat_human_scores.csv'); pred = (df['score'] >= 0.5).astype(int); tp=((pred==1)&(df['label']==1)).sum(); tn=((pred==0)&(df['label']==0)).sum(); fp=((pred==1)&(df['label']==0)).sum(); fn=((pred==0)&(df['label']==1)).sum(); mcc=(tp*tn-fp*fn)/((tp+fp)*(tp+fn)*(tn+fp)*(tn+fn))**0.5
```

## Evaluation signals

- MCC at cutoff 0.5 reproduces Table 5: human 0.9503, mouse 0.9219, zebrafish 0.8591 (workflow card records 0.8521 — reconcile with source table), wheat 0.8672, chicken 0.9047
- Human F1 at cutoff 0.5 reproduces the reported 0.9742 (mouse F1 0.9487)
- MCC/F1 optima across the 0.1–0.9 sweep fall between cutoffs 0.4 and 0.6 for all five species
- cutoff_sweep_results.csv contains exactly 45 rows (5 species x 9 cutoffs) with finite MCC and F1 values in [-1, 1] and [0, 1] respectively
- Labeling rule verified: scores >= cutoff are lncRNA (positive), scores < cutoff are PCT (negative)

## Limitations

- CatBoost hyperparameter tuning settings and CatBoost/BERT versions are not reported, so a full retraining-based reproduction may not be bitwise identical
- The abstract and workflow cards disagree on zebrafish MCC (0.8591 vs 0.8521), so the exact baseline target for zebrafish must be checked against the published Table 5
- Cross-species models transfer poorly — human, mouse, and zebrafish models score MCC of only 0.67, 0.63, and 0.66 on wheat — so per-species in-distribution models are required to reproduce the headline numbers

## Evidence

- [methods] Human, mouse, and zebrafish models perform well on each other's datasets but poorly on wheat, with MCC of 0.67, 0.63, and 0.66: [evidence span withheld — anchor status: not_found]
