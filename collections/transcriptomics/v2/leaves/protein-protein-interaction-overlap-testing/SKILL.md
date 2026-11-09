---
name: protein-protein-interaction-overlap-testing
description: Use when you have a list of predicted candidate regulators (e.g. 73 CRIT
  NMF-cluster candidates) and a gold-standard regulator list (gold_regulators.xlsx),
  and need to quantify physical-interaction enrichment between the two sets.
when_to_use_negative:
- The goal is co-expression or shared-transcription-factor association testing — these
  use the MSigDB Cancer Gene Co-expression Modules and TRANSFAC Curated Transcription
  Factor Targets datasets instead of the Pathway Commons PPI dataset
- The input is raw omics data (TCGA/MiOncoCirc/RNAct files) rather than an already-computed
  candidate regulator list from NMF clustering
- No gold-standard regulator list is available to define the reference set against
  which overlap is tested
edam_operation: http://edamontology.org/operation_2949
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: CRIT
  role: Pipeline whose NMF-clustered candidate regulators are the candidates tested
    for PPI overlap with gold regulators
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Implementation language for intersecting gene sets, running the 10,000 random-sampling
    permutations, and writing result tables (readr/data.table)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:4a6bba9a00f2114020975dd06eb6760e9c74875343083f8fd575b80010a56254
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  repo_url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - R
  edam_operation: http://edamontology.org/operation_2949
  edam_topics:
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# protein-protein-interaction-overlap-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Tests whether predicted circRNA regulators are functionally connected to gold-standard regulators through protein-protein interactions by intersecting candidate and gold gene sets in the Harmonizome Pathway Commons Protein-Protein Interactions dataset and assessing overlap significance with 10,000 random-sampling permutations at FDR < 0.05.

## When to use

Use when you have a list of predicted candidate regulators (e.g. 73 CRIT NMF-cluster candidates) and a gold-standard regulator list (gold_regulators.xlsx), and need to quantify physical-interaction enrichment between the two sets.

## When NOT to use

- The goal is co-expression or shared-transcription-factor association testing — these use the MSigDB Cancer Gene Co-expression Modules and TRANSFAC Curated Transcription Factor Targets datasets instead of the Pathway Commons PPI dataset
- The input is raw omics data (TCGA/MiOncoCirc/RNAct files) rather than an already-computed candidate regulator list from NMF clustering
- No gold-standard regulator list is available to define the reference set against which overlap is tested

## Inputs

- Candidate regulator list from CRIT NMF clustering (chained output, e.g. NMFgroup_XXclusterYYrun.txt)
- gold_regulators.xlsx (gold-standard circRNA regulator list, ./DATA/input/)
- Harmonizome Pathway Commons Protein-Protein Interactions dataset (./DATA/input/Harmonizome/)

## Outputs

- Physical-interaction association table reporting candidate counts, observed overlaps, and permutation-based p-values/FDR (significant candidates at FDR < 0.05)

## How to apply

Load the candidate regulator list produced by the CRIT NMF clustering (chained input) and the gold_regulators.xlsx file into R using readr/data.table. Intersect candidates and gold regulators in the Harmonizome 'Pathway Commons Protein-Protein Interactions' dataset, then compare the observed overlap against 10,000 random-sampling permutations to obtain per-candidate empirical p-values, adjusting for multiple testing to FDR < 0.05. In the CRIT application this recovered 57 of 73 newly predicted candidates with significantly more physical interactions with gold circRNA regulators, supporting their roles in circRNA regulation. Write the result table reporting candidate counts, observed overlaps, and permutation-based p-values/FDR to the output directory.

## Related tools

- **CRIT** (Pipeline whose NMF-clustered candidate regulators are the candidates tested for PPI overlap with gold regulators) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Implementation language for intersecting gene sets, running the 10,000 random-sampling permutations, and writing result tables (readr/data.table))

## Examples

```
Rscript RBP_analysis.R
```

## Evaluation signals

- Reproduction of the published result: 57 of the 73 newly predicted candidates show significantly more physical interactions with gold circRNA regulators
- Per-candidate significance is based on 10,000 random-sampling permutations, with multiple-testing correction applied so selected candidates pass FDR < 0.05
- Output table contains the required fields: candidate counts, observed overlap sizes, and permutation-based p-values/FDR
- Overlap is computed against the Harmonizome 'Pathway Commons Protein-Protein Interactions' dataset specifically, not other Harmonizome datasets

## Limitations

- Significance depends on the coverage and curation of the Pathway Commons Protein-Protein Interactions dataset; interactions absent from Harmonizome cannot be detected
- Permutation p-values are empirical (10,000 samples), so the minimum attainable p-value is bounded by the number of permutations
- Candidate results are chained from the CRIT NMF clustering step; upstream clustering errors propagate into the overlap test

## Evidence

- [methods] 57 of the 73 newly predicted candidates have significantly more physical interactions with gold circRNA regulators (FDR < 0.05): [evidence span withheld — anchor status: not_found]
