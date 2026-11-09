---
name: tesla-benchmark-pairwise-group-comparison
description: Use when you have a table of TESLA pMHC pairs with predicted binding/immunogenicity
  scores and experimental immunogenicity labels, and need a significance test comparing
  score distributions between groups.
when_to_use_negative:
- The data are not peptide-MHC pairs scored by DeepHLApan/TSNAD v2.0 with paired experimental
  immunogenicity labels — the test requires labeled groups to compare
- You are scoring neoantigen candidates in a discovery pipeline (use the binding score
  >0.5 and immunogenic score >0.5 filter instead of a group-comparison test)
- The score distributions require paired/matched samples — the Wilcoxon rank-sum (Mann-Whitney-type)
  test used here assumes independent groups
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3407
tools:
- name: TSNAD v2.0
  role: Pipeline whose DeepHLApan-derived binding and immunogenicity scores are compared
    between immunogenic and non-immunogenic TESLA pairs
  repo: https://github.com/jiujiezz/tsnad
- name: DeepHLApan
  role: Embedded predictor supplying the peptide-MHC binding and immunogenicity scores
    under statistical evaluation
  repo: https://github.com/jiujiezz/tsnad
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:9119830f0bb87aaa9d1fe2c62b4e2e4dc39dcd388af0121d70280b2a8aca08ed
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jiujiezz/tsnad
  repo_url: https://github.com/jiujiezz/tsnad
  tools:
  - DeepHLApan
  - TSNAD v2.0
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# tesla-benchmark-pairwise-group-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical skill for testing whether TSNAD v2.0 / DeepHLApan predicted immunogenicity scores discriminate immunogenic from non-immunogenic peptide-MHC pairs on the 608-pair TESLA benchmark, using Wilcoxon rank-sum tests on the full set and on the immunogenic-score>0.5 subset.

## When to use

Use when you have a table of TESLA pMHC pairs with predicted binding/immunogenicity scores and experimental immunogenicity labels, and need a significance test comparing score distributions between groups.

## When NOT to use

- The data are not peptide-MHC pairs scored by DeepHLApan/TSNAD v2.0 with paired experimental immunogenicity labels — the test requires labeled groups to compare
- You are scoring neoantigen candidates in a discovery pipeline (use the binding score >0.5 and immunogenic score >0.5 filter instead of a group-comparison test)
- The score distributions require paired/matched samples — the Wilcoxon rank-sum (Mann-Whitney-type) test used here assumes independent groups

## Inputs

- CSV/table of 608 TESLA pMHC pairs with TSNAD v2.0 (DeepHLApan) binding and immunogenicity scores
- Experimental immunogenicity labels from the TESLA benchmark (Synapse syn21048999-derived validation data)

## Outputs

- wilcoxon_results.csv with, per test: number of pairs (608 and 241), test statistic, and p-value (expected 1.2e-7 and 0.026)

## How to apply

Load the 608 TESLA pMHC pairs with their DeepHLApan-derived TSNAD v2.0 binding and immunogenicity scores into pandas, and split them into immunogenic vs non-immunogenic groups using the experimental TESLA labels (37 of 608, i.e. 6.1%, were immunogenic). Run a Wilcoxon rank-sum test (scipy.stats) comparing score distributions across all 608 pairs and record n, statistic, and p-value. Then apply the paper's candidate-neoantigen threshold — immunogenic score > 0.5 — to restrict to the 241 pairs passing that filter, and repeat the rank-sum test on this subset. Write both results to wilcoxon_results.csv and validate against the reported values of p = 1.2e-7 (all 608 pairs) and p = 0.026 (241 pairs with immunogenic score > 0.5).

## Related tools

- **TSNAD v2.0** (Pipeline whose DeepHLApan-derived binding and immunogenicity scores are compared between immunogenic and non-immunogenic TESLA pairs) — https://github.com/jiujiezz/tsnad
- **DeepHLApan** (Embedded predictor supplying the peptide-MHC binding and immunogenicity scores under statistical evaluation) — https://github.com/jiujiezz/tsnad

## Examples

```
from scipy.stats import ranksums; import pandas as pd; df = pd.read_csv('tesla_608.csv'); p_all = ranksums(df[df.immunogenic==1].score, df[df.immunogenic==0].score).pvalue; sub = df[df.immunogenic_score>0.5]; p_sub = ranksums(sub[sub.immunogenic==1].score, sub[sub.immunogenic==0].score).pvalue
```

## Evaluation signals

- Wilcoxon p-value on all 608 TESLA pairs matches the reported 1.2e-7 within numerical tolerance
- Wilcoxon p-value on the 241 pairs with immunogenic score > 0.5 matches the reported 0.026 within numerical tolerance
- Group counts reconcile: 608 total pairs tested, 37 (6.1%) experimentally immunogenic, 241 pairs above the 0.5 immunogenic-score threshold
- wilcoxon_results.csv contains n, statistic, and p-value rows for both the full-set and score>0.5 subset tests
- Immunogenic group shows significantly higher TSNAD v2.0 predicted scores than the non-immunogenic group (one-sided direction check)

## Limitations

- Only 37 of 608 TESLA pairs (6.1%) were experimentally immunogenic, so the non-immunogenic group heavily outnumbers the immunogenic group
- The subset restriction to immunogenic score > 0.5 discards most pairs, weakening statistical power relative to the full 608-pair test
- TSNAD v2.0 lacks a TCR-pMHC binding prediction model, so scores do not directly capture T-cell receptor interaction mechanisms

## Evidence
