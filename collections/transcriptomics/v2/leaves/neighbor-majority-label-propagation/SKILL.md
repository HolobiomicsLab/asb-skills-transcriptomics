---
name: neighbor-majority-label-propagation
description: Use when benchmarking a semi-supervised scRNA-seq label propagation model
  (e.g. scSHARP's GCN) against a simple nonparametric neighbor-voting baseline on
  PCA-embedded cells with reference labels.
when_to_use_negative:
- The goal is feature interpretation or identifying important genes for classification
  — the neighbor-majority baseline offers no interpretability, unlike the GCN method.
- The task is to improve accuracy over the consensus of component tools rather than
  to benchmark against a simple baseline — use the scSHARP GCN label propagation instead.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scSHARP
  role: Package whose GCN label propagation the neighbor-majority baseline benchmarks;
    installed via pip in a Python 3.9 conda environment
  repo: https://github.com/lewinsohndp/scSHARP
- name: pytorch-cluster
  role: Builds the k-NN graph on PCA vectors for neighbor voting, installed from the
    pyg channel
  repo: https://github.com/mperozek11/scSHARP_tool
- name: PyTorch
  role: Backend for the k-NN graph construction and propagation experiment
  repo: https://github.com/mperozek11/scSHARP_tool
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:5fa5f322e40e2a000914c4fca66890e80e6f363ff629f3f4a217a7b5a4ab98dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lewinsohndp/scSHARP
  tools:
  - PyTorch
  - Python
  - conda
  - pip
  - pytorch-cluster
  - scSHARP
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# neighbor-majority-label-propagation

## Summary

A nonparametric baseline for scRNA-seq cell type annotation that assigns each cell the majority label of its k nearest PCA-embedding neighbors, iterated to convergence. It benchmarks the scSHARP GCN label propagation model and shows comparable accuracy on simulated and real datasets.

## When to use

Use when benchmarking a semi-supervised scRNA-seq label propagation model (e.g. scSHARP's GCN) against a simple nonparametric neighbor-voting baseline on PCA-embedded cells with reference labels.

## When NOT to use

- The goal is feature interpretation or identifying important genes for classification — the neighbor-majority baseline offers no interpretability, unlike the GCN method.
- The task is to improve accuracy over the consensus of component tools rather than to benchmark against a simple baseline — use the scSHARP GCN label propagation instead.

## Inputs

- scRNA-seq dataset with reference cell-type labels (e.g. the four benchmark simulated/real datasets, including testis and PBMC)
- PCA embedding of cell expression profiles, reduced to 500 dimensions
- k-NN graph over PCA vectors built with pytorch-cluster (batch size 1000)
- k neighbor values and convergence strategy settings to sweep

## Outputs

- Accuracy table of annotation accuracy per k and convergence strategy per dataset (neighbor_majority_accuracy.csv)
- Comparison of baseline accuracy against GCN total and unconfident cell accuracy (Table 1 / Supplementary Appendix SD sweep table)

## How to apply

Compute PCA embeddings of the scRNA-seq count matrix reduced to 500 dimensions, then build a k-NN graph on the PCA vectors with pytorch-cluster, processing cells in batches of 1000 to keep memory bounded. Run neighbor-majority label propagation by iteratively updating each cell's label via majority vote over its labeled neighbors, sweeping over multiple k values and three convergence strategies: a single propagation round, stopping when fewer than 5% of labels change between iterations, and running until all cells are labeled or 50 epochs. Compute annotation accuracy against the reference cell-type labels for every k/convergence-strategy combination on each dataset (e.g. the four benchmark scRNA-seq datasets including the testis and PBMC sets). Assemble the accuracy table and compare against the GCN's accuracy and standard deviation to judge whether the baseline is comparable.

## Related tools

- **scSHARP** (Package whose GCN label propagation the neighbor-majority baseline benchmarks; installed via pip in a Python 3.9 conda environment) — https://github.com/lewinsohndp/scSHARP
- **pytorch-cluster** (Builds the k-NN graph on PCA vectors for neighbor voting, installed from the pyg channel) — https://github.com/mperozek11/scSHARP_tool
- **PyTorch** (Backend for the k-NN graph construction and propagation experiment) — https://github.com/mperozek11/scSHARP_tool

## Examples

```
conda create -n scsharp python=3.9 && conda install pytorch-cluster -c pyg && pip install scSHARP
```

## Evaluation signals

- Accuracy values under the selected k and convergence settings match the neighbor-majority accuracies reported in Table 1.
- Accuracy values are comparable to the Supplementary Appendix SD sweep table.
- On the testis dataset, baseline accuracy falls within the GCN's standard deviation for both total and unconfident cell accuracy.
- The sweep covers all three convergence strategies: single round, <5% label change stopping, and run-to-completion at 50 epochs.
- Output accuracies assemble into neighbor_majority_accuracy.csv without missing k/strategy combinations.

## Limitations

- The baseline shows only comparable, not improved, results relative to the GCN method and provides no feature interpretation.
- Additional PyTorch Geometric dependencies such as torch-cluster may need to be manually installed depending on the system.
- Software version numbers for scSHARP and dependencies, and the R version requirement, are not specified in the article.

## Evidence

- [abstract] nonparametric neighbor majority: "We also compare our method to a nonparametric neighbor majority approach, showing comparable results."
- [methods] conda python 3.9: "``conda create -n <env name> python=3.9`` ``conda activate <env name>``"
- [methods] pytorch-cluster install: "``conda install pytorch-cluster -c pyg``"
- [readme] knn experiment notebook: [evidence span withheld — anchor status: external_doc]
- [readme] pbmc knn results: [evidence span withheld — anchor status: external_doc]
