---
name: trajectory-path-simulation-between-clusters
description: Use when benchmarking single-cell DEG prediction methods on simulated
  data where cluster structure and continuous trajectories between clusters are both
  required, and ground-truth DE labels are needed for accuracy evaluation.
when_to_use_negative:
- When the goal is to benchmark methods on realistic biological variation rather than
  Splatter's cluster-centric generative model, since its cluster-based structure can
  bias results toward cluster-based DEG approaches
- When a real transcriptome dataset with known DEG labels is already available (e.g.
  the Tabula Muris thymus dataset used in the paper)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Splatter
  role: Simulation package used to generate artificial scRNA-seq datasets via splatSimulate
    in group mode, with and without path.from trajectory connections (method='paths')
- name: R
  role: Language in which the Splatter simulation workflow is implemented and run
- name: singleCellHaystack
  role: Downstream DEG prediction method whose accuracy on the simulated datasets
    is being benchmarked
  repo: https://github.com/alexisvdb/singleCellHaystack
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:38d4af005bcb3e8eaabe48dc365e8b2e87e80bca315ad51ac5bead5cdabb1450
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Splatter
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Trajectory path simulation between clusters

## Summary

Generates artificial scRNA-seq datasets in which clusters of cells are connected by simulated differentiation paths using Splatter's path mode (method='paths' with path.from), producing 200 benchmark count matrices with binary DEG ground-truth labels to test DEG prediction methods such as singleCellHaystack.

## When to use

Use when benchmarking single-cell DEG prediction methods on simulated data where cluster structure and continuous trajectories between clusters are both required, and ground-truth DE labels are needed for accuracy evaluation.

## When NOT to use

- When the goal is to benchmark methods on realistic biological variation rather than Splatter's cluster-centric generative model, since its cluster-based structure can bias results toward cluster-based DEG approaches
- When a real transcriptome dataset with known DEG labels is already available (e.g. the Tabula Muris thymus dataset used in the paper)

## Inputs

- R environment with Splatter package installed
- Parameter grid specification (10 settings: 1000 cells/2 clusters up to 10,000 cells/25 clusters)
- Random seed value for reproducibility
- path.from configuration defining cluster-to-cluster trajectory connections

## Outputs

- 200 simulated count matrices (100 default group-mode, 100 path-connected)
- 200 binary DEG ground-truth label files (positives = genes with DEFac != 1)
- File names encoding cells/clusters/replicate/path settings

## How to apply

Set up an R environment with the Splatter package and fix a random seed for reproducibility. Define a parameter grid of 10 simulation settings spanning 1000 cells/2 clusters to 10,000 cells/25 clusters; for each setting call splatSimulate in group mode twice: once with the default group structure and once with random paths connecting clusters via path.from and method='paths', generating 10 replicates each (100 datasets per condition, 200 total). Extract the counts matrix from each simulation and derive binary DEG ground-truth labels by flagging genes with a differential expression factor (DEFac) different from 1. Write the 200 count matrices and 200 label files to disk with consistent names encoding cells/clusters/replicate/path settings. The rationale is that Splatter's cluster-based generative model is expected to favor cluster-based DEG methods, so path-connected simulations probe methods like singleCellHaystack that operate without explicit clustering.

## Related tools

- **Splatter** (Simulation package used to generate artificial scRNA-seq datasets via splatSimulate in group mode, with and without path.from trajectory connections (method='paths'))
- **R** (Language in which the Splatter simulation workflow is implemented and run)
- **singleCellHaystack** (Downstream DEG prediction method whose accuracy on the simulated datasets is being benchmarked) — https://github.com/alexisvdb/singleCellHaystack

## Examples

```
library(Splatter); set.seed(42); sim <- splatSimulate(groupCells = 5000, nGroups = 10, method = 'paths', path.from = c(1,2,3), batchCells = 5000)
```

## Evaluation signals

- Exactly 200 count-matrix files and 200 label files exist on disk after the run
- Each count matrix has dimensions matching the requested cells and genes for its simulation setting
- Every label file contains only binary entries
- Positives in each label file correspond exactly to genes with DEFac != 1 in the corresponding simulation parameters
- Re-running with the same random seed reproduces identical simulated datasets

## Limitations

- Splatter's generative model is based on generating clusters of cells, so the artificial benchmark datasets are expected to give an advantage to cluster-based DEG prediction methods
- Simulated path structure may not reflect real trajectory biology, limiting generalization of benchmark results to real single-cell data

## Evidence

- [abstract] Splatter datasets favor cluster-based methods: "Comparisons with existing DEG prediction approaches on artificial datasets show that singleCellHaystack has higher accuracy"
- [readme] singleCellHaystack README on scope: "`singleCellHaystack` uses Kullback-Leibler
Divergence to find features that have patterns of activity in subsets of
samples that are non-randomly positioned inside any input space."
- [full_text] Splatter simulation task workflow: [evidence span withheld — anchor status: not_found]
- [full_text] DEFac ground-truth labels: [evidence span withheld — anchor status: not_found]
