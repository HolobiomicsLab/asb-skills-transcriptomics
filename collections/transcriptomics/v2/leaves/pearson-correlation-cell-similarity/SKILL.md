---
name: pearson-correlation-cell-similarity
description: Use when clustering scRNA-seq cells in scQA's CC stage and a cell-cell
  similarity graph is needed from the quantitative representation Q2; also as reference
  for choosing out-neighbor counts (0.1n vs 0.05n) by dataset size.
when_to_use_negative:
- The input is still the raw or preprocessed gene-by-cell expression matrix rather
  than the low-dimensional Q2 landmark matrix — scQA computes the graph on LC2 output,
  not the original data
- The goal is the qualitative (binary) representation step itself; Q1 is used for
  grouping cells with identical row vectors, not for the Pearson-correlation graph
- The number of cell types is already known and fixed — scQA's label propagation obviates
  a predetermined cluster count, so a k-NN graph plus forced k-means-style assignment
  is mismatched
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Provides the CC module (Algorithm 3) that computes Pearson correlation similarities
    on Q2 and builds the directed k-NN graph for seed generation and label propagation
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:5cd055349bcc23fc6bfef0a0a454f37acc361886bf190f9b921d439a0487fd95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - scQA
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pearson-correlation-cell-similarity

## Summary

Builds the directed k-nearest-neighbor cell graph inside scQA's cluster constructor (CC) module by computing Pearson correlation similarities on the quantitative cell-landmark matrix Q2 produced by the LC2 module, so that density-based seed generation and label propagation can run over cell-to-cell similarity edges.

## When to use

Use when clustering scRNA-seq cells in scQA's CC stage and a cell-cell similarity graph is needed from the quantitative representation Q2; also as reference for choosing out-neighbor counts (0.1n vs 0.05n) by dataset size.

## When NOT to use

- The input is still the raw or preprocessed gene-by-cell expression matrix rather than the low-dimensional Q2 landmark matrix — scQA computes the graph on LC2 output, not the original data
- The goal is the qualitative (binary) representation step itself; Q1 is used for grouping cells with identical row vectors, not for the Pearson-correlation graph
- The number of cell types is already known and fixed — scQA's label propagation obviates a predetermined cluster count, so a k-NN graph plus forced k-means-style assignment is mismatched

## Inputs

- Quantitative cell-landmark matrix Q2 (cells x landmarks) produced by the scQA LC2 module
- Number of cells n (to select the 0.1n vs 0.05n out-neighbor count)
- Optionally, the qualitative matrix Q1 for grouping cells with identical row vectors prior to density-based seed selection

## Outputs

- Directed k-nearest-neighbor graph over cells with Pearson-correlation-weighted edges
- Per-node density values (average similarity to neighbors) used for seed generation and label propagation scoring

## How to apply

After the LC1 and LC2 modules have produced the qualitative matrix Q1 and quantitative matrix Q2, compute Pearson correlation similarity between cells using Q2 (Algorithm 3 of the scQA CC module), not Q1. From these similarities construct a directed k-nearest-neighbor graph: each cell node receives 0.1n out-neighbors when n < 10,000 cells and 0.05n when n >= 10,000. Each node's density is then defined as the average similarity to its neighbors, which drives seed-candidate selection (threshold beta = minimum of per-group maximum densities over Q1-identical cell groups) and the two-hop directed path similarities used in label propagation (alpha = 0.5). Rationale: Q2's quantitative low-dimensional representation preserves quasi-trend-preserved gene expression patterns, making Pearson correlation on it a more informative cell-cell similarity measure than correlation on the binary Q1 representation.

## Related tools

- **scQA** (Provides the CC module (Algorithm 3) that computes Pearson correlation similarities on Q2 and builds the directed k-NN graph for seed generation and label propagation) — https://github.com/LD-Lyndee/scQA

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git && cd scQA && ./make && ./main xin_matrix.txt o
```

## Evaluation signals

- Every node in the graph has exactly floor(0.1n) out-neighbors (n < 10,000) or floor(0.05n) out-neighbors (n >= 10,000), with edges directed from each cell to its most Pearson-similar cells in Q2
- Per-node density equals the average Pearson similarity to that node's graph neighbors; seed threshold beta equals the minimum of per-group maximum densities over Q1-identical cell groups
- Running the full pipeline ends with a labels.txt file containing one cluster label per cell, and the number of clusters emerges from propagation rather than a preset k
- Similarity scores stay in the valid Pearson range [-1, 1]; nodes are labeled only when S(ni,Cj) > S(ni,U), so no node receives a cluster label while an unlabeled-node score is higher
- Cross-check on benchmark datasets (e.g., Xin GSE81608, Yan GSE36552) that cluster counts approximate the well-annotated cell-type counts reported in the scQA evaluation

## Limitations

- Genes may be misclassified in LC1 and LC2 under high dimensionality, which can propagate noise into the Q2-based similarity graph
- Retaining only 2000 highly variable genes in pre-processing excludes some informative genes, constraining the signal available to the similarity computation
- Some clusters can be spurious — e.g., Cluster8 in the Xenopus lung dataset contained only three cells, likely clustering outliers

## Evidence

- [methods] In cluster constructor (CC) module, a directed k-nearest neighbor graph is constructed using the matrix generated by LC2: "In cluster constructor (CC) module, a directed k-nearest neighbor graph is constructed using the matrix generated by LC2"
- [methods] CC module clusters cells by combination of the two cell-landmark matrices derived from LC1 and LC2 modules: "CC module clusters cells by combination of the two cell-landmark matrices derived from LC1 and LC2 modules"
- [methods] Intuitively, nodes with higher density are more likely to be authentic seeds.: "Intuitively, nodes with higher density are more likely to be authentic seeds."
- [methods] Each seed is assigned a unique label, which propagates to form a cluster.: "Each seed is assigned a unique label, which propagates to form a cluster."
- [readme] labels.txt Clustering result.: "- **labels.txt** Clustering result."
