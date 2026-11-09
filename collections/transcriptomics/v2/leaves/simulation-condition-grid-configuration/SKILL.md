---
name: simulation-condition-grid-configuration
description: Use when you need controlled simulated scRNA-seq benchmarks to quantify
  how varying batch-effect and cell-type effect strengths (plus balanced vs unbalanced
  cell-type proportions) affect batch-correction tool performance.
when_to_use_negative:
- Input is real experimental scRNA-seq data rather than a simulation scenario — use
  batch-correction methods (e.g. scJIVE, Harmony) directly instead of generating synthetic
  conditions
- The analysis question does not involve systematically varying batch/cell-type effect
  strengths, so a single fixed Splatter condition would suffice rather than the full
  100-condition grid
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Splatter
  role: Simulates the scRNA-seq count matrices for each of the 100 grid conditions
    using its group-based Splat model (splatSimulate)
- name: R
  role: Environment in which Splatter parameters are configured, splatSimulate is
    run, and per-condition count matrices are serialized
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:b129881949adbde4783c07f41a1137fa033a409882725724d42b4e71172d7f2c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Splatter
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# simulation-condition-grid-configuration

## Summary

Configures a systematic grid of in silico scRNA-seq benchmark conditions using the Splatter R package (Splat model), varying cell-type effect and batch effect location parameters over {0, 0.25, 0.5, 0.75, 1} crossed with balanced/unbalanced cell-type proportions to yield 100 simulation conditions for evaluating batch-effect correction methods.

## When to use

Use when you need controlled simulated scRNA-seq benchmarks to quantify how varying batch-effect and cell-type effect strengths (plus balanced vs unbalanced cell-type proportions) affect batch-correction tool performance.

## When NOT to use

- Input is real experimental scRNA-seq data rather than a simulation scenario — use batch-correction methods (e.g. scJIVE, Harmony) directly instead of generating synthetic conditions
- The analysis question does not involve systematically varying batch/cell-type effect strengths, so a single fixed Splatter condition would suffice rather than the full 100-condition grid

## Inputs

- Splatter global parameter settings (5000 genes, 2 batches, 1000 total cells, 3 cell types)
- Parameter grid definition: cell-type effect location x batch effect location in {0, 0.25, 0.5, 0.75, 1} x balanced/unbalanced cell-type proportions
- R environment with the Splatter package installed

## Outputs

- 100 simulated gene-by-cell count matrices (5000 gene rows x 1000 cell columns each), serialized one file per condition (.rds or .csv)
- Condition metadata mapping each file to its cell-type effect location, batch effect location, and balanced/unbalanced proportion setting

## How to apply

Load the Splatter R package and configure global Splat parameters to 5000 genes, two batches totalling 1000 cells, and three cell types (groups) in similar proportions between batches. Build the parameter grid by varying the cell-type effect location parameter and the batch effect location parameter over {0, 0.25, 0.5, 0.75, 1}, each crossed with balanced and unbalanced cell-type proportion settings, yielding 100 distinct simulation conditions. For each condition, generate the gene-by-cell count matrix with Splatter's group-based simulation via splatSimulate, then serialize each count matrix to disk (e.g. .rds or .csv), one file per condition. This grid underpins the benchmark in which JIVE was compared against Seurat v5, Harmony, LIGER, and ComBat-seq, so condition metadata (effect locations, proportion setting) should be stored alongside each file for downstream performance stratification by balanced vs unbalanced batch scenarios.

## Related tools

- **Splatter** (Simulates the scRNA-seq count matrices for each of the 100 grid conditions using its group-based Splat model (splatSimulate))
- **R** (Environment in which Splatter parameters are configured, splatSimulate is run, and per-condition count matrices are serialized)

## Examples

```
library(Splatter); sim <- splatSimulate(groupCells = c(round(1000/3), round(1000/3), 1000-2*round(1000/3)), nGenes = 5000, batchCells = c(500, 500), method = "groups")
```

## Evaluation signals

- All 100 output files exist, one per simulated condition (row_count_equals / dimension check per condition)
- Each count matrix has exactly 5000 gene rows and 1000 cell columns
- Each condition uses exactly two batches and three cell types, with proportions balanced or unbalanced according to the grid setting
- Grid coverage: cell-type effect and batch effect location parameters each take all values in {0, 0.25, 0.5, 0.75, 1} across the condition set
- Downstream benchmark reproduces the expected stratification, e.g. method performance reported separately for balanced vs unbalanced batch-size scenarios

## Limitations

- Simulated data follow the Splat model assumptions, so conclusions may not transfer to real datasets with more complex batch structure
- Benchmark conclusions are conditioned on this fixed size (5000 genes, 1000 cells, 2 batches, 3 cell types); other data scales are not covered by the grid
- Simulation conditions deliberately isolate batch and cell-type effects, so unmodeled technical artifacts are absent from the benchmark

## Evidence

- [full_text] Simulated data were generated with the Splatter R package using a Splat model: each condition simulated 5000 genes, two batches with 1000 total cells, and three cell types in similar proportions between batches.: [evidence span withheld — anchor status: not_found]
- [full_text] Grid definition from workflow: [evidence span withheld — anchor status: not_found]
- [abstract] Benchmark context for which the grid was built: "JIVE performed best in terms of preserving cell-type effects and in scenarios in which the batch sizes are balanced."
- [readme] R environment installation: [evidence span withheld — anchor status: not_found]
