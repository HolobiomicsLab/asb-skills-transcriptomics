---
name: label-propagation-cell-clustering
description: Use when single-cell RNA-seq cells must be clustered from dual low-dimensional
  cell-landmark representations (binary Q1 from LC1 and quantitative Q2 from LC2)
  with an unknown number of cell types.
when_to_use_negative:
- The number of cell types is known in advance and a fixed-k method (e.g., k-means-based
  consensus like SC3) is preferred
- Input is the raw count matrix rather than the LC1/LC2 landmark representations —
  run pre-processing and landmark construction first
- The dataset is so large that graph-based approaches exhaust memory (cf. SIMLR failing
  on the Shekhar dataset due to excessive memory usage)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: 'Implements the CC module: builds the directed k-NN graph from Q2, generates
    seeds from Q1 groups, and runs label propagation, pruning/merging, and cluster
    expansion to produce labels.txt'
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:49db78de115ff94aa610df131fa255e5b74ec2133ee54db4cb1f10a37dd947ba
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
yes_rate: 0.8181818181818182
n_claims: 11
---

# label-propagation-cell-clustering

## Summary

scQA's cluster constructor (CC) stage that combines the qualitative (LC1) and quantitative (LC2) cell-landmark matrices to assign cells to clusters via seed generation and label propagation on a directed k-NN graph, without requiring a predetermined number of cell types.

## When to use

Use when single-cell RNA-seq cells must be clustered from dual low-dimensional cell-landmark representations (binary Q1 from LC1 and quantitative Q2 from LC2) with an unknown number of cell types.

## When NOT to use

- The number of cell types is known in advance and a fixed-k method (e.g., k-means-based consensus like SC3) is preferred
- Input is the raw count matrix rather than the LC1/LC2 landmark representations — run pre-processing and landmark construction first
- The dataset is so large that graph-based approaches exhaust memory (cf. SIMLR failing on the Shekhar dataset due to excessive memory usage)

## Inputs

- Qualitative cell-landmark matrix Q1 (binary, from scQA LC1 module)
- Quantitative cell-landmark matrix Q2 (from scQA LC2 module)
- Pre-processed cells x genes expression matrix (no row/column names) prepared by pre.R

## Outputs

- labels.txt: per-cell cluster labels (final clustering result)

## How to apply

Load the qualitative matrix Q1 (LC1 output) and quantitative matrix Q2 (LC2 output), then build a directed k-nearest-neighbor graph over cells using Pearson correlation similarity on Q2, giving each node 0.1n out-neighbors when n<10,000 and 0.05n when n>=10,000. Group cells sharing identical Q1 row vectors, compute each node's density as the average similarity to its neighbors, set the seed threshold beta to the minimum of the per-group maximum densities, merge mutually neighboring seed candidates, and assign each seed a unique initial cluster label. Run label propagation with combinatorial coefficient alpha=0.5, scoring unlabeled nodes against clusters via two-hop directed path similarities and against other unlabeled nodes, labeling a node only when S(ni,Cj) > S(ni,U). Prune small clusters, merge cluster Ci into larger CJ whenever S(Ci,CJ) exceeds the number of edges within Ci (preventing re-merging of assimilated clusters), then expand clusters by relabeling unlabeled nodes with S(ni,Cj)=|ONiCj|-A(Cj), creating new clusters for nodes negative against all clusters, and repeat until convergence. Write per-cell cluster labels to labels.txt.

## Related tools

- **scQA** (Implements the CC module: builds the directed k-NN graph from Q2, generates seeds from Q1 groups, and runs label propagation, pruning/merging, and cluster expansion to produce labels.txt) — https://github.com/LD-Lyndee/scQA

## Examples

```
./main xin_matrix.txt o
```

## Evaluation signals

- labels.txt exists and contains one cluster label per input cell with no unlabeled nodes remaining after convergence
- Seed threshold beta equals the minimum of per-group maximum densities computed from cells grouped by identical Q1 row vectors
- Graph degree follows the specified rule: 0.1n out-neighbors for n<10,000 cells and 0.05n for n>=10,000
- Merged clusters that have assimilated other clusters are not merged again (S(Ci,CJ) vs. edge-count check honored)
- External validation: cluster assignments match well-annotated cell type labels on benchmark datasets (e.g., Yan GSE36552, Xin GSE81608)

## Limitations

- Retaining only 2000 highly variable genes upstream excluded some informative genes, which can degrade clustering
- Genes may be misclassified in LC1 and LC2 due to high dimensionality, corrupting the landmark matrices fed to CC
- Small artifacts can persist: e.g., Cluster8 in the Xenopus lung dataset contained only three cells, likely outliers, and lacked marker genes; endothelial and mesenchymal cells were incorrectly grouped into a single cluster (Cluster3)

## Evidence

- [abstract] It then clusters cells by employing an ingenious label propagation strategy, obviating the requirement for a predetermined number of cell types.: "It then clusters cells by employing an ingenious label propagation strategy, obviating the requirement for a predetermined number of cell types."
- [methods] In cluster constructor (CC) module, a directed k-nearest neighbor graph is constructed using the matrix generated by LC2: "In cluster constructor (CC) module, a directed k-nearest neighbor graph is constructed using the matrix generated by LC2"
- [methods] We first group cells based on Q1, ensuring that cells in the same group share identical row vector in Q1.: "We first group cells based on Q1, ensuring that cells in the same group share identical row vector in Q1."
- [methods] All unlabeled vertices are either added to existing clusters or isolated to create new ones in the cluster expansion step.: "All unlabeled vertices are either added to existing clusters or isolated to create new ones in the cluster expansion step."
- [readme] The input file must be in the form of a matrix of cells * genes, without names of rows and columns.: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
