---
name: batch-effect-simulation-parameter-design
description: Use when a benchmark of batch-correction or multi-source integration
  methods needs simulated scRNA-seq count matrices with known cell-type and batch
  effect magnitudes instead of real data lacking ground truth.
when_to_use_negative:
- The analysis requires real biological datasets (e.g. TCGA/BRCA multi-omics) rather
  than synthetic data with known ground truth
- The benchmark needs more than two batches or a different number of cell types than
  three, since the grid here fixes 2 batches and 3 cell types
- Only a single simulation condition is needed — the value of this skill is the crossed
  100-condition grid, not one-off simulation
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Splatter
  role: R package used to generate the synthetic Splat-model count matrices for each
    of the 100 simulation conditions
- name: R
  role: Environment in which Splatter simulation parameters are configured, splatSimulate
    is called, and outputs are serialized
- name: scJIVE
  role: Downstream batch-effect-correction method evaluated on the simulated conditions;
    JIVE performed best preserving cell-type effects and in balanced batch-size scenarios
  repo: https://github.com/oconnell-statistics-lab/scJIVE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:b626fc53a75369c808f91a93f19c600164c801d84cc00da9bf12c995e57a45be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/oconnell-statistics-lab/scJIVE
  tools:
  - R
  - Splatter
  - scJIVE
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# batch-effect-simulation-parameter-design

## Summary

Designs a controlled grid of synthetic scRNA-seq benchmark datasets with the Splatter R package, crossing cell-type effect and batch effect location parameters with balanced/unbalanced cell-type proportions so batch-correction methods can be evaluated under known ground truth.

## When to use

Use when a benchmark of batch-correction or multi-source integration methods needs simulated scRNA-seq count matrices with known cell-type and batch effect magnitudes instead of real data lacking ground truth.

## When NOT to use

- The analysis requires real biological datasets (e.g. TCGA/BRCA multi-omics) rather than synthetic data with known ground truth
- The benchmark needs more than two batches or a different number of cell types than three, since the grid here fixes 2 batches and 3 cell types
- Only a single simulation condition is needed — the value of this skill is the crossed 100-condition grid, not one-off simulation

## Inputs

- Simulation parameter specification: gene count (5000), batch count (2), total cells (1000), cell types/groups (3)
- Grid values for cell-type effect location and batch effect location parameters {0, 0.25, 0.5, 0.75, 1}
- Balanced vs. unbalanced cell-type proportion settings
- Splatter R package (Splat group-based simulation)

## Outputs

- 100 simulated scRNA-seq count matrices (5000 gene rows x 1000 cell columns), one per condition
- Per-condition .rds (or .csv) serialized files on disk
- A parameter-grid map recording which location parameters and proportion setting define each condition

## How to apply

Load Splatter in R and fix global Splat-model parameters: 5000 genes, two batches totalling 1000 cells, and three cell types (groups) in similar proportions between batches. Build a parameter grid crossing the cell-type effect location parameter and the batch effect location parameter over {0, 0.25, 0.5, 0.75, 1}, combined with balanced and unbalanced cell-type proportion settings, yielding 100 simulation conditions. For each condition call Splatter's group-based splatSimulate so genes are rows and cells are columns, then serialize each count matrix to disk as one .rds (or .csv) file per condition. The varying batch-effect magnitudes plus the balanced/unbalanced split let downstream tools (e.g. JIVE, Seurat v5, Harmony, LIGER, ComBat-seq) be scored on preserving cell-type structure, and the balanced cases specifically probe performance when batch sizes are equal.

## Related tools

- **Splatter** (R package used to generate the synthetic Splat-model count matrices for each of the 100 simulation conditions)
- **R** (Environment in which Splatter simulation parameters are configured, splatSimulate is called, and outputs are serialized)
- **scJIVE** (Downstream batch-effect-correction method evaluated on the simulated conditions; JIVE performed best preserving cell-type effects and in balanced batch-size scenarios) — https://github.com/oconnell-statistics-lab/scJIVE

## Examples

```
library(Splatter); params <- newSplatParams(nGenes=5000, batchCells=c(500,500), nGroups=3, group.prob=rep(1/3,3)); sim <- splatSimulate(params, method='groups', group.loc=0.5, batch.loc=0.25); saveRDS(counts(sim), 'condition_001.rds')
```

## Evaluation signals

- All 100 output files exist on disk after the grid run (row_count_equals / file-existence check per condition)
- Each simulated count matrix has exactly 5000 gene rows and 1000 cell columns (dimension check per condition)
- Each condition maps one-to-one onto a grid cell: a (cell-type location, batch location) pair from {0, 0.25, 0.5, 0.75, 1} plus a balanced or unbalanced proportion flag
- Within each condition, the three cell types are present in the intended (balanced ~equal or unbalanced) proportions between the two batches
- Downstream sanity: batch-correction methods applied to the grid should show degraded cell-type preservation at high batch-effect locations (e.g. location = 1) relative to location = 0

## Limitations

- The grid fixes the Splat model at 5000 genes, 2 batches, 1000 cells and 3 cell types, so results do not speak to other dimensionalities or batch counts
- Only the cell-type effect and batch effect location parameters vary; other Splat parameters (dropout, library size, etc.) remain at defaults and are not probed
- Original JIVE was designed for bulk sequencing data and is computationally infeasible for large-scale single-cell datasets, so simulation scale was kept at 1000 cells per condition

## Evidence

- [methods] Splatter simulation with crossed location parameters: [evidence span withheld — anchor status: not_found]
- [methods] Parameter grid definition: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmark finding motivating balanced scenarios: "JIVE performed best in terms of preserving cell-type effects and in scenarios in which the batch sizes are balanced."
- [readme] Decomposition rationale for joint vs. individual structures: "scJIVE aims to decompose scRNA-seq datasets into a joint structure capturing the true biological variability and individual structures which capture technical variability within each batch."
