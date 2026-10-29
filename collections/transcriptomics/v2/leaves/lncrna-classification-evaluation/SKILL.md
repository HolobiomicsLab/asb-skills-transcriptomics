---
name: lncrna-classification-evaluation
description: Use when you have LncCat/CatBoost prediction scores for transcripts labeled
  lncRNA or PCT and need to reproduce or benchmark MCC/F1 classification results against
  the paper's fixed 0.5-cutoff baseline.
when_to_use_negative:
- Input is raw transcript FASTA sequences rather than model prediction scores — use
  the full LncCat feature-extraction and classification pipeline first (Orfipy ORF
  extraction, BERT peptide encoding, CatBoost), not just the evaluation step
- You need to distinguish lncRNAs from other ncRNA classes (rRNA, tRNA, etc.) — LncCat
  only classifies lncRNA versus protein-coding transcripts (PCTs)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0099
tools:
- name: LncCat
  role: CatBoost-based classifier whose prediction scores are thresholded to produce
    lncRNA/PCT labels; the subject of the evaluation
  repo: https://github.com/a525076133/LncCat
- name: CatBoost
  role: Gradient-boosting model used by LncCat to generate the prediction scores that
    are evaluated at each cutoff
  repo: https://github.com/a525076133/LncCat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.02.012
    title: 'LncCat: An ORF attention model to identify LncRNA based on ensemble learning
      strategy and fused sequence information'
  source_content_hash: sha256:626856403618530ab9b878213e28c9f42e67c653dd99d5e669fe9526673eb033
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# lncRNA-classification-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate a CatBoost-based lncRNA-vs-PCT classifier (LncCat) by thresholding prediction scores at a fixed cutoff of 0.5 and computing MCC and F1 on held-out test sets from human, mouse, zebrafish, wheat, and chicken, with a cutoff sweep (0.1–0.9) to confirm optima fall between 0.4 and 0.6.

## When to use

Use when you have LncCat/CatBoost prediction scores for transcripts labeled lncRNA or PCT and need to reproduce or benchmark MCC/F1 classification results against the paper's fixed 0.5-cutoff baseline.

## When NOT to use

- Input is raw transcript FASTA sequences rather than model prediction scores — use the full LncCat feature-extraction and classification pipeline first (Orfipy ORF extraction, BERT peptide encoding, CatBoost), not just the evaluation step
- You need to distinguish lncRNAs from other ncRNA classes (rRNA, tRNA, etc.) — LncCat only classifies lncRNA versus protein-coding transcripts (PCTs)

## Inputs

- LncCat (CatBoost) prediction scores for five species test sets (human, mouse, zebrafish, wheat, chicken) from the LncCat repository datasets
- Ground-truth labels per transcript (lncRNA = positive, PCT = negative)

## Outputs

- MCC and F1 per species at the fixed 0.5 cutoff
- cutoff_sweep_results.csv: MCC and F1 per species per cutoff (5 species x 9 cutoffs, 0.1 to 0.9 in steps of 0.1)
- TP/TN/FP/FN counts per species per cutoff

## How to apply

Load LncCat prediction scores for the five species test sets from the a525076133/LncCat repository datasets. Apply the paper's decision rule: prediction score >= 0.5 is labeled lncRNA (positive), < 0.5 is PCT (negative). Compute MCC and F1 treating lncRNAs as the positive class; the reported baseline at cutoff 0.5 is MCC of 0.9503 (human), 0.9219 (mouse), 0.8591 (zebrafish), 0.8672 (wheat), and 0.9047 (chicken). To verify cutoff sensitivity, sweep cutoffs from 0.1 to 0.9 in steps of 0.1, recompute TP/TN/FP/FN, MCC, and F1 at each cutoff, and confirm optima occur between 0.4 and 0.6 with cutoff 0.5 matching the reported Table 5 values. Tabulate results per species per cutoff (5 species x 9 cutoffs) as cutoff_sweep_results.csv.

## Related tools

- **LncCat** (CatBoost-based classifier whose prediction scores are thresholded to produce lncRNA/PCT labels; the subject of the evaluation) — https://github.com/a525076133/LncCat
- **CatBoost** (Gradient-boosting model used by LncCat to generate the prediction scores that are evaluated at each cutoff) — https://github.com/a525076133/LncCat

## Examples

```
python sweep_cutoffs.py --species human mouse zebrafish wheat chicken --cutoffs 0.1 0.2 0.3 0.4 0.5 0.6 0.7 0.8 0.9 --positive lncRNA --out cutoff_sweep_results.csv
```

## Evaluation signals

- MCC at cutoff 0.5 matches Table 5 baseline values: human 0.9503, mouse 0.9219, zebrafish 0.8591 (abstract) / 0.8521 (validation expectation), wheat 0.8672, chicken 0.9047
- Human F1 at cutoff 0.5 equals 0.9742 as reported in the methods comparison
- MCC/F1 optima across the 0.1–0.9 sweep occur with cutoffs between 0.4 and 0.6 for all five species
- cutoff_sweep_results.csv contains exactly 45 rows (5 species x 9 cutoffs) with non-null MCC and F1 values in [-1, 1] and [0, 1] respectively
- Labels assigned by the rule score >= cutoff as lncRNA and < cutoff as PCT reproduce identical TP/TN/FP/FN counts at cutoff 0.5 as the published binary classifications

## Limitations

- No version of CatBoost or BERT is specified in the paper, and CatBoost hyperparameter tuning settings are not reported, limiting exact retraining reproducibility
- The zebrafish baseline MCC is reported as 0.8591 in the abstract but the sweep validation expects 0.8521 at cutoff 0.5 — verify which value corresponds to Table 5
- Web server results are returned only by email rather than directly, and cross-species models perform poorly on wheat (MCC 0.67, 0.63, 0.66), so cutoff behavior may not transfer across species

## Evidence

- [methods] Human, mouse, and zebrafish models perform well on each other's datasets but poorly on wheat, with MCC of 0.67, 0.63, and 0.66: [evidence span withheld — anchor status: not_found]
