---
name: density-based-cluster-seed-selection
description: Use when initializing label-propagation clustering of single-cell RNA-seq
  data without a predetermined number of cell types, and seed cells must be selected
  from a directed kNN graph built on a quantitative cell-landmark matrix.
when_to_use_negative:
- Input is a raw gene-by-cell expression matrix that has not yet been preprocessed
  and converted into the Q1/Q2 cell-landmark representations by scQA's LC1 and LC2
  modules
- You already know the number of cell types and want a method that requires a predetermined
  cluster count — scQA's seed-based label propagation is designed to avoid specifying
  the number of cell types in advance
- The goal is clustering on the raw expression values rather than the landmark-based
  representation, where higher-density nodes are not meaningful seeds
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: 'Implements the CC module''s seed generation: groups cells by Q1 row vectors,
    computes node densities from the Q2-based kNN graph, applies threshold beta, and
    merges mutually neighboring seed candidates into uniquely labeled seeds'
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:699633ab8d06ee77ae0309ca0197c62d693cf341fed28b233f1079ff08598e39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - scQA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# density-based-cluster-seed-selection

## Summary

A seed-generation step from the scQA CC module that identifies authentic initial cluster cores by grouping cells with identical qualitative (Q1) row vectors, ranking nodes by neighbor-averaged density computed on the quantitative (Q2) matrix, and merging mutually neighboring seed candidates before label propagation.

## When to use

Use when initializing label-propagation clustering of single-cell RNA-seq data without a predetermined number of cell types, and seed cells must be selected from a directed kNN graph built on a quantitative cell-landmark matrix.

## When NOT to use

- Input is a raw gene-by-cell expression matrix that has not yet been preprocessed and converted into the Q1/Q2 cell-landmark representations by scQA's LC1 and LC2 modules
- You already know the number of cell types and want a method that requires a predetermined cluster count — scQA's seed-based label propagation is designed to avoid specifying the number of cell types in advance
- The goal is clustering on the raw expression values rather than the landmark-based representation, where higher-density nodes are not meaningful seeds

## Inputs

- Qualitative cell-landmark matrix Q1 (binary, from scQA LC1 module)
- Quantitative cell-landmark matrix Q2 (from scQA LC2 module)
- Directed k-nearest-neighbor graph over cells built from Pearson correlation similarity on Q2

## Outputs

- Set of merged seed cells, each assigned a unique initial cluster label
- Seed density threshold beta (minimum of per-group maximum densities)

## How to apply

After constructing the qualitative (Q1) and quantitative (Q2) cell-landmark matrices via scQA's LC1 and LC2 modules, build a directed k-nearest-neighbor graph from Pearson correlations on Q2, giving each node 0.1n out-neighbors when n<10,000 and 0.05n when n>=10,000. Group cells sharing identical Q1 row vectors, compute each node's density as the average similarity to its neighbors, and set the seed threshold beta to the minimum of the per-group maximum densities. Nodes whose density exceeds beta become seed candidates; merge mutually neighboring candidates into single seeds so each merged seed represents one cluster core. Assign each seed a unique initial cluster label, which then propagates in the subsequent label-propagation step with combinatorial coefficient alpha=0.5.

## Related tools

- **scQA** (Implements the CC module's seed generation: groups cells by Q1 row vectors, computes node densities from the Q2-based kNN graph, applies threshold beta, and merges mutually neighboring seed candidates into uniquely labeled seeds) — https://github.com/LD-Lyndee/scQA

## Examples

```
./main xin_matrix.txt o
```

## Evaluation signals

- Each seed node's density equals or exceeds beta, where beta is the minimum of per-group maximum densities across Q1-identical groups
- No two seeds are mutually nearest neighbors — all mutually neighboring seed candidates have been merged into single seeds
- Every seed carries a unique initial cluster label before label propagation begins
- Out-degree of every node in the directed kNN graph matches 0.1n (n<10,000) or 0.05n (n>=10,000) based on Pearson correlation on Q2
- Downstream labels.txt contains one cluster assignment per input cell with no cells left unlabeled at convergence

## Limitations

- Seed quality depends on the LC2-derived quantitative matrix; genes may be misclassified in LC1 and LC2 due to high dimensionality, which can corrupt density estimates
- Retaining only 2000 highly variable genes in upstream preprocessing excluded some informative genes, potentially affecting landmark matrices that feed seed selection
- The method was designed for and tested on scRNA-seq count data (e.g., Biase, Yan, Deng datasets); its behavior on other data modalities is not established

## Evidence

- [methods] In cluster constructor (CC) module, a directed k-nearest neighbor graph is constructed using the matrix generated by LC2: "In cluster constructor (CC) module, a directed k-nearest neighbor graph is constructed using the matrix generated by LC2"
- [methods] We first group cells based on Q1, ensuring that cells in the same group share identical row vector in Q1.: "We first group cells based on Q1, ensuring that cells in the same group share identical row vector in Q1."
- [methods] Intuitively, nodes with higher density are more likely to be authentic seeds.: "Intuitively, nodes with higher density are more likely to be authentic seeds."
- [methods] Each seed is assigned a unique label, which propagates to form a cluster.: "Each seed is assigned a unique label, which propagates to form a cluster."
- [methods] CC module clusters cells by combination of the two cell-landmark matrices derived from LC1 and LC2 modules: "CC module clusters cells by combination of the two cell-landmark matrices derived from LC1 and LC2 modules"
- [readme] The input file must be in the form of a matrix of cells * genes, without names of rows and columns.: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
