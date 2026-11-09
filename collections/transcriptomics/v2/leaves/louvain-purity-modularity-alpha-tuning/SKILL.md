---
name: louvain-purity-modularity-alpha-tuning
description: Use when running cell-attribute aware (EVA) community detection on a
  cell-cell nearest-neighbor graph and an optimal alpha must be chosen before attribute-aware
  clustering and downstream DA testing.
when_to_use_negative:
- When only a standard Louvain partition is needed with no cell attribute to exploit
  — run FindClusters/Louvain (alpha=0) directly instead of the 900-run alpha grid
- When compute resources are limited and the up-to-900-run tuning loop (100 runs x
  9 alphas) is prohibitive, since EVA already carries increased runtime versus Milo
  or the Louvain-analog
- When the clustering attribute (sample/phenotype label) has many suspected false
  positives, as EVA is more sensitive to FPs in the attribute than in cell-type annotation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: ELVAR
  role: R package providing Eva_partitions and SelOptAlpha for attribute-aware community
    detection and alpha tuning
  repo: https://github.com/aet21/ELVAR
- name: Seurat
  role: Builds the cell-cell nearest-neighbor state-manifold graph (FindNeighbors,
    k.param=20, PCA dims) that Eva_partitions consumes
- name: igraph
  role: Holds the undirected adjacency graph and its vertex cell attributes passed
    to Eva_partitions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:b2fd87d60052a2a5ab4fc551456a2ab5d9d4b0aec2d0cba0ef68ac993c4a393b
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# louvain-purity-modularity-alpha-tuning

## Summary

Selecting an optimal EVA/Louvain alpha parameter by running Eva_partitions 100 times per alpha (0.1–0.9) and using SelOptAlpha to pick the alpha where purity and modularity stay within the 75th centile of their maxima while cluster count deviates from Louvain at the 95% quantile.

## When to use

Use when running cell-attribute aware (EVA) community detection on a cell-cell nearest-neighbor graph and an optimal alpha must be chosen before attribute-aware clustering and downstream DA testing.

## When NOT to use

- When only a standard Louvain partition is needed with no cell attribute to exploit — run FindClusters/Louvain (alpha=0) directly instead of the 900-run alpha grid
- When compute resources are limited and the up-to-900-run tuning loop (100 runs x 9 alphas) is prohibitive, since EVA already carries increased runtime versus Milo or the Louvain-analog
- When the clustering attribute (sample/phenotype label) has many suspected false positives, as EVA is more sensitive to FPs in the attribute than in cell-type annotation

## Inputs

- Undirected igraph cell-cell nearest-neighbor graph (from Seurat FindNeighbors with k.param=20 on PCA dims) converted via graph.adjacency
- Vertex cell attributes (e.g. age, disease stage) attached with vertex_attr()
- Louvain (alpha=0) reference cluster count nc0
- Purity (pur.m), modularity (mod.m) and cluster-count (nC.m) matrices accumulated across 100 runs per alpha from Eva_partitions

## Outputs

- Selected optimal alpha value (from SelOptAlpha, e.g. alpha ~0.8 in the tutorial)
- selAlpha.o object from SelOptAlpha containing purity/modularity/cluster-count summaries across the alpha grid
- Final Eva_partitions object (eva.o) computed at the optimal alpha with the attribute-aware Vattr.name set, yielding EVA communities for downstream ProcessEVA/DoDA

## How to apply

Run Eva_partitions 100 times for each alpha from 0.1 to 0.9 on the undirected igraph adjacency graph (resolution=1, threshold=0.0000001), collecting per-run cluster counts (nC.m), purity (pur.m), and modularity (mod.m). Call SelOptAlpha(nC.m, nc0, pur.m, mod.m, alpha.v=seq(0.1,0.9,0.1)), where nc0 is the cluster count from a Louvain (alpha=0) reference run. The rationale is to keep the EVA partition comparable to Louvain in quality while avoiding degenerate splits: accept alphas whose purity and modularity are both within the 75th centile of their maxima and whose cluster count deviates from the Louvain nc0 only at the 95% quantile. After selection, re-run Eva_partitions at the chosen alpha with Vattr.name set to the cell attribute (e.g. disease stage or age) to produce the final attribute-aware communities. Note the cost: this grid implies up to 900 runs, which the authors parallelize (e.g. 9 instances on a 100-node server).

## Related tools

- **ELVAR** (R package providing Eva_partitions and SelOptAlpha for attribute-aware community detection and alpha tuning) — https://github.com/aet21/ELVAR
- **Seurat** (Builds the cell-cell nearest-neighbor state-manifold graph (FindNeighbors, k.param=20, PCA dims) that Eva_partitions consumes)
- **igraph** (Holds the undirected adjacency graph and its vertex cell attributes passed to Eva_partitions)

## Examples

```
eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha=0.8, Vattr.name="age"); selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1));
```

## Evaluation signals

- SelOptAlpha-selected alpha yields purity and modularity both within the 75th centile of their per-alpha maxima
- Cluster count at the chosen alpha deviates from the Louvain (alpha=0) nc0 reference only at the 95% quantile across the 100 runs
- Grid covers nine alpha values 0.1–0.9 with 100 Eva_partitions runs each (900 total runs; resolution=1, threshold=0.0000001)
- Resulting EVA communities, after re-running with Vattr.name set, support stage-enriched clusters detectable by ProcessEVA Binomial tests with Bonferroni threshold
- Downstream DoDA on enriched communities outperforms the Louvain (alpha=0, LVnonseq) benchmark, e.g. stronger Wald/LRT significance in the Becker colon snRNA-Seq stem-cell and T-reg DA shifts

## Limitations

- EVA and ELVAR carry increased computational complexity and runtimes compared to Milo and the Louvain-analog, though not deemed practically prohibitive by the authors
- EVA is more sensitive to false positives in the clustering attribute (sample/phenotype label) than to FPs in cell-type annotation; FPs in the attribute can degrade community quality
- Improvement of ELVAR over its Louvain-analog is more pronounced in datasets with weaker clustering structure and less noticeable when clustering structure is strong

## Evidence

- [discussion] Run EVA 100 times across nine alpha parameter values: "to identify an optimal a parameter value may require a total of 100 runs for each of nine a parameter values, so a total of 900 runs"
- [methods] SelOptAlpha call in methods: "selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1));"
- [methods] Eva_partitions call in methods: "eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");"
- [intro] EVA described as Louvain generalization: "we here adapt a node-attribute aware community detection algorithm called EVA17, which is a generalization of the very popular Louvain clustering method18"
- [readme] ELVAR summary from README: "It implements an Extended Louvain clustering Algorithm (EVA) that takes cell attribute information into acccount when inferring cellular communities"
