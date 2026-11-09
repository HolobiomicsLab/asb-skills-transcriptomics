---
name: consensus-nmf-factorization-of-single-cell-counts
description: 'Use when benchmarking GEP-identification methods on a single-cell transcriptome
  dataset: you need to decompose counts into transcriptional programs and classify
  each program as cell-type-specific (identity) versus shared-usage (activity) to
  compare with SingleCellGGM output.'
when_to_use_negative:
- Your goal is to identify shared activity GEPs across cell types — cNMF recovered
  almost no activity programs on MCA (1 of 137) and will underperform versus SingleCellGGM
  (18 activity GEPs)
- The dataset has fewer than ~5,000 cells or unfiltered low-expression genes, which
  prevents reliable matrix factorization and co-expression analysis
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: cNMF
  role: Consensus NMF factorization of the MCA count matrix at k=137 to obtain program
    usage and loading matrices for benchmarking against SingleCellGGM GEPs
- name: SingleCellGGM
  role: Reference method whose 18 MCA activity GEPs provide the comparison target
    for cNMF's program classifications
  repo: https://github.com/MaShisongLab/SingleCellGGM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100813
    title: SingleCellGGM enables gene expression program identification from single-cell
      transcriptomes and facilitates universal cell label transfer
  source_content_hash: sha256:8c9763ddc5f7481f792a3163024ae9ee0b0111c763d36943e0d1c9c7ff2bda03
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Consensus NMF factorization of single-cell count matrices (cNMF)

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs cNMF (Kotliar et al.) consensus non-negative matrix factorization on a single-cell count matrix (here the mouse MCA dataset at fixed k=137) to decompose cells x genes expression into program usage and program loading matrices, as a baseline for comparison against SingleCellGGM-identified gene expression programs (GEPs).

## When to use

Use when benchmarking GEP-identification methods on a single-cell transcriptome dataset: you need to decompose counts into transcriptional programs and classify each program as cell-type-specific (identity) versus shared-usage (activity) to compare with SingleCellGGM output.

## When NOT to use

- Your goal is to identify shared activity GEPs across cell types — cNMF recovered almost no activity programs on MCA (1 of 137) and will underperform versus SingleCellGGM (18 activity GEPs)
- The dataset has fewer than ~5,000 cells or unfiltered low-expression genes, which prevents reliable matrix factorization and co-expression analysis

## Inputs

- Mouse MCA (Mouse Cell Atlas) count matrix (cells in rows, genes in columns, e.g. Han et al. 2018 microwell-seq data)
- Known MCA cell-type labels for each cell
- Fixed number of factors (k=137)

## Outputs

- cNMF program usage matrix (cells x 137 programs)
- cNMF program loading matrix (137 programs x genes)
- cnmf_program_specificity.csv: one row per cNMF program with per-program identity/activity (cell-type-specificity) assignments

## How to apply

Load the mouse MCA count matrix (cells x genes) and run cNMF (consensus NMF, Kotliar et al.) with the number of factors fixed at k=137 to obtain the program usage matrix and the program loading matrix. Compute cell-type specificity for each of the 137 programs by associating program usage values with known MCA cell-type labels to determine which programs are restricted to a single cell type. Classify each program as an identity program (cell-type-specific usage) or a shared-usage activity GEP (used across multiple cell types), then tabulate identity versus activity counts and compare against the SingleCellGGM result of 18 activity GEPs identified from the same MCA dataset. Rationale: in the MCA dataset almost all cNMF GEPs were cell-type-specific and only 1 was a shared activity GEP, so cNMF underperforms at identifying activity GEPs shared across cell types relative to SingleCellGGM's 18.

## Related tools

- **cNMF** (Consensus NMF factorization of the MCA count matrix at k=137 to obtain program usage and loading matrices for benchmarking against SingleCellGGM GEPs)
- **SingleCellGGM** (Reference method whose 18 MCA activity GEPs provide the comparison target for cNMF's program classifications) — https://github.com/MaShisongLab/SingleCellGGM

## Evaluation signals

- The output table cnmf_program_specificity.csv contains exactly one row per cNMF program (137 rows)
- Reproduced identity/activity counts match the reported finding: almost all programs are identity programs and exactly 1 shared-usage activity GEP
- Each program is assigned exactly one class: identity (cell-type-specific usage) or shared-usage activity GEP
- The activity-GEP count for cNMF (1) is far below the SingleCellGGM count (18) on the same MCA dataset, confirming the reported underperformance
- Program usage values associate cleanly with known MCA cell-type labels for identity programs

## Limitations

- cNMF underperforms at identifying activity GEPs shared across cell types: on MCA only 1 of 137 programs was a shared activity GEP versus 18 from SingleCellGGM
- Activity and morphogenesis GEPs like those SingleCellGGM identifies have been largely overlooked by previous single-cell studies, so factorization methods tuned to marker (identity) programs may systematically miss them

## Evidence

- [intro] cNMF is a consensus NMF method for single-cell data: "Kotliar et al. developed a consensus NMF (cNMF) method for analyzing two single-cell datasets from brain organoid and visual cortex samples"
- [discussion] Activity/morphogenesis GEPs overlooked by prior studies: "Notably, these GEPs have been largely overlooked by previous single-cell studies, potentially due to previous studies mainly focusing on identifying marker genes specific for individual cell types"
- [discussion] SingleCellGGM handles large-scale transcriptomes (MCA benchmark context): "When applied to the MCA and Mouse Brain datasets, SingleCellGGM generated two GGM GCNs from which GEPs of biological significance were identified"
- [abstract] GEPs defined as functionally enriched co-expression modules: "SingleCellGGM constructed networks from which gene co-expression modules with highly significant functional enrichment were identified. We considered the modules as gene expression programs (GEPs)."
