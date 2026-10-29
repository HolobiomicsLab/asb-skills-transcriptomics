---
name: identity-versus-activity-gep-classification
description: Use when comparing GEP-identification methods on a single-cell dataset
  with known cell-type labels, specifically to quantify how many programs are cell-type-specific
  identity programs versus shared activity programs across cell types.
when_to_use_negative:
- When cell-type labels are unavailable, since the identity-versus-activity classification
  depends on associating program usage with known cell-type annotations.
- When the goal is to annotate cells in a novel dataset rather than benchmark GEP-identification
  methods — use CellGEP label transfer instead.
- When the dataset has fewer than ~5000 cells or high-expression noise, which SingleCellGGM's
  README warns against for its pcor-based network construction.
tools:
- name: cNMF
  role: Consensus NMF used to factorize the MCA count matrix at k=137 into program
    usage and loading matrices for classification as identity or activity programs
- name: SingleCellGGM
  role: GGM-based gene co-expression network analysis used as the benchmark method
    whose 18 MCA activity GEPs are compared against cNMF results
  repo: https://github.com/MaShisongLab/SingleCellGGM
- name: CellGEP
  role: Downstream tool that uses marker GEPs (from mca.marker.geps.txt) for cell-type
    annotation, providing the identity-program context
  repo: https://github.com/MaShisongLab/cellgep
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:e1460fc9933be129818688bb4d810cf041825062aee7a321c79b4f2ab4e24f0c
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
  - CellGEP
  - SingleCellGGM
  - cNMF
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# identity-versus-activity-gep-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies gene expression programs (GEPs) identified on a single-cell dataset as cell-type-specific identity programs versus shared-usage activity GEPs, and benchmarks alternative GEP-identification methods (e.g., cNMF with fixed k against SingleCellGGM) on the same dataset for their ability to recover shared activity programs.

## When to use

Use when comparing GEP-identification methods on a single-cell dataset with known cell-type labels, specifically to quantify how many programs are cell-type-specific identity programs versus shared activity programs across cell types.

## When NOT to use

- When cell-type labels are unavailable, since the identity-versus-activity classification depends on associating program usage with known cell-type annotations.
- When the goal is to annotate cells in a novel dataset rather than benchmark GEP-identification methods — use CellGEP label transfer instead.
- When the dataset has fewer than ~5000 cells or high-expression noise, which SingleCellGGM's README warns against for its pcor-based network construction.

## Inputs

- Single-cell gene count matrix (e.g., mouse MCA dataset, log-normalized for SingleCellGGM; cells in rows, genes in columns)
- Known cell-type labels for the dataset
- SingleCellGGM reference GEPs from the same dataset (e.g., 18 activity GEPs from MCA)

## Outputs

- cnmf_program_specificity.csv — one row per cNMF program (137 rows) with per-program identity/activity specificity assignments
- Tabulated counts of identity programs versus shared-usage activity GEPs per method (cNMF vs SingleCellGGM)

## How to apply

Load the single-cell count matrix (e.g., the mouse MCA dataset) and run cNMF (consensus NMF, Kotliar et al.) with the number of factors fixed at k=137 to obtain the program usage and program loading matrices. For each of the 137 programs, compute cell-type specificity by associating program usage values with known cell-type labels to determine whether usage is restricted to a single cell type. Classify each program as an identity program (cell-type-specific usage) or a shared-usage activity GEP (used across multiple cell types), then tabulate identity versus activity counts and compare against the SingleCellGGM result of 18 activity GEPs identified from the same MCA dataset. Write the per-program classification table to cnmf_program_specificity.csv and validate that it has 137 rows and that the reproduced counts match the reported finding (almost all programs are identity programs, only 1 shared-usage activity GEP). The rationale is that cNMF recovers mostly identity (marker-gene) programs, while SingleCellGGM's GGM co-expression networks recover activity GEPs that are largely overlooked by marker-gene-focused approaches.

## Related tools

- **cNMF** (Consensus NMF used to factorize the MCA count matrix at k=137 into program usage and loading matrices for classification as identity or activity programs)
- **SingleCellGGM** (GGM-based gene co-expression network analysis used as the benchmark method whose 18 MCA activity GEPs are compared against cNMF results) — https://github.com/MaShisongLab/SingleCellGGM
- **CellGEP** (Downstream tool that uses marker GEPs (from mca.marker.geps.txt) for cell-type annotation, providing the identity-program context) — https://github.com/MaShisongLab/cellgep

## Examples

```
ggm = SingleCellGGM(expression_matrix, 20000, gene_names, 'MCA')
```

## Evaluation signals

- cnmf_program_specificity.csv contains exactly one row per cNMF program (137 rows).
- Reproduced counts show almost all cNMF programs classified as identity (cell-type-specific) programs, with only 1 shared-usage activity GEP.
- The comparison against SingleCellGGM reproduces the reported gap: SingleCellGGM identifies 18 activity GEPs on the same MCA dataset vs 1 for cNMF.
- Each program has a definitive identity/activity classification supported by its cell-type association statistic.

## Limitations

- cNMF underperforms at identifying activity GEPs shared across cell types compared with SingleCellGGM, recovering only 1 shared activity GEP on MCA.
- Activity and morphogenesis GEPs have been largely overlooked by previous single-cell studies because prior work mainly focused on marker genes specific for individual cell types.
- SingleCellGGM relies exclusively on transcriptome data and does not integrate other single-cell modalities (e.g., ATAC-seq, single-cell proteomics).

## Evidence

- [readme] SingleCellGGM usage call: [evidence span withheld — anchor status: external_doc]
