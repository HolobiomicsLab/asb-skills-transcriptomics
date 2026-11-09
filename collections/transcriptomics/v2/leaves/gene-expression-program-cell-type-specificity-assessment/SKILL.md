---
name: gene-expression-program-cell-type-specificity-assessment
description: Use when you have a program usage matrix (e.g., cNMF usage values for
  k factors) plus known cell-type labels and need to count identity vs. shared-usage
  activity GEPs or compare methods on activity-program recovery.
when_to_use_negative:
- Input is raw gene expression without a pre-computed program usage matrix — identify
  GEPs first (e.g., via SingleCellGGM or cNMF) before assessing specificity
- The dataset lacks reliable cell-type labels, since specificity classification depends
  on associating program usage with known labels
- The goal is functional (GO/MP) interpretation of GEPs rather than identity/activity
  classification — use ModuleEnrichmentAnalysis instead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: cNMF
  role: Consensus NMF method used to decompose the MCA count matrix into program usage
    and loading matrices (k=137) whose per-program specificity is then assessed
- name: SingleCellGGM
  role: Reference GEP-identification method whose 18 MCA activity GEPs serve as the
    comparison baseline for activity-program recovery
  repo: https://github.com/MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:582524f859b1cae59f87902d91244e0a2e7aa132ad15373b8ffd140819a39268
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/MaShisongLab/CellGEP
  repo_url: https://github.com/MaShisongLab/SingleCellGGM
  tools:
  - SingleCellGGM
  - cNMF
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-expression-program-cell-type-specificity-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify gene expression programs (GEPs) from single-cell data as cell-type-specific identity programs or shared-usage activity GEPs by associating program usage with known cell-type labels. This skill enables benchmarking of GEP-identification methods such as cNMF and SingleCellGGM on their ability to recover shared activity programs.

## When to use

Use when you have a program usage matrix (e.g., cNMF usage values for k factors) plus known cell-type labels and need to count identity vs. shared-usage activity GEPs or compare methods on activity-program recovery.

## When NOT to use

- Input is raw gene expression without a pre-computed program usage matrix — identify GEPs first (e.g., via SingleCellGGM or cNMF) before assessing specificity
- The dataset lacks reliable cell-type labels, since specificity classification depends on associating program usage with known labels
- The goal is functional (GO/MP) interpretation of GEPs rather than identity/activity classification — use ModuleEnrichmentAnalysis instead

## Inputs

- Program usage matrix (cells x programs) from cNMF or an equivalent NMF-style factorization
- Program loading matrix (programs x genes)
- Known cell-type label annotations for the dataset (e.g., MCA cell types)
- Number of factors k used in the factorization (e.g., k=137 for MCA)

## Outputs

- Per-program specificity classification table (one row per program), e.g., cnmf_program_specificity.csv
- Counts of identity (cell-type-specific) vs. shared-usage activity GEPs
- Method comparison summary against a reference method's activity-GEP count (e.g., SingleCellGGM's 18 MCA activity GEPs)

## How to apply

Run the GEP-identification method (e.g., cNMF with k fixed to the number of cell types, k=137 for MCA) to obtain a program usage matrix and program loading matrix. For each program, compute its usage values per cell and associate them with known cell-type labels (e.g., MCA cell-type annotations) to determine whether usage is restricted to a single cell type. Classify programs with cell-type-specific usage as identity programs and programs used across multiple cell types as shared-usage activity GEPs. Tabulate identity vs. activity counts and write a per-program classification table (e.g., cnmf_program_specificity.csv) with one row per program. For benchmarking, compare the activity-GEP count against a reference such as SingleCellGGM, which identified 18 activity GEPs on the same MCA dataset. Rationale: SingleCellGGM GEPs are derived from GGM co-expression networks and largely show cell-type-specific expression, so programs that break this pattern represent genuine shared activity programs.

## Related tools

- **cNMF** (Consensus NMF method used to decompose the MCA count matrix into program usage and loading matrices (k=137) whose per-program specificity is then assessed)
- **SingleCellGGM** (Reference GEP-identification method whose 18 MCA activity GEPs serve as the comparison baseline for activity-program recovery) — https://github.com/MaShisongLab/SingleCellGGM

## Examples

```
geps = read.table('data/mca.marker.geps.txt'); # classify each of 137 cNMF programs by associating usage with MCA cell-type labels, then write cnmf_program_specificity.csv
```

## Evaluation signals

- The output classification table contains exactly one row per program (137 rows for MCA with k=137)
- Reproduced identity/activity counts match the reported finding: almost all cNMF programs are identity (cell-type-specific) programs with only 1 shared-usage activity GEP
- Each program is assigned to exactly one class (identity or shared-usage activity GEP) with no unclassified programs
- The activity-GEP count is comparable against the SingleCellGGM reference of 18 activity GEPs identified from the same MCA dataset
- Specificity assignments are consistent with known MCA cell-type labels when spot-checked per program

## Limitations

- cNMF underperforms at identifying activity GEPs shared across cell types relative to SingleCellGGM (1 vs. 18 on MCA), so its programs should not be assumed to capture shared activity programs
- Activity and morphogenesis GEPs have been largely overlooked by previous single-cell studies that mainly focused on marker genes specific for individual cell types, so activity-program detection may be underrepresented in benchmarking references
- Specificity classification depends on the granularity and coverage of the provided cell-type labels

## Evidence

- [results] cNMF underperforms at identifying activity GEPs shared across cell types: [evidence span withheld — anchor status: not_found]
