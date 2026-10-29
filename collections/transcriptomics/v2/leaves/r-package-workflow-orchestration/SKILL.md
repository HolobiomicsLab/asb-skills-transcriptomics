---
name: r-package-workflow-orchestration
description: Use when the task requires chaining R package exports (CombinationDrugs,
  AssignGroup, PropTest) to generalize a cancer/drug-agnostic secondary-drug-combination
  pipeline beyond the paper's original myeloma demonstration.
when_to_use_negative:
- When the goal is testing combinations of more than two secDrugs — the pipeline is
  currently limited to two-drug combinations
- When the query requires pre-clinical/clinical validation (xenograft or mouse models)
  — the package only predicts and scores candidate combinations computationally
- When input drug-response data cannot be partitioned into resistant/sensitive groups,
  since AssignGroup(x, l) depends on that dichotomy
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: secDrug
  role: Core R package implementing the greedy set cover pipeline that predicts novel
    synergistic secondary drug combinations
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: R
  role: Runtime environment hosting the universally applicable R-software package
    for predicting novel secondary therapies
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugs
  role: Exported function (dir, file, dest) that executes the greedy set cover pipeline
    to generate ranked secondary drug combination candidates
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: AssignGroup
  role: Exported function (x, l) that partitions drug-response data into resistant
    and sensitive groups before combination search
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: PropTest
  role: Exported function (p1, p2, n) yielding proportion-based confidence values
    in [0,1] for top combinations
  repo: github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: Demonstration dataset/case for the myeloma-resistant-cell use case
  repo: github.com/Ujjal-Mukherjee/secDrug
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:f31b445042237b6e266fda019f5d4f2d0bb8efbf31ad4983d4f7f6c40311ed60
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AssignGroup
  - CombinationDrugMyeloma
  - CombinationDrugs
  - PropTest
  - R
  - secDrug
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# r-package-workflow-orchestration

## Summary

Orchestrate the secDrug R package's exported functions into a reproducible pipeline that turns any cancer-type/primary-drug query into ranked, confidence-scored secondary drug combinations using a greedy set cover algorithm on pharmacogenomics drug-response data.

## When to use

Use when the task requires chaining R package exports (CombinationDrugs, AssignGroup, PropTest) to generalize a cancer/drug-agnostic secondary-drug-combination pipeline beyond the paper's original myeloma demonstration.

## When NOT to use

- When the goal is testing combinations of more than two secDrugs — the pipeline is currently limited to two-drug combinations
- When the query requires pre-clinical/clinical validation (xenograft or mouse models) — the package only predicts and scores candidate combinations computationally
- When input drug-response data cannot be partitioned into resistant/sensitive groups, since AssignGroup(x, l) depends on that dichotomy

## Inputs

- Query directory with GDSC1000 cancer-type and primary/standard-of-care drug specification file (for CombinationDrugs(dir, file, dest))
- Pharmacogenomics drug-response data (e.g., the >70 human myeloma cell lines panel) for AssignGroup(x, l) resistant/sensitive partitioning
- Proportion parameters p1, p2 and sample size n for PropTest confidence scoring

## Outputs

- secdrug_combination_ranking.csv with populated rank and confidence columns
- Ranked list of top secondary drug combination candidates with confidence scores in [0,1]

## How to apply

Assemble a query directory and input file specifying a GDSC1000 cancer type and its standard-of-care primary drug matching the CombinationDrugs(dir, file, dest) call signature. Call AssignGroup(x, l) to partition pharmacogenomics drug-response data into resistant and sensitive groups, then execute the greedy set cover pipeline via CombinationDrugs(dir, file, dest) to generate ranked secondary drug combination candidates. Score the top combinations with PropTest(p1, p2, n), which yields proportion-based confidence values, and write the ranked candidates to secdrug_combination_ranking.csv. The pipeline is cancer- and drug-agnostic: the article states the package takes 'any cancer type and any test/primary/standard-of-care drug' as the query.

## Related tools

- **secDrug** (Core R package implementing the greedy set cover pipeline that predicts novel synergistic secondary drug combinations) — github.com/Ujjal-Mukherjee/secDrug
- **R** (Runtime environment hosting the universally applicable R-software package for predicting novel secondary therapies) — github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugs** (Exported function (dir, file, dest) that executes the greedy set cover pipeline to generate ranked secondary drug combination candidates) — github.com/Ujjal-Mukherjee/secDrug
- **AssignGroup** (Exported function (x, l) that partitions drug-response data into resistant and sensitive groups before combination search) — github.com/Ujjal-Mukherjee/secDrug
- **PropTest** (Exported function (p1, p2, n) yielding proportion-based confidence values in [0,1] for top combinations) — github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (Demonstration dataset/case for the myeloma-resistant-cell use case) — github.com/Ujjal-Mukherjee/secDrug

## Examples

```
CombinationDrugs(dir = "<query_dir>", file = "<cancer_primary_drug_input>", dest = "<output_dir>"); ranking <- read.csv("secdrug_combination_ranking.csv"); ci <- PropTest(p1, p2, n)
```

## Evaluation signals

- secdrug_combination_ranking.csv exists and its rank column is populated
- Confidence column in secdrug_combination_ranking.csv is populated and all values lie within [0,1]
- Output is a ranked list of top drug combination candidates as produced by the greedy set cover pipeline
- AssignGroup correctly split the pharmacogenomics data into exactly two groups (resistant and sensitive) before running CombinationDrugs
- The query cancer type and primary drug were accepted without hard-coding, confirming the cancer-/drug-agnostic behavior

## Limitations

- Pipeline currently limited to two-drug combinations; model systems for testing more than two secDrugs are not yet developed
- Predicted secDrugs (e.g., KIN001-102 and SB505124) have not yet undergone pre-clinical and clinical validation in xenograft or mouse models
- Novel secDrugs have not yet been tested in a wider panel of HMCLs representing refractory and clonally derived acquired resistant cell lines

## Evidence

- [abstract] cancer/drug-agnostic query design: "we developed a universally applicable R-software package for predicting novel secondary therapies in chemotherapy-resistant cancers"
- [abstract] outputs with rank and confidence: "outputs a list of the top drug combination candidates with rank and confidence scores"
- [methods] exported function signatures: "- `CombinationDrugs(dir, file, dest)`"
- [abstract] greedy set cover algorithm: "a computational pipeline that uses pharmacogenomics data-driven optimization-regularization/greedy algorithm to predict novel drugs ("secDrugs") against drug-resistant myeloma"
- [discussion] two-drug limitation: "A logical extension of this pipeline would be the development of model systems where a combination of more than two secDrugs can be effectively tested."
