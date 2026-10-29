---
name: cancer-type-agnostic-pipeline-execution
description: Use when a query cancer type and drug pair must be pushed through secDrug's
  greedy set cover pipeline to rank secondary combination candidates with confidence
  scores in [0,1].
when_to_use_negative:
- The goal is validating candidate combinations experimentally (functional assays,
  CRISPR, CyTOF) rather than in silico ranking
- A query for combinations of more than two secondary drugs — the pipeline is currently
  limited to two-drug combinations
- No pharmacogenomics drug-response data exist for the queried cancer type, so resistant/sensitive
  grouping via AssignGroup cannot be formed
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0659
tools:
- name: secDrug
  role: R software package executing the greedy set cover pipeline that ranks secondary
    drug combination candidates for any queried cancer type and drug
  repo: https://github.com/Ujjal-Mukherjee/secDrug
- name: R
  role: Runtime environment for the universally applicable secDrug package
  repo: https://github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugs
  role: Core exported function CombinationDrugs(dir, file, dest) that runs the greedy
    set cover pipeline and writes secdrug_combination_ranking.csv
  repo: https://github.com/Ujjal-Mukherjee/secDrug
- name: AssignGroup
  role: Exported function AssignGroup(x, l) partitioning drug-response data into resistant/sensitive
    groups
  repo: https://github.com/Ujjal-Mukherjee/secDrug
- name: PropTest
  role: Exported function PropTest(p1, p2, n) producing proportion-based confidence
    values in [0,1] for top combinations
  repo: https://github.com/Ujjal-Mukherjee/secDrug
- name: CombinationDrugMyeloma
  role: Exported dataset used as the demonstration cancer-type/drug configuration
  repo: https://github.com/Ujjal-Mukherjee/secDrug
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41408-022-00636-2
    title: 'secDrug: a pipeline to discover novel drug combinations to kill drug-resistant
      multiple myeloma cells using a greedy set cover algorithm and single-cell multi-omics'
  source_content_hash: sha256:7fe5f49268787a8f7d22fd23e679d00913b8cfa824f4eab9d649accdd916417d
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cancer-type-agnostic-pipeline-execution

## Summary

Run the secDrug R package as a cancer- and drug-agnostic query engine: any GDSC1000 cancer type plus a test/primary/standard-of-care drug yields a ranked list of secondary drug combinations with proportion-based confidence scores via a greedy set cover algorithm.

## When to use

Use when a query cancer type and drug pair must be pushed through secDrug's greedy set cover pipeline to rank secondary combination candidates with confidence scores in [0,1].

## When NOT to use

- The goal is validating candidate combinations experimentally (functional assays, CRISPR, CyTOF) rather than in silico ranking
- A query for combinations of more than two secondary drugs — the pipeline is currently limited to two-drug combinations
- No pharmacogenomics drug-response data exist for the queried cancer type, so resistant/sensitive grouping via AssignGroup cannot be formed

## Inputs

- Query directory + input file specifying a GDSC1000 cancer type and its standard-of-care primary drug
- Pharmacogenomics drug-response data (e.g., GDSC1000/HMCL panel drug-response matrix)
- CombinationDrugMyeloma dataset object from the secDrug R package

## Outputs

- secdrug_combination_ranking.csv with populated rank and confidence columns
- Ranked list of top secondary drug combination candidates with confidence scores in [0,1] from PropTest(p1, p2, n)

## How to apply

Assemble the query directory and input file specifying the chosen GDSC1000 cancer type and its standard-of-care primary drug, matching the CombinationDrugs(dir, file, dest) call signature. Partition the pharmacogenomics drug-response data into resistant and sensitive groups with AssignGroup(x, l), which the greedy set cover algorithm uses to identify combinations that shift resistant cells toward sensitivity. Execute CombinationDrugs(dir, file, dest) to generate ranked secondary drug combination candidates, then score each top combination with PropTest(p1, p2, n), which yields proportion-based confidence values constrained to [0,1]. Rationale: the package is explicitly designed as 'a universally applicable R-software package for predicting novel secondary therapies in chemotherapy-resistant cancers,' demonstrated for PI/IMiD-resistant myeloma but not limited to it.

## Related tools

- **secDrug** (R software package executing the greedy set cover pipeline that ranks secondary drug combination candidates for any queried cancer type and drug) — https://github.com/Ujjal-Mukherjee/secDrug
- **R** (Runtime environment for the universally applicable secDrug package) — https://github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugs** (Core exported function CombinationDrugs(dir, file, dest) that runs the greedy set cover pipeline and writes secdrug_combination_ranking.csv) — https://github.com/Ujjal-Mukherjee/secDrug
- **AssignGroup** (Exported function AssignGroup(x, l) partitioning drug-response data into resistant/sensitive groups) — https://github.com/Ujjal-Mukherjee/secDrug
- **PropTest** (Exported function PropTest(p1, p2, n) producing proportion-based confidence values in [0,1] for top combinations) — https://github.com/Ujjal-Mukherjee/secDrug
- **CombinationDrugMyeloma** (Exported dataset used as the demonstration cancer-type/drug configuration) — https://github.com/Ujjal-Mukherjee/secDrug

## Examples

```
CombinationDrugs(dir, file, dest)
```

## Evaluation signals

- secdrug_combination_ranking.csv exists and its rank column is populated
- The confidence column in secdrug_combination_ranking.csv is populated
- All confidence values lie within [0,1], consistent with PropTest proportion-based scoring
- Output is a ranked list of top drug combination candidates, matching the documented package behavior
- The query succeeded with a cancer type/drug pair different from the PI-resistant B-cell myeloma demonstration, confirming cancer-agnostic generalization

## Limitations

- Pipeline is currently limited to two-drug combinations; model systems for testing more than two secDrugs are not yet developed
- Pre-clinical and clinical validation in xenograft or mouse models not yet performed for novel secDrugs (e.g., KIN001-102 and SB505124)
- Testing of predicted secDrugs in a wider panel of HMCLs representing refractory and clonally derived acquired resistant cell lines is not yet done
- Supplementary Methods, Tables, and Figures are referenced but not included/verifiable in the provided text

## Evidence

- [abstract] R package outputs ranked, confidence-scored combinations for any resistant cancer: "developed a universally applicable R-software package for predicting novel secondary therapies in chemotherapy-resistant cancers"
- [abstract] Package outputs top combination candidates with rank and confidence scores: "outputs a list of the top drug combination candidates with rank and confidence scores"
- [abstract] Greedy set cover algorithm predicts secondary drugs: "a computational pipeline that uses pharmacogenomics data-driven optimization-regularization/greedy algorithm to predict novel drugs"
- [methods] Exported function signatures for the pipeline steps: "- `CombinationDrugs(dir, file, dest)`"
- [methods] PropTest confidence scoring signature: "- `PropTest(p1, p2, n)`"
- [discussion] Pipeline limited to two-drug combinations: "A logical extension of this pipeline would be the development of model systems where a combination of more than two secDrugs can be"
