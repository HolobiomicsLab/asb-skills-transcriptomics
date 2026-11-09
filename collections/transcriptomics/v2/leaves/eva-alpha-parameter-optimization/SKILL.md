---
name: eva-alpha-parameter-optimization
description: Use when running ELVAR/EVA clustering on a cell-cell nearest-neighbor
  graph with a vertex attribute (age, disease stage/severity) and an optimal alpha
  balancing attribute-similarity against state-manifold topology must be chosen before
  downstream DA testing.
when_to_use_negative:
- When no meaningful cell attribute exists to guide clustering — alpha=0 standard
  Louvain (FindClusters) is the appropriate benchmark instead
- When the cell-cell graph is disconnected or the vertex attribute has many false
  positives (ELVAR is more sensitive to FPs in the sample/phenotype label), which
  degrades inferred communities
- When data have strong clustering structure, where ELVAR's improvement over Louvain
  is less noticeable and extra runtime is not justified
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: ELVAR
  role: R package providing Eva_partitions and SelOptAlpha for alpha-parameter sweep
    and optimal-alpha selection
  repo: https://github.com/aet21/ELVAR
- name: Seurat
  role: Builds the Seurat object, runs FindVariableFeatures/ScaleData/RunPCA/FindNeighbors
    to produce the kNN graph consumed by EVA
- name: igraph
  role: Constructs the undirected graph (graph.adjacency) and stores vertex attributes
    used by Eva_partitions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:f8a49478b192596798763e6a6b1f65a9218e1aeb555c7a2fd97489288019c7cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - MASS
  - Matrix
  - R
  - Seurat
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# eva-alpha-parameter-optimization

## Summary

Selects an optimal alpha parameter for EVA cell-attribute aware community detection by running Eva_partitions 100 times across alpha = 0.1–0.9 and applying SelOptAlpha criteria on purity, modularity, and cluster count. Alpha balances contribution of cell attributes (e.g. disease severity) versus graph topology in the Louvain objective.

## When to use

Use when running ELVAR/EVA clustering on a cell-cell nearest-neighbor graph with a vertex attribute (age, disease stage/severity) and an optimal alpha balancing attribute-similarity against state-manifold topology must be chosen before downstream DA testing.

## When NOT to use

- When no meaningful cell attribute exists to guide clustering — alpha=0 standard Louvain (FindClusters) is the appropriate benchmark instead
- When the cell-cell graph is disconnected or the vertex attribute has many false positives (ELVAR is more sensitive to FPs in the sample/phenotype label), which degrades inferred communities
- When data have strong clustering structure, where ELVAR's improvement over Louvain is less noticeable and extra runtime is not justified

## Inputs

- Undirected cell-cell nearest-neighbor graph (igraph) built from Seurat FindNeighbors, with a vertex attribute (e.g. age, Covid-19 severity) assigned via vertex_attr
- Seurat object with variable features (vst), scaled data, PCA and kNN graph (dims=1:8, k.param=20)
- Number-of-clusters, purity, and modularity matrices from Eva_partitions runs (nC.m, pur.m, mod.m) and the alpha=0 Louvain cluster count (nc0)

## Outputs

- Optimal alpha value and SelOptAlpha selection object (selAlpha.o)
- Per-alpha summaries of cluster counts, purity, and modularity over 100 runs
- Chosen EVA partition object (eva.o) at the optimal alpha for downstream ProcessEVA/DoDA

## How to apply

After building the Seurat cell-cell kNN graph (FindNeighbors with dims=1:8, k.param=20) and assigning the vertex attribute, run Eva_partitions 100 times for each alpha in seq(0.1, 0.9, 0.1) (resolution=1, threshold=1e-07) with the attribute name (e.g. 'age' or Covid-19 severity), plus a benchmark alpha=0 standard Louvain run. Collect the number-of-clusters matrix (nC.m), purity matrix (pur.m), and modularity matrix (mod.m) over the 900 runs, then call SelOptAlpha over alpha.v=seq(0.1,0.9,0.1) to pick the optimal alpha. The decision criteria require the optimal alpha to have purity within the 75th centile of maximum purity, modularity within the 75th centile of maximum modularity, and a cluster count that deviates from the standard Louvain (alpha=0) cluster count — ensuring attribute-aware communities differ meaningfully from purely topology-driven ones. Because 900 runs are computationally heavy, parallelize (e.g. 100 runs per alpha across nodes); on a 100-node server 9 instances suffice. The selected alpha is then used for ProcessEVA community enrichment and DoDA negative-binomial DA testing.

## Related tools

- **ELVAR** (R package providing Eva_partitions and SelOptAlpha for alpha-parameter sweep and optimal-alpha selection) — https://github.com/aet21/ELVAR
- **Seurat** (Builds the Seurat object, runs FindVariableFeatures/ScaleData/RunPCA/FindNeighbors to produce the kNN graph consumed by EVA)
- **igraph** (Constructs the undirected graph (graph.adjacency) and stores vertex attributes used by Eva_partitions)

## Examples

```
eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha=0.8, Vattr.name="age"); selAlpha.o <- SelOptAlpha(nC.m, nc0, pur.m, mod.m, alpha.v=seq(0.1,0.9,0.1));
```

## Evaluation signals

- Selected alpha yields purity within the 75th centile of maximum purity across the alpha sweep
- Selected alpha yields modularity within the 75th centile of maximum modularity
- Cluster count at the optimal alpha deviates from the alpha=0 standard Louvain cluster count (nc0)
- Stability across the 100 independent runs per alpha (consistency of cluster counts, purity, modularity)
- Graph connectivity verified before running Eva_partitions

## Limitations

- Computational burden: identifying an optimal alpha may require 100 runs for each of nine alpha values (900 runs total), though parallelization over a 100-node server reduces this to 9 instances
- ELVAR is more sensitive to false positives in the clustering attribute (sample/phenotype label) than in cell-type annotation; FPs in the attribute affect community quality
- Improvement over the Louvain-analog is more pronounced in studies with weaker clustering structure and less noticeable in strongly clustered data

## Evidence

- [methods] Eva_partitions call: "eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");"
- [methods] SelOptAlpha call: "selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1));"
- [discussion] computational burden of alpha sweep: "to identify an optimal a parameter value may require a total of 100 runs for each of nine a parameter values, so a total of 900 runs, which on a 100-node server would require 9 instances"
- [intro] EVA as attribute-aware Louvain generalization: "we here adapt a node-attribute aware community detection algorithm called EVA17, which is a generalization of the very popular Louvain clustering method18"
- [readme] ELVAR installation and purpose: "It implements an Extended Louvain clustering Algorithm (EVA) that takes cell attribute information into acccount when inferring cellular communities from the cell-cell nearest neighbour graph."
