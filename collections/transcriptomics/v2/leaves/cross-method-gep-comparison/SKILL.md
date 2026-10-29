---
name: cross-method-gep-comparison
description: Use when you need to compare activity GEPs from a baseline method (e.g.,
  cNMF at k=137 factors on MCA) with SingleCellGGM GEPs to assess cell-type specificity
  and shared-program recovery.
when_to_use_negative:
- The goal is cell-type label transfer or annotation of a novel dataset — use CellGEP's
  cellgep/average_by_gep workflow instead of program benchmarking.
- The method under comparison operates on metacell-aggregated data (e.g., MetaCell,
  scCorr) rather than producing per-cell program usage matrices comparable to cNMF's
  usage matrix.
- The dataset is small-scale (<5000 cells); the SingleCellGGM comparison baseline
  assumes large cell numbers.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: cNMF
  role: Consensus NMF baseline run at k=137 factors on MCA to produce program usage/loading
    matrices for the comparison
- name: SingleCellGGM
  role: Reference GEP-identification method whose 18 MCA activity GEPs serve as the
    comparison benchmark
  repo: MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:a89706e5027deca7aa336e33270041e46a1c42f2ccc5e946a64fa8ea40529241
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-method-gep-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark gene expression programs (GEPs) from an alternative method (cNMF, consensus NMF by Kotliar et al.) against SingleCellGGM-derived GEPs on the same single-cell dataset (mouse MCA), classifying each program as a cell-type-specific identity program versus a shared-usage activity GEP.

## When to use

Use when you need to compare activity GEPs from a baseline method (e.g., cNMF at k=137 factors on MCA) with SingleCellGGM GEPs to assess cell-type specificity and shared-program recovery.

## When NOT to use

- The goal is cell-type label transfer or annotation of a novel dataset — use CellGEP's cellgep/average_by_gep workflow instead of program benchmarking.
- The method under comparison operates on metacell-aggregated data (e.g., MetaCell, scCorr) rather than producing per-cell program usage matrices comparable to cNMF's usage matrix.
- The dataset is small-scale (<5000 cells); the SingleCellGGM comparison baseline assumes large cell numbers.

## Inputs

- Mouse MCA (Mouse Cell Atlas) raw count matrix (cells x genes)
- Known MCA cell-type labels for each cell
- SingleCellGGM reference result: 18 activity GEPs identified from the same MCA dataset
- cNMF with number of factors fixed at k=137

## Outputs

- cnmf_program_specificity.csv — one row per cNMF program (137 rows) with per-program cell-type-specificity and identity/activity classification
- Tabulated counts of identity vs. activity programs for cNMF vs. SingleCellGGM

## How to apply

Run cNMF on the mouse MCA count matrix with the number of factors fixed at k=137 to obtain the program usage and loading matrices, mirroring the dataset on which SingleCellGGM identified 18 activity GEPs. Compute cell-type specificity for each of the 137 programs by associating per-cell program usage values with known MCA cell-type labels, and classify each program as an identity program (cell-type-specific usage) or a shared-usage activity GEP (used across multiple cell types). Tabulate identity versus activity program counts and compare against the SingleCellGGM result of 18 activity GEPs from the same dataset. The expected outcome is that almost all cNMF programs are cell-type-specific identity programs with only 1 shared-usage activity GEP, so cNMF underperforms at identifying activity GEPs shared across cell types. Write the full per-program classification table to cnmf_program_specificity.csv.

## Related tools

- **cNMF** (Consensus NMF baseline run at k=137 factors on MCA to produce program usage/loading matrices for the comparison)
- **SingleCellGGM** (Reference GEP-identification method whose 18 MCA activity GEPs serve as the comparison benchmark) — MaShisongLab/SingleCellGGM

## Examples

```
ggm = SingleCellGGM(`expression_matrix`,`number_of_iterations`,`gene_names`,`dataset_name`)
```

## Evaluation signals

- cnmf_program_specificity.csv contains exactly one row per cNMF program (137 programs total).
- Reproduced counts show almost all 137 programs classified as identity (cell-type-specific) programs.
- Exactly 1 program is classified as a shared-usage activity GEP, matching the reported finding.
- Activity-program count (1 for cNMF) is far below the SingleCellGGM benchmark of 18 activity GEPs on the same MCA dataset.
- Each program's specificity assignment is supported by an association of its usage values with known MCA cell-type labels.

## Limitations

- cNMF comparison reflects the specific factorization choice k=137 on MCA; a different k could change identity/activity program counts.
- Activity and morphogenesis GEPs have been largely overlooked by previous single-cell studies, potentially because prior work focused on identifying marker genes specific for individual cell types, so few methods are optimized for shared-usage programs.

## Evidence

- [readme] SingleCellGGM input and pcor computation: [evidence span withheld — anchor status: external_doc]
