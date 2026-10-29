---
name: single-cell-rna-seq-count-data-simulation
description: Use when you need synthetic scRNA-Seq datasets with known cell-type labels
  to benchmark clustering or transfer-learning methods, especially across controlled
  source-target cell-type overlap scenarios.
when_to_use_negative:
- The task requires real, biologically observed scRNA-Seq data (e.g., Usoskin, Hockley,
  or Tasic datasets) rather than artificial ground-truth data
- Source and target datasets share no cell types and the goal is transfer of biologically
  meaningful reference knowledge rather than controlled overlap benchmarking (ground
  truth is simulated, not measured)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scRNA (cmd_generate_data.py)
  role: Command line script used to generate artificial/simulated scRNA-Seq datasets
    with source and target splits and ground-truth labels
  repo: https://github.com/nicococo/scRNA/blob/master/scRNA/cmd_generate_data.py
- name: SC3
  role: Downstream clustering method applied to the simulated target data when benchmarking
    transfer-learning performance against baselines
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:e8eea2e34816461ab8db1a3ea6b85af0e118fc98afe42ec0465596b93ee4b255
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SC3
  - scRNA (cmd_generate_data.py)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-cell-rna-seq-count-data-simulation

## Summary

Simulates synthetic single-cell RNA-Seq count matrices with known ground-truth cluster labels using the scRNA package's data-generation script, enabling controlled benchmarking of transfer-learning clustering (TransferCluster vs. TargetCluster vs. ConcatenateCluster) across source/target overlap conditions.

## When to use

Use when you need synthetic scRNA-Seq datasets with known cell-type labels to benchmark clustering or transfer-learning methods, especially across controlled source-target cell-type overlap scenarios.

## When NOT to use

- The task requires real, biologically observed scRNA-Seq data (e.g., Usoskin, Hockley, or Tasic datasets) rather than artificial ground-truth data
- Source and target datasets share no cell types and the goal is transfer of biologically meaningful reference knowledge rather than controlled overlap benchmarking (ground truth is simulated, not measured)

## Inputs

- Simulation parameters (number of clusters, number of genes, number of cells, Dirichlet concentration, DE fraction, fold-change distribution, NB dispersion)
- Overlap scenario specification (complete, incomplete, or no overlap between source and target cell types)
- Target cell sizes for downsampling (e.g., 10, 50, 100, 200, 400, 600, 800)

## Outputs

- Simulated count-level scRNA-Seq source and target data matrices
- Gene id files
- Source- and target ground-truth cluster labels
- Aggregated ARI results table (e.g., simulation_ari_results.csv) after downstream clustering evaluation

## How to apply

Generate count-level datasets with the scRNA package's simulation script (scRNA/simulation.py, cmd_generate_data.py): e.g., 8 clusters derived from 5 top nodes, 10,000 genes, 1800 cells, cluster sizes drawn from a Dirichlet distribution (concentration 10), gamma background expression (shape 2, rate 0.1), 10-40% differentially expressed genes per cluster with log2 fold changes ~ N(1, 0.5), and negative binomial count sampling (dispersion 0.1). Split each dataset into a source dataset (e.g., 1000 cells) and a target dataset downsampled to sizes such as 10, 50, 100, 200, 400, 600, or 800 cells, assigning top-node clusters to source/target according to a complete-overlap, incomplete-overlap, or no-overlap scenario. Save source and target data, gene ids, and ground-truth labels so that Adjusted Rand Index can later be computed on target cells only. Vary the scenario and target size across 100 repetitions per condition to support aggregated ARI estimates with 95% confidence intervals.

## Related tools

- **scRNA (cmd_generate_data.py)** (Command line script used to generate artificial/simulated scRNA-Seq datasets with source and target splits and ground-truth labels) — https://github.com/nicococo/scRNA/blob/master/scRNA/cmd_generate_data.py
- **SC3** (Downstream clustering method applied to the simulated target data when benchmarking transfer-learning performance against baselines)

## Examples

```
scRNA-generate-data.sh  # installed via pip install git+https://github.com/nicococo/scRNA.git
```

## Evaluation signals

- Each generated run produces the expected set of files: gene ids, source- and target data, and source- and target ground truth labels
- Datasets match the specified simulation design (e.g., 10,000 genes, 1800 cells, 8 clusters; 100 sets per overlap setting)
- Downstream ARI table shows TransferCluster mean ARI above TargetCluster and ConcatenateCluster for complete- and incomplete-overlap settings across target sizes
- In the no-overlap setting, TransferCluster must not significantly degrade performance relative to TargetCluster (overlapping 95% confidence intervals), consistent with Figure 2B-D

## Limitations

- Simulated data distributions (Dirichlet sizes, gamma background, NB counts) may not capture all real scRNA-Seq characteristics such as dropout or batch effects
- Performance claims from simulated data must be complemented by evaluation on real datasets (Tasic subsampling, Hockley/Usoskin transfer)
- Overlapping only partially specified gene sets between source and target cause loss of information, which the authors defer to future work

## Evidence

- [methods] Simulated data generation design: "100 sets of simulated data were generated for each of the three settings simulating the expression levels of 10 000 genes in 1800 cells"
- [supplementary] Data generation script location: "The command line script for simulating scRNA-Seq datasets is available at https://github.com/nicococo/scRNA/blob/master/scRNA/cmd_generate_data.py"
- [readme] Package capability: "This package contains methods for generating artificial data, clustering, and transfering knowledge from a source to a target dataset."
- [readme] Data generation output files: "This will result in a number of files:
* Gene ids
* Source- and target data
* Source- and target ground truth labels"
