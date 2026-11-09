---
name: wilcoxon-rank-sum-significance-testing
description: Use when you have per-pair predicted scores (e.g. DeepHLApan binding/immunogenicity
  scores) plus binary experimental labels and must test whether the scores separate
  the labeled groups non-parametrically.
when_to_use_negative:
- Data are paired/matched measurements rather than two independent groups — use a
  paired Wilcoxon signed-rank test instead
- Scores are already validated against the same labels and only thresholded neoantigen
  calls are needed (binding score >0.5 and immunogenic score >0.5 filter), with no
  group-difference hypothesis to test
- Input is raw WES/WGS/RNA-seq data requiring the TSNAD prediction pipeline first,
  rather than a scored pMHC table
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: TSNAD v2.0
  role: Source of the predicted DeepHLApan binding and immunogenicity scores being
    compared between immunogenic and non-immunogenic TESLA pMHC pairs
  repo: https://github.com/jiujiezz/tsnad
- name: DeepHLApan
  role: Scoring model inside TSNAD v2.0 that produces the per-pMHC binding and immunogenicity
    scores fed into the Wilcoxon tests
  repo: https://github.com/jiujiezz/tsnad
- name: pandas
  role: Loading and splitting the 608-pair scored TESLA table into immunogenic vs
    non-immunogenic groups
- name: scipy.stats
  role: Running the Wilcoxon rank-sum tests on all 608 pairs and the 241-pair score>0.5
    subset
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:57ae265685fb9b776f0d841bede7044bb08a7718c37b5ab70c724a98035974b2
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
  - pandas
  - scipy.stats
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# wilcoxon-rank-sum-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Wilcoxon rank-sum (Mann–Whitney U) tests to compare predicted neoantigen score distributions between immunogenic and non-immunogenic pMHC pairs, as demonstrated when validating TSNAD v2.0's DeepHLApan immunogenicity scores on the 608-pair TESLA benchmark.

## When to use

Use when you have per-pair predicted scores (e.g. DeepHLApan binding/immunogenicity scores) plus binary experimental labels and must test whether the scores separate the labeled groups non-parametrically.

## When NOT to use

- Data are paired/matched measurements rather than two independent groups — use a paired Wilcoxon signed-rank test instead
- Scores are already validated against the same labels and only thresholded neoantigen calls are needed (binding score >0.5 and immunogenic score >0.5 filter), with no group-difference hypothesis to test
- Input is raw WES/WGS/RNA-seq data requiring the TSNAD prediction pipeline first, rather than a scored pMHC table

## Inputs

- CSV/table of 608 TESLA pMHC pairs with TSNAD v2.0 DeepHLApan binding and immunogenicity scores
- Experimental immunogenicity labels from the TESLA benchmark (37 of 608 pairs immunogenic)
- Score threshold defining the subset (immunogenic score > 0.5, yielding 241 pairs)

## Outputs

- wilcoxon_results.csv with n pairs, statistic, and p-value for the all-pairs test (expected p = 1.2e-7) and the score>0.5 subset test (expected p = 0.026)

## How to apply

Load the scored pMHC table (e.g. 608 TESLA peptide-MHC pairs with TSNAD v2.0 DeepHLApan binding and immunogenicity scores) with pandas, and split rows into immunogenic vs non-immunogenic groups using the experimental TESLA labels. Run scipy.stats Wilcoxon rank-sum tests comparing score distributions between the two groups, first on all 608 pairs and then on the restricted subset of 241 pairs with predicted immunogenic score > 0.5 (the threshold the paper defines for potential neoantigens). Record n, statistic, and p-value for each test to an output file such as wilcoxon_results.csv. In the reference application the p-values were p = 1.2 × 10^-7 (all 608 pairs) and p = 0.026 (241 pairs with immunogenic score > 0.5), confirming scores are significantly higher for immunogenic pairs.

## Related tools

- **TSNAD v2.0** (Source of the predicted DeepHLApan binding and immunogenicity scores being compared between immunogenic and non-immunogenic TESLA pMHC pairs) — https://github.com/jiujiezz/tsnad
- **DeepHLApan** (Scoring model inside TSNAD v2.0 that produces the per-pMHC binding and immunogenicity scores fed into the Wilcoxon tests) — https://github.com/jiujiezz/tsnad
- **pandas** (Loading and splitting the 608-pair scored TESLA table into immunogenic vs non-immunogenic groups)
- **scipy.stats** (Running the Wilcoxon rank-sum tests on all 608 pairs and the 241-pair score>0.5 subset)

## Examples

```
from scipy.stats import mannwhitneyu; U, p = mannwhitneyu(df[df.label==1].score, df[df.label==0].score, alternative='two-sided')
```

## Evaluation signals

- p-value for the all-608-pairs test matches the reported p = 1.2e-7 within numerical tolerance
- p-value for the 241-pair subset (immunogenic score > 0.5) matches the reported p = 0.026 within tolerance
- Group sizes reconstruct correctly: 608 total TESLA pairs, subset restricted to 241 pairs with immunogenic score > 0.5, 37 (6.1%) experimentally immunogenic
- wilcoxon_results.csv contains rows for both tests with n, statistic, and p-value fields populated
- Immunogenic group shows significantly higher predicted scores than non-immunogenic group (direction of the statistic consistent with the reported finding)

## Limitations

- Only 37 of 608 TESLA pairs (6.1%) are experimentally immunogenic, so the positive class is heavily imbalanced and the subset test rests on just 241 pairs
- TSNAD v2.0 lacks a TCR-pMHC binding prediction model, so significance of score separation does not explain the mechanism by which pMHC induces T-cell response
- Validation requires the exact TESLA benchmark labels and scores; results will not reproduce on other datasets or score sources

## Evidence

- [abstract] Score source (DeepHLApan in TSNAD v2.0): [evidence span withheld — anchor status: not_found]
