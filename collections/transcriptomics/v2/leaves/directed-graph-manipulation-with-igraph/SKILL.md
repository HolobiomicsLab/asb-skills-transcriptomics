---
name: directed-graph-manipulation-with-igraph
description: Use when the analysis input is a directed reference network (igraph object,
  e.g. alsData$graph or kegg) plus expression data, and the goal is to weight, filter,
  or convert that digraph into a DAG or directed tree (arborescence) for SEM fitting
  or Chu–Liu–Edmonds recovery.
when_to_use_negative:
- The input graph is already a DAG or arborescence and only needs SEM fitting (SEMrun),
  not graph restructuring or weighting.
- The network is inherently undirected (e.g. the STRING interactome, 10829 nodes /
  213445 undirected edges) and no causal direction information is available to weight
  digraph edges.
- The goal is differential-expression testing rather than structure recovery — node/edge
  perturbation statistics from SEMrun (16 DRNs, 3 DREs at p<0.05 in the ALS model)
  do not require rebuilding the digraph.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3366
tools:
- name: igraph
  role: 'Primary directed-graph data structure: interchangeably convertible with lavaan
    SEM syntax; holds the weighted digraph assembled for Chu–Liu–Edmonds tree extraction'
  repo: https://igraph.org/
- name: SEMgraph
  role: Provides transformData (nonparanormal), SEMrun, SEMdag and SEMbap operating
    on the igraph digraph; hosts the SEMtree tree-structure learning methods
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: RBGL
  role: edmondsOptimumBranching implements Chu–Liu–Edmonds on the weighted digraph
    to recover the minimum spanning arborescence
- name: mgcv
  role: gam penalized regression splines (y_j ~ s(y_i)) on bivariate fits supply the
    edge weights of the fully connected digraph
- name: SEMdata
  role: Supplies the alsData graph/exprs and KEGG reference networks used as igraph
    digraph inputs
  repo: https://github.com/fernandoPalluzzi/SEMdata
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:269d490d9feaf366b9ee70c80c7cc5afcbbd9ab4276c161a2d1dffcbdeb3cd1e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RBGL
  - SEMdata
  - SEMgraph
  - SEMtree()
  - igraph
  - mgcv
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# directed-graph-manipulation-with-igraph

## Summary

Build, inspect, weight, and restructure directed graphs (igraph objects) as the backbone of SEMtree()-based causal network analysis, e.g. turning the ALS 32-node/47-edge alsData$graph or the 5007-node/44755-edge KEGG interactome into scored digraphs from which an arborescence can be extracted.

## When to use

Use when the analysis input is a directed reference network (igraph object, e.g. alsData$graph or kegg) plus expression data, and the goal is to weight, filter, or convert that digraph into a DAG or directed tree (arborescence) for SEM fitting or Chu–Liu–Edmonds recovery.

## When NOT to use

- The input graph is already a DAG or arborescence and only needs SEM fitting (SEMrun), not graph restructuring or weighting.
- The network is inherently undirected (e.g. the STRING interactome, 10829 nodes / 213445 undirected edges) and no causal direction information is available to weight digraph edges.
- The goal is differential-expression testing rather than structure recovery — node/edge perturbation statistics from SEMrun (16 DRNs, 3 DREs at p<0.05 in the ALS model) do not require rebuilding the digraph.

## Inputs

- Directed reference network as an igraph object (e.g. alsData$graph: 32 nodes / 47 edges; kegg: 5007 nodes / 44755 edges)
- Gene expression matrix (e.g. alsData$exprs: 160 x 17695 RNA-seq matrix from GSE124439)
- Optional binary group vector (e.g. alsData$group) for perturbation scoring
- Edge weight scores derived from bivariate additive model fits (mgcv::gam residuals/association strength)

## Outputs

- Weighted directed igraph object ready for arborescence extraction
- Minimum-weight spanning arborescence (directed tree) as parent–child–weight edge list (e.g. cat_arborescence.csv)
- igraph-to-lavaan model representation for SEMrun fitting

## How to apply

Represent the model as an igraph object so it can interoperate with lavaan SEM syntax inside SEMgraph. Before weighting, relax normality on the expression matrix with the nonparanormal transform (SEMgraph::transformData) so edge scores derived from bivariate fits (e.g. mgcv::gam penalized regression splines y_j ~ s(y_i)) are well behaved; map stronger dependence to lower cost (negated association strength). Assemble the scored weighted digraph with directed edges only (KEGG reference: 41122 directed plus 3164 bidirected edges, the latter halved for handling). Decision point: for tree recovery, feed the weighted digraph to Chu–Liu–Edmonds (RBGL::edmondsOptimumBranching) to extract the minimum-weight spanning arborescence rooted at a designated node; for DAG estimation instead use SEMdag node/layer ordering, and for deconfounding use SEMbap bow-free covariance search. Export parent–child–weight edge lists (e.g. cat_arborescence.csv) for downstream SEMrun perturbation testing.

## Related tools

- **igraph** (Primary directed-graph data structure: interchangeably convertible with lavaan SEM syntax; holds the weighted digraph assembled for Chu–Liu–Edmonds tree extraction) — https://igraph.org/
- **SEMgraph** (Provides transformData (nonparanormal), SEMrun, SEMdag and SEMbap operating on the igraph digraph; hosts the SEMtree tree-structure learning methods) — https://github.com/fernandoPalluzzi/SEMgraph
- **RBGL** (edmondsOptimumBranching implements Chu–Liu–Edmonds on the weighted digraph to recover the minimum spanning arborescence)
- **mgcv** (gam penalized regression splines (y_j ~ s(y_i)) on bivariate fits supply the edge weights of the fully connected digraph)
- **SEMdata** (Supplies the alsData graph/exprs and KEGG reference networks used as igraph digraph inputs) — https://github.com/fernandoPalluzzi/SEMdata

## Examples

```
summary(alsData$graph); data.npn <- transformData(alsData$exprs)$data
```

## Evaluation signals

- Output arborescence is a valid spanning tree of the 32-node input graph: exactly 31 directed edges, every non-root node has exactly one parent, no cycles.
- The weighted digraph contains only directed edges; bidirected edges are split/halved (KEGG convention: 3164/2 = 1582 bidirected pairs) before scoring.
- igraph and lavaan representations of the same model agree after SEMgraph graph-to-SEM conversion.
- Edge weights are monotonically ordered with dependence strength: stronger bivariate association (gam smooth) maps to lower (more negative) cost.
- Script runs to completion without error on the transformed (nonparanormal) data matrix.

## Limitations

- Bidirected edges (e.g. the 3164 in the KEGG reference network) are not directly representable in a pure digraph and must be halved or handled separately.
- Very large reference networks (Reactome: 9762 nodes / 416128 edges) require SEMgraph's heuristic filtering, node/edge weighting, resampling and parallelization settings for tractable fitting.
- The nonparanormal transform relaxes but does not guarantee normality, so residual-variance-based edge weights from bivariate gam fits can still be unstable for weakly connected nodes.

## Evidence

- [methods] Interchangeable model representation as either an igraph object or the corresponding SEM in lavaan syntax: "Interchangeable model representation as either an **igraph** object or the corresponding SEM in **lavaan** syntax"
- [methods] Nonparanormal transform on ALS expression data: "we apply a nonparanormal transform to relax the normality assumption."
- [readme] Directed graph as SEMgraph input: [evidence span withheld — anchor status: external_doc]
- [readme] KEGG reference network as igraph object: [evidence span withheld — anchor status: external_doc]
- [abstract] Chu–Liu–Edmonds conversion of active subnetworks to directed trees: [evidence span withheld — anchor status: not_found]
