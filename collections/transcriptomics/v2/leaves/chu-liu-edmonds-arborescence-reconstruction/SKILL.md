---
name: chu-liu-edmonds-arborescence-reconstruction
description: Use when a fully connected directed graph over measured variables must
  be collapsed into a single rooted directed tree, and data-driven edge weights (e.g.,
  from bivariate SEM or spline fits) are available or can be estimated per edge.
when_to_use_negative:
- The input is already a validated DAG from structural learning and no tree-structured
  summary is needed — CAT collapses a fully connected scored digraph to a tree and
  discards polytree structure
- The goal is SEM model improvement with bow-free covariances or latent-variable confounding
  adjustment (use SEMbap-style steps instead)
- Edge weights cannot be estimated per directed edge because data are missing or the
  graph is undirected-only with no bivariate fit possible
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: SEMtree()
  role: Tree-based structure discovery algorithm whose CAT variant performs the two-phase
    weighting + Chu–Liu–Edmonds arborescence recovery
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgraph
  role: R package hosting SEMtree() and transformData (nonparanormal transform) used
    to prepare data before tree learning
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Companion package supplying the ALS dataset (alsData$graph, alsData$exprs)
    used as training input
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: mgcv
  role: Fits the bivariate additive model y_j ~ s(y_i) with penalized regression splines
    to derive per-edge weights in phase one
- name: RBGL
  role: Provides edmondsOptimumBranching, the Chu–Liu–Edmonds implementation that
    recovers the minimum-weight arborescence in phase two
- name: igraph
  role: Interchangeable graph representation for the scored weighted digraph and the
    exported arborescence edge list
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:024f460a984a400f12193c2e7abd661ebde63a02060f2e34b4469425e6dded6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - RBGL
  - SEMdata
  - SEMgraph
  - SEMtree()
  - igraph
  - mgcv
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# chu-liu-edmonds-arborescence-reconstruction

## Summary

Reconstruct a directed spanning tree (arborescence) from a weighted digraph by first scoring every directed edge with weights derived from bivariate additive regression fits (penalized regression splines via mgcv::gam, residual-variance-based weights so stronger dependence maps to lower cost), then applying the Chu–Liu–Edmonds algorithm (RBGL::edmondsOptimumBranching) to extract the minimum-weight spanning arborescence, as used in the CAT (causal additive tree) phase of SEMtree().

## When to use

Use when a fully connected directed graph over measured variables must be collapsed into a single rooted directed tree, and data-driven edge weights (e.g., from bivariate SEM or spline fits) are available or can be estimated per edge.

## When NOT to use

- The input is already a validated DAG from structural learning and no tree-structured summary is needed — CAT collapses a fully connected scored digraph to a tree and discards polytree structure
- The goal is SEM model improvement with bow-free covariances or latent-variable confounding adjustment (use SEMbap-style steps instead)
- Edge weights cannot be estimated per directed edge because data are missing or the graph is undirected-only with no bivariate fit possible

## Inputs

- Directed igraph object encoding hypothesized or data-driven causal relations (e.g., alsData$graph, 32 nodes / 47 directed edges)
- Expression or quantitative data matrix with n samples x p variables (e.g., alsData$exprs, 160 x 17695 RNA-seq matrix)
- Optional binary group vector of experimental conditions (e.g., 139 ALS cases vs 21 healthy controls)
- A designated root node for the arborescence

## Outputs

- Weighted directed igraph with one cost per edge (from the gam/spline weighting phase)
- Minimum-weight spanning arborescence: edge list (parent, child, weight) exported e.g. to cat_arborescence.csv
- Valid tree structure with exactly V-1 directed edges, one parent per non-root node

## How to apply

Operate in two phases as described for CAT within SEMtree(): (1) estimate mutual edge weights of the fully connected directed graph from the residual variances of bivariate additive regression fits — in the R implementation each directed edge i->j is scored by fitting y_j ~ s(y_i) with mgcv::gam using penalized regression splines, and the fitted smooth is converted to a cost (e.g., negated association strength) so stronger dependence yields lower weight; (2) feed the scored weighted digraph assembled as an igraph object into the Chu–Liu–Edmonds algorithm via RBGL::edmondsOptimumBranching to recover the directed tree of minimal total edge weight rooted at a designated node. Before fitting, relax the normality assumption by applying a nonparanormal transform with SEMgraph::transformData (e.g., on the 160 x 17695 ALS RNA-seq matrix alsData$exprs from GSE124439, with alsData$graph as the 32-node/47-edge reference digraph). Export the resulting arborescence as a (parent, child, weight) edge list for downstream SEM fitting or pathway comparison.

## Related tools

- **SEMtree()** (Tree-based structure discovery algorithm whose CAT variant performs the two-phase weighting + Chu–Liu–Edmonds arborescence recovery) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgraph** (R package hosting SEMtree() and transformData (nonparanormal transform) used to prepare data before tree learning) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Companion package supplying the ALS dataset (alsData$graph, alsData$exprs) used as training input) — https://github.com/fernandoPalluzzi/SEMdata
- **mgcv** (Fits the bivariate additive model y_j ~ s(y_i) with penalized regression splines to derive per-edge weights in phase one)
- **RBGL** (Provides edmondsOptimumBranching, the Chu–Liu–Edmonds implementation that recovers the minimum-weight arborescence in phase two)
- **igraph** (Interchangeable graph representation for the scored weighted digraph and the exported arborescence edge list)

## Examples

```
library(SEMgraph); data.npn <- transformData(alsData$exprs)$data
```

## Evaluation signals

- Output must be a valid spanning tree of the input graph: for the 32-node ALS graph, exactly 31 directed edges
- Every non-root node has exactly one parent (arborescence property)
- Every recovered edge is a subset of the scored digraph; stronger bivariate dependence must correspond to lower edge weight (negated association strength)
- The R script runs to completion without error through transformData, mgcv::gam weighting, and RBGL::edmondsOptimumBranching
- The exported edge list (parent, child, weight) round-trips as an igraph object

## Limitations

- CAT recovers a tree (arborescence), not a general DAG, so polytree or bidirected structure is lost during the Chu–Liu–Edmonds projection
- Edge weights come from bivariate fits only, so confounding between variables is not adjusted in the weighting phase
- Requires a designated root node; the CLE solution depends on that choice
- The discussion section text is absent from the source document, limiting caveats available from the article itself

## Evidence

- [abstract] The latter are supplied to causal additive trees based on Chu–Liu–Edmonds’ algorithm: [evidence span withheld — anchor status: not_found]
- [methods] we apply a nonparanormal transform to relax the normality assumption.: "we apply a nonparanormal transform to relax the normality assumption."
- [readme] tree (arborescence) from correlation matrices.: [evidence span withheld — anchor status: external_doc]
- [methods] data.npn <- transformData(alsData$exprs)$data: "data.npn <- transformData(alsData$exprs)$data"
- [abstract] SEMtree() is able to capture biologically relevant subnetworks with simple visualization of directed paths: "SEMtree() is able to capture biologically relevant subnetworks with simple visualization of directed paths, good perturbation extraction, and classifier performance."
