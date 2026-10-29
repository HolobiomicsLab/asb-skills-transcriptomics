---
name: cell-landmark-matrix-fusion
description: Use when scRNA-seq data have already been reduced by scQA's LC1 and LC2
  modules into qualitative (Q1, binary) and quantitative (Q2) cell-landmark matrices
  and cells must be clustered without predetermining the number of cell types.
when_to_use_negative:
- Input is a raw cells x genes count matrix that has not been processed through LC1/LC2
  to produce the Q1 and Q2 landmark matrices — run scQA's upstream modules first
- The number of cell types must be fixed in advance — scQA's label propagation deliberately
  obviates a predetermined cluster count
- Gene-level (rather than cell-level) analysis is the goal — this skill only assigns
  cells to clusters
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: scQA
  role: Provides the CC module (Algorithm 3) that constructs the directed kNN graph,
    generates seeds, and runs label propagation over the fused Q1/Q2 cell-landmark
    matrices to output cluster labels
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:e0edee92168118ecc49cb10609ec26076bc3e32e8233c53e962e38fdbb8d8fd0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - scQA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cell-landmark-matrix-fusion

## Summary

scQA's Cluster Constructor (CC) stage: fuses the qualitative (Q1, binary) and quantitative (Q2) cell-landmark matrices from LC1/LC2 into cell clusters via a directed kNN graph, seed generation, and label propagation—no predetermined number of cell types required.

## When to use

Use when scRNA-seq data have already been reduced by scQA's LC1 and LC2 modules into qualitative (Q1, binary) and quantitative (Q2) cell-landmark matrices and cells must be clustered without predetermining the number of cell types.

## When NOT to use

- Input is a raw cells x genes count matrix that has not been processed through LC1/LC2 to produce the Q1 and Q2 landmark matrices — run scQA's upstream modules first
- The number of cell types must be fixed in advance — scQA's label propagation deliberately obviates a predetermined cluster count
- Gene-level (rather than cell-level) analysis is the goal — this skill only assigns cells to clusters

## Inputs

- Qualitative cell-landmark matrix Q1 (binary, from scQA LC1 module)
- Quantitative cell-landmark matrix Q2 (from scQA LC2 module)
- Cells x genes preprocessed matrix (from pre.R, cells as rows, no row/column names)

## Outputs

- labels.txt: final per-cell cluster labels

## How to apply

Combine the Q1 binary cell-landmark matrix (from LC1) and the Q2 quantitative cell-landmark matrix (from LC2): build a directed k-nearest-neighbor graph over cells using Pearson correlation similarity on Q2, giving each node 0.1n out-neighbors when n<10,000 and 0.05n when n>=10,000 (Algorithm 3). Group cells sharing identical Q1 row vectors, compute each node's density as its average similarity to neighbors, set the seed threshold beta to the minimum of per-group maximum densities, merge mutually neighboring seed candidates, and assign each seed a unique label. Run label propagation with combinatorial coefficient alpha=0.5, scoring unlabeled nodes against clusters via two-hop directed path similarities (Eq. 10) and against unlabeled nodes (Eq. 11), labeling a node only when S(ni,Cj) > S(ni,U); prune small clusters and merge Ci into CJ when S(Ci,CJ) exceeds the number of edges within Ci (preventing re-merge of assimilated clusters). Expand clusters by relabeling unlabeled nodes with S(ni,Cj)=|ONiCj|-A(Cj), create new clusters for nodes negative against all clusters, and iterate label propagation to convergence, writing per-cell labels to labels.txt.

## Related tools

- **scQA** (Provides the CC module (Algorithm 3) that constructs the directed kNN graph, generates seeds, and runs label propagation over the fused Q1/Q2 cell-landmark matrices to output cluster labels) — https://github.com/LD-Lyndee/scQA

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git && cd scQA && ./make && ./main xin_matrix.txt o
```

## Evaluation signals

- Every cell receives exactly one label in labels.txt (no unlabeled cells after convergence)
- Number of clusters emerges from seed count and pruning/merging — verify no cluster is empty and no cluster remains at size below the pruning threshold
- Clustering accuracy (e.g., ARI/NMI against annotated cell types) should exceed baseline tools (SC3, Seurat, RCSL, CIDR) as reported across the 20 validated datasets (Biase GSE57249, Yan GSE36552, Deng GSE45719, Xin GSE81608, etc.)
- Check that kNN graph out-degree matches 0.1n (n<10,000) or 0.05n (n>=10,000) neighbors per node
- Clusters flagged as merged (Ci into CJ) must not be merged again — verify the assimilation-prevention invariant holds in the merge log

## Limitations

- Retaining only 2000 highly variable genes in upstream preprocessing excluded some informative genes, which can degrade cluster quality
- Genes may be misclassified in LC1 and LC2 under high dimensionality, propagating errors into the Q1/Q2 matrices fused here
- Occasional biologically distinct cell types can be fused: endothelial and mesenchymal cells were incorrectly grouped into a single cluster (Cluster3), and Cluster8 in the Xenopus lung dataset contained only three likely outlier cells
- Only tested on Linux servers per the README; input matrix must lack row/column names and binning parameter defaults to 6

## Evidence

- [methods] CC module clusters cells by combination of the two cell-landmark matrices derived from LC1 and LC2 modules: "CC module clusters cells by combination of the two cell-landmark matrices derived from LC1 and LC2 modules"
- [methods] directed kNN graph: "In cluster constructor (CC) module, a directed k-nearest neighbor graph is constructed using the matrix generated by LC2"
- [methods] seed labels propagate: "Each seed is assigned a unique label, which propagates to form a cluster."
- [methods] cluster expansion: "All unlabeled vertices are either added to existing clusters or isolated to create new ones in the cluster expansion step."
- [abstract] no predetermined cluster number: "It then clusters cells by employing an ingenious label propagation strategy, obviating the requirement for a predetermined number of cell types."
- [readme] labels.txt output: "- **labels.txt** Clustering result."
