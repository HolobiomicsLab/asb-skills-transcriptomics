---
name: neighbor-cell-type-score-computation
description: Use when building cell-cell communication networks from single-cell-resolution
  spatial transcriptomics data (e.g., seqFISH+ mouse cortex, Seq-Scope mouse colon)
  where a coordinate matrix and cell type label vector are available and neighbor
  exposure scores must precede regression modeling.
when_to_use_negative:
- Data lack single-cell spatial coordinates (e.g., spot-level or bulk data) — the
  log10(distance/dist0) decay requires per-cell positions
- The goal is ligand-receptor-based communication inference (e.g., CellPhoneDB-style)
  rather than spatial neighbor-effect regression on HVG expression
- Cell type labels have not yet been assigned to individual cells
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CPPLS-MLP
  role: Provides the R scripts (2023_01_seqFISH+.RMD, 2023_01_Seq-Scope.RMD) implementing
    the adjacent cell type score computation and subsequent CPPLS communication network
    construction
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: R (version <= 4.2.3)
  role: Runtime environment for the score computation and normalization scripts
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: pls
  role: R package used in the downstream CPPLS regression that consumes the normalized
    neighbor scores
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:209762e2e3b9b987c09a27fd7762c7d278b81daef48eac6a3ced295bcb39899b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - R
  - R (version <= 4.2.3)
  - R (version <=)
  - Seurat
  - pls
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# neighbor-cell-type-score-computation

## Summary

Computes raw adjacent cell type scores that quantify how strongly each cell is exposed to neighboring cell types, using spatial coordinates and cell type labels, then z-score normalizes them per cell type. These scores are the independent variables (B matrix) for downstream CPPLS regression modeling of cell-cell communication effects on HVG expression.

## When to use

Use when building cell-cell communication networks from single-cell-resolution spatial transcriptomics data (e.g., seqFISH+ mouse cortex, Seq-Scope mouse colon) where a coordinate matrix and cell type label vector are available and neighbor exposure scores must precede regression modeling.

## When NOT to use

- Data lack single-cell spatial coordinates (e.g., spot-level or bulk data) — the log10(distance/dist0) decay requires per-cell positions
- The goal is ligand-receptor-based communication inference (e.g., CellPhoneDB-style) rather than spatial neighbor-effect regression on HVG expression
- Cell type labels have not yet been assigned to individual cells

## Inputs

- Spatial coordinate matrix O (per-cell x/y positions) from single-cell-resolution spatial transcriptomics data
- Cell type label vector D (one label per cell)
- Optionally the gene expression matrix (seqFISH+ mouse cortex or Seq-Scope mouse colon) used downstream

## Outputs

- Raw adjacent cell type score matrix (per cell x neighboring cell type)
- Z-score normalized adjacent cell type scores per cell type, used as regression predictors in CPPLS modeling

## How to apply

Start from the input coordinate matrix O (per-cell x/y positions) and cell type label vector D. For each cell, compute raw adjacent cell type scores using the log10(distance/dist0) decay formula, which down-weights neighboring cells of each type as a function of their spatial distance from the focal cell. Then z-score transform the resulting scores separately for each cell type so that neighbor-effect predictors are comparable across cell types in the subsequent CPPLS regression (fitted with 10-fold cross-validation to estimate coefficients w_b,h for neighboring cell type effects on HVGs). The scoring and normalization are implemented in the 2023_01_seqFISH+.RMD and 2023_01_Seq-Scope.RMD scripts of the CPPLS-MLP repository.

## Related tools

- **CPPLS-MLP** (Provides the R scripts (2023_01_seqFISH+.RMD, 2023_01_Seq-Scope.RMD) implementing the adjacent cell type score computation and subsequent CPPLS communication network construction) — https://github.com/wuzhenao/CPPLS-MLP
- **R (version <= 4.2.3)** (Runtime environment for the score computation and normalization scripts) — https://github.com/wuzhenao/CPPLS-MLP
- **pls** (R package used in the downstream CPPLS regression that consumes the normalized neighbor scores)

## Examples

```
run 2023_01_seqFISH+.RMD
```

## Evaluation signals

- The score matrix has one row per cell and one column per cell type present in label vector D
- After normalization, each cell type's score column has mean ~0 and standard deviation ~1 across cells
- Distances between cells computed from O are consistent with the physical scale of the seqFISH+ or Seq-Scope field of view (dist0 threshold applied as in the log10(distance/dist0) formula)
- Downstream CPPLS regression for L5/L6 eNeuron runs to completion and produces a non-empty edge list CSV/RDS of cell type communication coefficients
- The script runs in R <= 4.2.3 with pls, Seurat, cluster, dplyr, purrr, stringr and ComplexHeatmap installed without errors

## Limitations

- The decay formula depends on a distance scale parameter (dist0) tuned for the seqFISH+/Seq-Scope datasets; other tissues or imaging resolutions may need recalibration
- Requires pre-computed cell type labels; the paper does not provide an automated clustering-to-label assignment step for this input
- Image-processing extraction of cell locations is only a proposal in the paper, not an implemented or validated component

## Evidence

- [methods] Computation of raw adjacent cell type scores from coordinate matrix and cell type labels: [evidence span withheld — anchor status: not_found]
- [other] Compute raw adjacent cell type scores using log10 decay then z-score per cell type: [evidence span withheld — anchor status: not_found]
- [readme] Run script to build cell-to-cell communication network with CPPLS: "If you want to use CPPLS to build a cell-to-cell communication network, you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages and run them in R"
