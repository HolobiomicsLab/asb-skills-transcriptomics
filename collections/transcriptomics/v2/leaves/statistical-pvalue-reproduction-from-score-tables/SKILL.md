---
name: statistical-pvalue-reproduction-from-score-tables
description: Use when a paper reports a significance p-value (e.g. Wilcoxon p = 1.2e-7)
  comparing score distributions over a labeled table, and you have the score table
  to verify it.
when_to_use_negative:
- The input is raw sequencing data (FASTQ/BAM) rather than a table of already-computed
  prediction scores — use the full TSNAD pipeline instead.
- The published statistic is paired or parametric (e.g. paired Wilcoxon signed-rank,
  t-test) rather than a rank-sum comparison of two labeled groups.
- The score table lacks ground-truth immunogenicity labels, so no two-group split
  for the rank-sum test can be defined.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: TSNAD v2.0
  role: Source of the DeepHLApan binding and immunogenicity scores being statistically
    tested on the TESLA benchmark
  repo: https://github.com/jiujiezz/tsnad
- name: DeepHLApan
  role: Embedded predictor in TSNAD v2.0 that produces the peptide-MHC binding and
    immunogenicity scores compared by the Wilcoxon tests
  repo: https://github.com/jiujiezz/tsnad
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:0b469cac75ac6433e592badafb80d5e9408e9b58bf9304ec8d7eba6c0769f26e
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# statistical-pvalue-reproduction-from-score-tables

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces published p-values by re-running Wilcoxon rank-sum tests on tabulated prediction scores split by ground-truth labels — here, TSNAD v2.0/DeepHLApan immunogenicity scores on 608 TESLA pMHC pairs.

## When to use

Use when a paper reports a significance p-value (e.g. Wilcoxon p = 1.2e-7) comparing score distributions over a labeled table, and you have the score table to verify it.

## When NOT to use

- The input is raw sequencing data (FASTQ/BAM) rather than a table of already-computed prediction scores — use the full TSNAD pipeline instead.
- The published statistic is paired or parametric (e.g. paired Wilcoxon signed-rank, t-test) rather than a rank-sum comparison of two labeled groups.
- The score table lacks ground-truth immunogenicity labels, so no two-group split for the rank-sum test can be defined.

## Inputs

- CSV/TSV table of 608 TESLA pMHC pairs with DeepHLApan binding and immunogenicity scores (TSNAD v2.0 output format)
- Experimental immunogenicity labels from the TESLA benchmark (37 of 608 pairs immunogenic, i.e. 6.1%)

## Outputs

- wilcoxon_results.csv with rows for each test: n pairs (608 and 241), test statistic, and p-value
- Reproduced p-values expected to match p = 1.2e-7 (all pairs) and p = 0.026 (immunogenic score > 0.5 subset)

## How to apply

Load the score table (608 TESLA pMHC pairs with TSNAD v2.0 DeepHLApan binding and immunogenicity scores) into pandas, then split rows into immunogenic vs non-immunogenic groups using the experimental TESLA labels. Run scipy.stats rank-sum (Wilcoxon) tests comparing DeepHLApan score distributions: first on all 608 pairs, then on the subset of 241 pairs whose predicted immunogenic score is > 0.5 (the paper's 'potential neoantigen' cutoff). Record n, statistic, and p-value per test to a results file (e.g. wilcoxon_results.csv). The rationale is that DeepHLApan scores should rank experimentally immunogenic pMHC pairs higher, so significant p-values (reported as p = 1.2e-7 for all pairs and p = 0.026 for the > 0.5 subset) validate that the prediction tool discriminates immunogenicity. Match computed p-values to the published values within numerical tolerance.

## Related tools

- **TSNAD v2.0** (Source of the DeepHLApan binding and immunogenicity scores being statistically tested on the TESLA benchmark) — https://github.com/jiujiezz/tsnad
- **DeepHLApan** (Embedded predictor in TSNAD v2.0 that produces the peptide-MHC binding and immunogenicity scores compared by the Wilcoxon tests) — https://github.com/jiujiezz/tsnad

## Examples

```
import pandas as pd; from scipy.stats import ranksums; df = pd.read_csv('tesla_scores.csv'); p_all = ranksums(df[df.label==1].immunogenic_score, df[df.label==0].immunogenic_score); sub = df[df.immunogenic_score > 0.5]; p_sub = ranksums(sub[sub.label==1].immunogenic_score, sub[sub.label==0].immunogenic_score)
```

## Evaluation signals

- Wilcoxon p-value on all 608 pairs matches 1.2 x 10^-7 within numerical tolerance
- Wilcoxon p-value on the 241 pairs with predicted immunogenic score > 0.5 matches 0.026 within tolerance
- Group sizes are correct: 608 total pairs; the > 0.5 score filter yields exactly 241 pairs
- wilcoxon_results.csv contains n, statistic, and p-value for both tests
- Label sanity check: 37 of 608 TESLA pairs (6.1%) are experimentally immunogenic

## Limitations

- Only 37 of 608 tested TESLA neoantigens (6.1%) were immunogenic, so the immunogenic group is small and p-value reproduction is sensitive to exact tie handling and test variant (ranksums vs mannwhitneyu).
- TSNAD v2.0/DeepHLApan outputs ranking thresholds (score > 0.5) are specific to this predictor; the workflow does not generalize to tools without an equivalent immunogenicity score.
- The paper notes no TCR-pMHC binding prediction model is included, so score-based significance does not explain the mechanism of T-cell response.

## Evidence
