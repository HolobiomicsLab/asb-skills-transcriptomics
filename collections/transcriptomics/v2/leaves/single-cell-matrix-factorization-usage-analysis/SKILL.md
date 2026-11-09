---
name: single-cell-matrix-factorization-usage-analysis
description: Use when benchmarking matrix-factorization-based GEP discovery (e.g.,
  cNMF) against network-based GEP identification (e.g., SingleCellGGM) on a labeled
  single-cell dataset, to quantify how many programs are cell-type-specific versus
  shared activity programs.
when_to_use_negative:
- The goal is cell-type label transfer or annotation of a novel dataset — use CellGEP/cellgep
  with pre-identified marker GEPs instead of de novo factorization.
- The dataset has fewer than ~5000 cells; SingleCellGGM explicitly recommends large
  cell numbers, and the GGM-based comparison baseline assumes such scale.
- You only need marker-gene-per-cell-type identification; the paper notes previous
  studies mainly focused on cell-type-specific markers, whereas this skill targets
  shared activity programs.
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: cNMF
  role: Consensus NMF factorization of the MCA count matrix at k=137 to produce usage
    and loading matrices for GEP identification
- name: SingleCellGGM
  role: GGM-based co-expression network baseline whose 18 MCA activity GEPs serve
    as the comparison reference for cNMF program classification
  repo: MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:542e1e0279f5061e47b7597163cc47794b55d9888df19b4c8b55095ec1159fcc
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
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-matrix-factorization-usage-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies consensus non-negative matrix factorization (cNMF, Kotliar et al.) to a single-cell count matrix — e.g., the mouse MCA dataset with k=137 factors — to derive program usage and loading matrices, then classifies each program as a cell-type-specific identity program or a shared-usage activity GEP for comparison against SingleCellGGM-identified GEPs.

## When to use

Use when benchmarking matrix-factorization-based GEP discovery (e.g., cNMF) against network-based GEP identification (e.g., SingleCellGGM) on a labeled single-cell dataset, to quantify how many programs are cell-type-specific versus shared activity programs.

## When NOT to use

- The goal is cell-type label transfer or annotation of a novel dataset — use CellGEP/cellgep with pre-identified marker GEPs instead of de novo factorization.
- The dataset has fewer than ~5000 cells; SingleCellGGM explicitly recommends large cell numbers, and the GGM-based comparison baseline assumes such scale.
- You only need marker-gene-per-cell-type identification; the paper notes previous studies mainly focused on cell-type-specific markers, whereas this skill targets shared activity programs.

## Inputs

- Single-cell gene count matrix (e.g., mouse MCA Microwell-Seq counts, cells x genes)
- Known cell-type labels for each cell (MCA annotations)
- Fixed number of factors (k=137 for the MCA benchmark)

## Outputs

- cNMF program usage matrix (cells x 137 programs)
- cNMF program loading matrix (programs x genes)
- Per-program classification table written to cnmf_program_specificity.csv (137 rows: identity vs activity assignment)
- Counts of identity vs activity programs for comparison with SingleCellGGM (18 activity GEPs)

## How to apply

Load the single-cell count matrix (e.g., mouse MCA from Microwell-Seq) and run cNMF with the number of factors fixed at k=137 to obtain a program usage matrix and a program loading matrix. For each of the 137 programs, compute cell-type specificity by associating program usage values with known cell-type labels, then classify each program as an identity program (usage restricted to a single cell type) or a shared-usage activity GEP (used across multiple cell types). Tabulate identity-versus-activity program counts and compare against the SingleCellGGM result on the same dataset (18 activity GEPs). Write the full per-program classification table to cnmf_program_specificity.csv. The comparison rationale is that SingleCellGGM's graphical Gaussian model co-expression network analysis recovers activity GEPs that cNMF largely misses.

## Related tools

- **cNMF** (Consensus NMF factorization of the MCA count matrix at k=137 to produce usage and loading matrices for GEP identification)
- **SingleCellGGM** (GGM-based co-expression network baseline whose 18 MCA activity GEPs serve as the comparison reference for cNMF program classification) — MaShisongLab/SingleCellGGM

## Examples

```
ggm = SingleCellGGM(expression_matrix, 20000, gene_names, 'MCA')
```

## Evaluation signals

- cnmf_program_specificity.csv contains exactly one row per cNMF program (137 rows for the MCA run)
- Reproduced counts match the reported finding: almost all programs are identity (cell-type-specific) programs and exactly 1 is a shared-usage activity GEP
- Comparison baseline holds: SingleCellGGM identifies 18 activity GEPs on the same MCA dataset, so cNMF activity-GEP count (1) << 18
- Each program's classification is supported by an association test of its usage values against known MCA cell-type labels
- Program loading and usage matrices have consistent dimensions with the input count matrix and k=137 factors

## Limitations

- cNMF underperforms at identifying activity GEPs shared across cell types: only 1 shared activity GEP was recovered versus 18 from SingleCellGGM on MCA
- Activity and morphogenesis GEPs have been largely overlooked by previous single-cell studies, so factorization-based benchmarks may inherit a marker-gene bias
- Number of factors is fixed (k=137 for MCA), so results depend on that choice rather than being tuned per dataset

## Evidence

- [discussion] Notably, these GEPs have been largely overlooked by previous single-cell studies, potentially due to previous studies mainly focusing on identifying marker genes specific for individual cell types: [evidence span withheld — anchor status: not_found]
- [readme] SingleCellGGM takes a single-cell gene expression matrix as input and uses a process consisting of ~20,000 iterations to calculate partial correlation coefficients (pcors) between gene pairs.: [evidence span withheld — anchor status: external_doc]
