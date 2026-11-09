---
name: greedy-set-cover-optimization
description: Use when predicting novel secondary drug combinations for a chemotherapy-resistant
  cancer, given pharmacogenomics drug-response data (e.g., GDSC1000 IC50) partitioned
  into resistant/sensitive groups and a standard-of-care primary drug query.
when_to_use_negative:
- The goal is testing combinations of more than two secDrugs — the pipeline is currently
  limited to two-drug combinations per the article's discussion.
- Pre-clinical/clinical efficacy validation is required — the pipeline predicts candidates
  only; novel secDrugs such as KIN001-102 and SB505124 lack xenograft or mouse-model
  validation.
- The input is already a ranked combination table with confidence scores rather than
  raw pharmacogenomics drug-response data.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0208
- http://edamontology.org/topic_2640
tools:
- name: secDrug
  role: R package implementing the optimization-regularization greedy set cover algorithm
    to predict secondary drug combinations
  repo: https://github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: Package providing AssignGroup, CombinationDrugs, and PropTest functions used
    to categorize cell lines, run the set-cover search, and score combinations
  repo: https://github.com/Ujjal-Mukherjee/secDrug
- name: R
  role: Execution environment for the secDrug pipeline and its exported functions
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:2e650950aa07a8c2ef49f18a9cea1466f2197588f6b35fc9a3ff3f536661e87c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ujjal-Mukherjee/secDrug
  tools:
  - AssignGroup
  - CombinationDrugMyeloma
  - CombinationDrugs
  - PropTest
  - R
  - secDrug
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0208
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# greedy-set-cover-optimization

## Summary

The secDrug greedy set cover algorithm selects secondary drug combinations that maximally kill resistant cancer cell lines, outputting ranked candidates with proportion-based confidence scores via an R package.

## When to use

Use when predicting novel secondary drug combinations for a chemotherapy-resistant cancer, given pharmacogenomics drug-response data (e.g., GDSC1000 IC50) partitioned into resistant/sensitive groups and a standard-of-care primary drug query.

## When NOT to use

- The goal is testing combinations of more than two secDrugs — the pipeline is currently limited to two-drug combinations per the article's discussion.
- Pre-clinical/clinical efficacy validation is required — the pipeline predicts candidates only; novel secDrugs such as KIN001-102 and SB505124 lack xenograft or mouse-model validation.
- The input is already a ranked combination table with confidence scores rather than raw pharmacogenomics drug-response data.

## Inputs

- GDSC1000 pharmacogenomics drug-response (IC50) data for a cancer cell-line panel
- Query file specifying the cancer type and its standard-of-care primary drug
- List of candidate secondary drugs from the drug-response dataset

## Outputs

- secdrug_combination_ranking.csv — ranked secondary drug combinations with confidence scores in [0,1]
- secdrug_regimen_coverage.csv — top combination regimens with percent cell-line coverage at 2 and 3 secDrugs
- Resistant/sensitive/null grouping of cell lines per AssignGroup

## How to apply

Load GDSC1000 pharmacogenomics drug-response data in R and categorize each cell line's drug IC50 into sensitive (S), resistant (R), or null (N) groups with AssignGroup(x, l). Assemble a query specifying the cancer type (e.g., a GDSC1000 cancer type) and its standard-of-care primary drug, then run CombinationDrugs(dir, file, dest), which executes the optimization-regularization greedy set cover algorithm to select secondary drugs (secDrugs) that cover the maximum number of resistant cell lines. Output is a ranked list of top secondary drug combination candidates with percent cell-line coverage; score top candidates with PropTest(p1, p2, n), yielding proportion-based confidence values in [0,1]. Write results to secdrug_combination_ranking.csv and secdrug_regimen_coverage.csv, verifying rank and confidence columns are populated and all confidence values lie in [0,1].

## Related tools

- **secDrug** (R package implementing the optimization-regularization greedy set cover algorithm to predict secondary drug combinations) — https://github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (Package providing AssignGroup, CombinationDrugs, and PropTest functions used to categorize cell lines, run the set-cover search, and score combinations) — https://github.com/Ujjal-Mukherjee/secDrug
- **R** (Execution environment for the secDrug pipeline and its exported functions)

## Examples

```
CombinationDrugs(dir, file, dest); ranking <- read.csv('secdrug_combination_ranking.csv'); stopifnot(all(ranking$confidence >= 0 & ranking$confidence <= 1))
```

## Evaluation signals

- Reproduce published coverage values: PI only 33.0%, top regimen PI+FK866+17-AAG at 72.2% with 2 secDrugs and 82.5% with 3 secDrugs
- Verify the rank column in secdrug_combination_ranking.csv is populated
- Verify the confidence column is populated and all confidence values lie within [0,1]
- Confirm AssignGroup output partitions each cell line's drug IC50 into exactly one of sensitive (S), resistant (R), or null (N)

## Limitations

- Pipeline is currently limited to two-drug combinations; model systems for testing more than two secDrugs are not yet developed
- Predictions for novel secDrugs (e.g., KIN001-102, SB505124) lack pre-clinical/clinical validation in xenograft or mouse models
- Coverage of predicted secDrugs in wider HMCL panels representing refractory and clonally derived acquired resistant cell lines is not yet established

## Evidence

- [methods] greedy set cover pipeline: [evidence span withheld — anchor status: not_found]
- [methods] PropTest scoring: [evidence span withheld — anchor status: not_found]
- [results] published coverage: [evidence span withheld — anchor status: not_found]
- [abstract] algorithm description: "a computational pipeline that uses pharmacogenomics data-driven optimization-regularization/greedy algorithm to predict novel drugs ("secDrugs") against drug-resistant myeloma"
- [abstract] R package output: "outputs a list of the top drug combination candidates with rank and confidence scores"
