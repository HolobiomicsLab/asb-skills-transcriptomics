---
name: neoantigen-immunogenicity-score-analysis
description: Use when you have a table of predicted pMHC scores (e.g., DeepHLApan
  binding and immunogenicity scores) with experimental immunogenicity labels, and
  need to test whether the score separates immunogenic from non-immunogenic peptides.
when_to_use_negative:
- Input is raw WES/WGS or RNA-Seq data requiring the full TSNAD v2.0 prediction pipeline
  (Trimmomatic, BWA, GATK Mutect2, OptiType, DeepHLApan) rather than score validation
- Data lacks experimental immunogenicity labels, so immunogenic vs non-immunogenic
  groups cannot be formed for a rank-sum comparison
- Comparing paired or matched score distributions, where a Wilcoxon rank-sum (independent-samples)
  test is inappropriate
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3360
tools:
- name: TSNAD v2.0
  role: Source of the predicted binding and immunogenicity scores being validated
    on the TESLA benchmark
  repo: https://github.com/jiujiezz/tsnad
- name: DeepHLApan
  role: Embedded MHC binding and immunogenicity prediction model whose scores are
    compared between immunogenic and non-immunogenic groups
- name: scipy.stats
  role: Runs the Wilcoxon rank-sum tests on the full 608-pair set and the 241-pair
    score>0.5 subset
- name: pandas
  role: Loads the 608-pair TESLA score table and splits it into immunogenic vs non-immunogenic
    groups
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.08.016
    title: 'TSNAD v2.0: A one-stop software solution for tumor-specific neoantigen
      detection'
  source_content_hash: sha256:6461bbae3a1167ddec8e567a69b4d57af80cf6dd277904b3eb93d62b7541e9ed
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
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# neoantigen-immunogenicity-score-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical validation of neoantigen prediction scores by running Wilcoxon rank-sum tests that compare immunogenic versus non-immunogenic peptide-MHC (pMHC) pairs, as demonstrated for TSNAD v2.0/DeepHLApan scores on the 608-pair TESLA benchmark.

## When to use

Use when you have a table of predicted pMHC scores (e.g., DeepHLApan binding and immunogenicity scores) with experimental immunogenicity labels, and need to test whether the score separates immunogenic from non-immunogenic peptides.

## When NOT to use

- Input is raw WES/WGS or RNA-Seq data requiring the full TSNAD v2.0 prediction pipeline (Trimmomatic, BWA, GATK Mutect2, OptiType, DeepHLApan) rather than score validation
- Data lacks experimental immunogenicity labels, so immunogenic vs non-immunogenic groups cannot be formed for a rank-sum comparison
- Comparing paired or matched score distributions, where a Wilcoxon rank-sum (independent-samples) test is inappropriate

## Inputs

- Table (CSV/pandas DataFrame) of TESLA pMHC pairs with DeepHLApan/TSNAD v2.0 binding and immunogenicity scores
- Experimental immunogenicity labels for the 608 TESLA predicted neoantigens (TESLA dataset, Synapse syn21048999)

## Outputs

- wilcoxon_results.csv with n pairs, statistic, and p-value for the full 608-pair test and the 241-pair (immunogenic score > 0.5) subset test

## How to apply

Load the labeled pMHC score table (e.g., the 608 TESLA pairs scored by TSNAD v2.0's DeepHLApan module) with pandas, and split it into immunogenic and non-immunogenic groups using the experimental TESLA immunogenicity labels. Run a Wilcoxon rank-sum test (scipy.stats) comparing the score distributions between groups on the full dataset, then repeat the test on a restricted subset applying the paper's potential-neoantigen threshold (immunogenic score > 0.5), which reduces the TESLA set to 241 pairs. Record n pairs, test statistic, and p-value for both tests (expected: p = 1.2e-7 for all 608 pairs; p = 0.026 for the 241-pair subset) and write results to wilcoxon_results.csv. The rationale is that DeepHLApan models both peptide-MHC binding and immunogenicity, so a significant rank-sum result confirms the predicted scores discriminate the 37 immunogenic TESLA neoantigens from non-immunogenic ones.

## Related tools

- **TSNAD v2.0** (Source of the predicted binding and immunogenicity scores being validated on the TESLA benchmark) — https://github.com/jiujiezz/tsnad
- **DeepHLApan** (Embedded MHC binding and immunogenicity prediction model whose scores are compared between immunogenic and non-immunogenic groups)
- **scipy.stats** (Runs the Wilcoxon rank-sum tests on the full 608-pair set and the 241-pair score>0.5 subset)
- **pandas** (Loads the 608-pair TESLA score table and splits it into immunogenic vs non-immunogenic groups)

## Examples

```
from scipy.stats import ranksums; stat, p = ranksums(df[df.label=='immunogenic'].immunogenic_score, df[df.label=='non-immunogenic'].immunogenic_score)
```

## Evaluation signals

- Wilcoxon p-value on all 608 TESLA pairs matches the reported 1.2e-7 within numerical tolerance
- Wilcoxon p-value on the 241 pairs with predicted immunogenic score > 0.5 matches the reported 0.026
- Subset restriction (immunogenic score > 0.5) yields exactly 241 pairs out of 608
- wilcoxon_results.csv contains n pairs, statistic, and p-value rows for both the full and subset tests
- Group split reproduces the TESLA composition of 37 (6.1%) immunogenic out of 608 tested neoantigens

## Limitations

- Only 37 (6.1%) of the 608 TESLA neoantigens were experimentally immunogenic, so the immunogenic group is small and the test is imbalanced
- TSNAD v2.0 lacks a TCR-pMHC binding prediction model and TCR sequencing analysis, so the scores capture only MHC binding and immunogenicity, not T-cell receptor interaction
- Validation applies the paper's own threshold (immunogenic score > 0.5); conclusions may not transfer to other scoring cutoffs or non-TESLA datasets

## Evidence

- [other] Run a Wilcoxon rank-sum test (scipy.stats) comparing the score distributions between immunogenic and non-immunogenic groups: [evidence span withheld — anchor status: not_found]
- [results] we obtained the WES, tumor RNA-Seq, and clinical-grade HLA typing of five patients ... from Synapse with identifier syn21048999: [evidence span withheld — anchor status: not_found]
