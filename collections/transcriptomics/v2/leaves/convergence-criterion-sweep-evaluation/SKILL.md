---
name: convergence-criterion-sweep-evaluation
description: Use when you need to test whether a simple k-NN majority-vote label propagation
  baseline matches a GCN annotation method, by sweeping k and convergence criteria
  against reference cell-type labels.
when_to_use_negative:
- The goal is GCN-based consensus annotation or feature/ gene interpretation rather
  than establishing a nonparametric baseline comparison
- The datasets lack reference cell-type labels, so no ground-truth accuracy can be
  computed for the sweep
- Input is a different prediction task than label propagation over a cell-cell k-NN
  graph (e.g. differential expression or clustering without labels)
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scSHARP
  role: Package providing the pipeline environment and benchmark data/notebooks (knn_experiment.ipynb,
    pbmc_knn_result_fixed.txt) used for the neighbor majority sweep
  repo: https://github.com/lewinsohndp/scSHARP
- name: pytorch-cluster
  role: Builds the k-NN graph on 500-dim PCA vectors in batches of 1000 cells for
    neighbor-majority propagation
- name: Python 3.9
  role: Runtime environment for running the sweep scripts and notebooks
- name: conda
  role: Environment management; pytorch-cluster installed from the pyg channel
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:da05dbad5912aae50962fa228f3d70f02d918ceeae3d2eb1a7541411df767f62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lewinsohndp/scSHARP
  tools:
  - Python
  - Python 3.9
  - conda
  - pip
  - pytorch-cluster
  - scSHARP
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# convergence-criterion-sweep-evaluation

## Summary

Benchmark the nonparametric neighbor majority label propagation baseline on scRNA-seq annotation by sweeping k neighbor values and three convergence strategies, then compare accuracies to the scSHARP GCN model on the same datasets.

## When to use

Use when you need to test whether a simple k-NN majority-vote label propagation baseline matches a GCN annotation method, by sweeping k and convergence criteria against reference cell-type labels.

## When NOT to use

- The goal is GCN-based consensus annotation or feature/ gene interpretation rather than establishing a nonparametric baseline comparison
- The datasets lack reference cell-type labels, so no ground-truth accuracy can be computed for the sweep
- Input is a different prediction task than label propagation over a cell-cell k-NN graph (e.g. differential expression or clustering without labels)

## Inputs

- scRNA-seq count matrices with reference cell-type labels (e.g. testis, PBMC, and simulated datasets used in the paper)
- PCA embeddings of cells reduced to 500 dimensions
- Sweep parameter grid: k neighbor values × 3 convergence strategies

## Outputs

- neighbor_majority_accuracy.csv: accuracy table for each k and convergence-strategy combination per dataset
- Per-dataset total accuracy and unconfident cell accuracy comparable to GCN results

## How to apply

Create a conda environment with Python 3.9 and install scSHARP via pip together with pytorch-cluster from the pyg channel. Load benchmark scRNA-seq datasets with reference cell-type labels and compute PCA embeddings reduced to 500 dimensions. Build a k-NN graph on the PCA vectors using pytorch-cluster, processing cells in batches of 1000 to bound memory. Run neighbor-majority label propagation — iteratively updating each cell's label by majority vote over labeled neighbors — sweeping k values against three convergence strategies: a single propagation round, stopping when fewer than 5% of labels change between iterations, and running until all cells are labeled or 50 epochs. Compute annotation accuracy (total and unconfident cell accuracy) against reference labels for each k and strategy combination, assemble into neighbor_majority_accuracy.csv, and check that selected settings match the paper's Table 1 and Supplementary Appendix SD sweep table.

## Related tools

- **scSHARP** (Package providing the pipeline environment and benchmark data/notebooks (knn_experiment.ipynb, pbmc_knn_result_fixed.txt) used for the neighbor majority sweep) — https://github.com/lewinsohndp/scSHARP
- **pytorch-cluster** (Builds the k-NN graph on 500-dim PCA vectors in batches of 1000 cells for neighbor-majority propagation)
- **Python 3.9** (Runtime environment for running the sweep scripts and notebooks)
- **conda** (Environment management; pytorch-cluster installed from the pyg channel)

## Examples

```
conda create -n scsharp python=3.9; conda activate scsharp; pip install scSHARP; conda install pytorch-cluster -c pyg
```

## Evaluation signals

- Accuracy values under the selected k and convergence settings must match the neighbor-majority accuracies reported in Table 1
- Sweep results should be comparable to the Supplementary Appendix SD sweep table
- On the testis dataset, the neighbor-majority accuracy must fall within the GCN's standard deviation for both total and unconfident cell accuracy
- Sweep table must contain one accuracy entry per (dataset, k, convergence-strategy) combination before writing neighbor_majority_accuracy.csv
- Convergence strategies must be checked: single round, <5% label-change stopping rule, and 50-epoch / fully-labeled cap are all represented

## Limitations

- The sweep is a baseline comparison; the GCN method was shown to improve accuracy over a simple consensus algorithm, so comparable performance to neighbor majority does not establish superiority over underlying tools
- Memory is bounded only by batching k-NN graph construction in groups of 1000 cells; larger datasets may need adjusted batch sizes
- Software version numbers for scSHARP and its dependencies are not specified; additional PyTorch Geometric dependencies such as torch-cluster may need manual installation
- The R tools (R4scSHARP) are necessary for running full scSHARP, though not for the pure neighbor-majority baseline

## Evidence

- [abstract] We also compare our method to a nonparametric neighbor majority approach, showing comparable results.: "We also compare our method to a nonparametric neighbor majority approach, showing comparable results."
- [readme] knn_experiment.ipynb - Results of non-parametric neighbor majority approach for simulated and testis data sets.: [evidence span withheld — anchor status: external_doc]
- [readme] pbmc_knn_result_fixed.txt - Results of non-parametric neighbor majority approach for PBMC data set.: [evidence span withheld — anchor status: external_doc]
- [methods] ``conda install pytorch-cluster -c pyg``: "``conda install pytorch-cluster -c pyg``"
