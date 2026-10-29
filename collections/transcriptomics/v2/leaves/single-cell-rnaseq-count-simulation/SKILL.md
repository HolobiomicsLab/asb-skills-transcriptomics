---
name: single-cell-rnaseq-count-simulation
description: Use when benchmarking batch-effect correction or cell-type preservation
  methods and controlled synthetic scRNA-seq data are needed with known ground-truth
  cell types, batches, and effect magnitudes.
when_to_use_negative:
- The analysis requires real biological data (e.g. the BRCA/TCGA multi-omics samples)
  rather than controlled synthetic benchmarks with known ground-truth cell types
- The target dataset has more than two batches or more than three cell types than
  the 2-batch / 3-group Splat configuration used here, unless the parameter grid is
  redefined
- Downstream tools require full-length transcript or multi-modal (methylation/miRNA)
  matrices rather than single-modality gene-by-cell count matrices
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: Splat-model simulator that generates the group-based scRNA-seq count matrices
    via splatSimulate for each of the 100 conditions
- name: R
  role: Execution environment for configuring Splatter parameters, simulating each
    condition, and serializing count matrices to .rds/.csv files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbae134
    title: Batch-effect correction in single-cell RNA sequencing data using JIVE
  source_content_hash: sha256:fa22a5b428bae7eef50dd12e5d2853b78e49296703c090f52e1994a140270651
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Splatter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-rnaseq-count-simulation

## Summary

Generates synthetic scRNA-seq count matrices with the Splatter R package (Splat model) to create controlled benchmark datasets in which cell-type effects, batch effects, and cell-type proportion balance are systematically varied. Used in this article to produce 100 simulation conditions for evaluating batch-effect correction methods.

## When to use

Use when benchmarking batch-effect correction or cell-type preservation methods and controlled synthetic scRNA-seq data are needed with known ground-truth cell types, batches, and effect magnitudes.

## When NOT to use

- The analysis requires real biological data (e.g. the BRCA/TCGA multi-omics samples) rather than controlled synthetic benchmarks with known ground-truth cell types
- The target dataset has more than two batches or more than three cell types than the 2-batch / 3-group Splat configuration used here, unless the parameter grid is redefined
- Downstream tools require full-length transcript or multi-modal (methylation/miRNA) matrices rather than single-modality gene-by-cell count matrices

## Inputs

- Parameter grid specification: cell-type effect location parameter and batch effect location parameter over {0, 0.25, 0.5, 0.75, 1}, crossed with balanced/unbalanced cell-type proportion settings (100 conditions)
- Global Splat simulation parameters: nGenes=5000, 2 batches totalling 1000 cells, 3 cell-type groups

## Outputs

- 100 simulated scRNA-seq count matrices (genes as rows, cells as columns), one per condition
- One serialized .rds (or .csv) file per simulation condition

## How to apply

Load Splatter in R and configure global Splat parameters: 5000 genes, 2 batches totalling 1000 cells, and 3 cell types (groups) in similar proportions between batches. Build a parameter grid by crossing the cell-type effect location parameter and the batch effect location parameter over {0, 0.25, 0.5, 0.75, 1} with balanced and unbalanced cell-type proportion settings, yielding 100 distinct simulation conditions. For each condition, simulate the count matrix with Splatter's group-based simulation (splatSimulate), keeping genes as rows and cells as columns. Serialize each condition's count matrix to disk as a .rds (or .csv) file, one file per condition, so downstream correction methods (e.g. JIVE, Seurat v5, Harmony, LIGER, ComBat-seq) consume identical simulated inputs.

## Related tools

- **Splatter** (Splat-model simulator that generates the group-based scRNA-seq count matrices via splatSimulate for each of the 100 conditions)
- **R** (Execution environment for configuring Splatter parameters, simulating each condition, and serializing count matrices to .rds/.csv files)

## Examples

```
library(Splatter); sim <- splatSimulate(newSplatParams(nGenes=5000, batchCells=c(500,500), group.prob=c(1/3,1/3,1/3)), method="groups"); saveRDS(counts(sim), "condition_001.rds")
```

## Evaluation signals

- All 100 output files exist after the simulation run (one per grid condition)
- Each simulated count matrix has exactly 5000 gene rows and 1000 cell columns (dimension check per condition)
- Each condition contains exactly 2 batches summing to 1000 total cells and 3 cell types in similar proportions between batches
- Balanced vs unbalanced cell-type proportion settings are represented in the grid alongside the 5x5 crossing of cell-type and batch effect location parameters
- Downstream benchmarking on balanced scenarios should be interpretable since JIVE was reported to perform best 'in scenarios in which the batch sizes are balanced'

## Limitations

- Simulation conditions are fixed to 5000 genes, 2 batches with 1000 total cells, and 3 cell types; other data regimes require re-deriving the grid
- Findings drawn from these simulations apply to balanced-batch scenarios in particular: JIVE performed best 'in terms of preserving cell-type effects and in scenarios in which the batch sizes are balanced'
- Simulated Splat-model data may not capture all real-data technical artifacts; the article notes real multi-source data (e.g. BRCA/TCGA) are analyzed separately

## Evidence

- [other] Simulated data were generated with the Splatter R package using a Splat model: each condition simulated 5000 genes, two batches with 1000 total cells, and three cell types: [evidence span withheld — anchor status: not_found]
- [other] varying the cell-type effect location parameter and the batch effect location parameter over {0, 0.25, 0.5, 0.75, 1}, crossed with balanced and unbalanced cell-type proportion settings, yielding 100 simulation conditions: [evidence span withheld — anchor status: not_found]
- [abstract] JIVE performed best in terms of preserving cell-type effects and in scenarios in which the batch sizes are balanced.: "JIVE performed best in terms of preserving cell-type effects and in scenarios in which the batch sizes are balanced."
- [readme] These factors cause unwanted variation in the data and often obfuscate the underlying biological signal of interest.: "These factors cause unwanted variation in the data and often obfuscate the underlying biological signal of interest."
