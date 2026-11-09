---
name: penalized-spline-additive-model-edge-weighting
description: 'Use when you need data-driven weights for every directed edge of an
  input igraph network from RNA-seq-style expression data, as the first phase of CAT
  tree reconstruction in SEMtree(); trigger: fully connected or candidate digraph
  plus an n x p expression matrix.'
when_to_use_negative:
- When the goal is SEM perturbation testing on an existing graph (SEMrun node/edge
  perturbation or SEMace), not data-driven edge scoring for tree reconstruction
- When edges are undirected and must remain undirected (e.g., active subnetwork modules
  before CAT conversion) — the fits model directed i->j relationships
- When relationships are known to be linear and a simpler linear SEM edge weighting
  (e.g., Fisher r-to-z on correlations) suffices; penalized splines target nonlinear
  additive dependence
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: mgcv
  role: Fits the bivariate additive model y_j ~ s(y_i) per directed edge with penalized
    regression splines to produce edge weights
- name: SEMgraph
  role: Provides transformData for the nonparanormal transform of the expression data
    prior to edge scoring, and hosts SEMtree() CAT functionality
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: igraph
  role: Holds the assembled scored weighted digraph that is passed to the Chu–Liu–Edmonds
    phase
- name: RBGL
  role: Runs edmondsOptimumBranching (Chu–Liu–Edmonds) on the spline-scored weighted
    digraph in the second CAT phase
- name: SEMdata
  role: Supplies the ALS training data (alsData$graph and alsData$exprs RNA-seq matrix)
    used to fit the edge models
  repo: https://github.com/fernandoPalluzzi/SEMdata
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:7d6ca279a1f2a63272201f566013311e475be8e4fe5534f4aa3067b91f893057
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# penalized-spline-additive-model-edge-weighting

## Summary

Scores each directed edge i->j of a graph by fitting a bivariate additive regression y_j ~ s(y_i) with penalized regression splines (mgcv::gam) and converting the fitted smooth's association strength into an edge weight (e.g., negated so stronger dependence = lower cost), feeding the weighted digraph into causal additive tree (CAT) recovery via Chu–Liu–Edmonds in SEMtree().

## When to use

Use when you need data-driven weights for every directed edge of an input igraph network from RNA-seq-style expression data, as the first phase of CAT tree reconstruction in SEMtree(); trigger: fully connected or candidate digraph plus an n x p expression matrix.

## When NOT to use

- When the goal is SEM perturbation testing on an existing graph (SEMrun node/edge perturbation or SEMace), not data-driven edge scoring for tree reconstruction
- When edges are undirected and must remain undirected (e.g., active subnetwork modules before CAT conversion) — the fits model directed i->j relationships
- When relationships are known to be linear and a simpler linear SEM edge weighting (e.g., Fisher r-to-z on correlations) suffices; penalized splines target nonlinear additive dependence

## Inputs

- Directed input graph as an igraph object (e.g., alsData$graph, 32 nodes / 47 edges)
- Expression matrix with n samples x p variables matching graph nodes (e.g., alsData$exprs, 160 x 17695 RNA-seq matrix)
- Optionally, a nonparanormal-transformed data matrix from SEMgraph::transformData

## Outputs

- Weighted directed graph (igraph object) with an additive-model-derived weight per edge
- Edge scores from penalized spline fits usable as CLE costs (e.g., negated association strength)
- Downstream: minimum-weight spanning arborescence from Chu–Liu–Edmonds (e.g., exported as parent/child/weight edge list such as cat_arborescence.csv)

## How to apply

First relax the normality assumption on the expression matrix with a nonparanormal transform (SEMgraph::transformData), as done for the ALS dataset (alsData$exprs, 160 x 17695 RNA-seq matrix over the 32-node/47-edge alsData$graph). For each directed edge i->j in the input graph, fit the bivariate additive model y_j ~ s(y_i) with mgcv::gam using penalized regression splines, and derive an edge weight from the fitted smooth (e.g., negated association strength) so stronger dependence maps to lower cost. Assemble the scored weighted digraph as an igraph object, then pass it to the second CAT phase (Chu–Liu–Edmonds, e.g. RBGL::edmondsOptimumBranching) to extract the minimum-weight spanning arborescence. Decision point: only candidate edges present in the input graph need scoring, keeping the bivariate fits proportional to the graph's edge count rather than the full p^2 node pairs.

## Related tools

- **mgcv** (Fits the bivariate additive model y_j ~ s(y_i) per directed edge with penalized regression splines to produce edge weights)
- **SEMgraph** (Provides transformData for the nonparanormal transform of the expression data prior to edge scoring, and hosts SEMtree() CAT functionality) — https://github.com/fernandoPalluzzi/SEMgraph
- **igraph** (Holds the assembled scored weighted digraph that is passed to the Chu–Liu–Edmonds phase)
- **RBGL** (Runs edmondsOptimumBranching (Chu–Liu–Edmonds) on the spline-scored weighted digraph in the second CAT phase)
- **SEMdata** (Supplies the ALS training data (alsData$graph and alsData$exprs RNA-seq matrix) used to fit the edge models) — https://github.com/fernandoPalluzzi/SEMdata

## Examples

```
library(mgcv); w <- sapply(E(graph), function(e) -summary(gam(y[, e[2]] ~ s(y[, e[1]])))$s.table[,"p-values"])
```

## Evaluation signals

- Each directed edge of the input graph has a finite numeric weight derived from its mgcv::gam fit (no missing scores)
- Downstream validation: the CLE output is a valid spanning tree of the 32-node ALS graph with exactly 31 directed edges and every non-root node having exactly one parent
- Stronger fitted dependence consistently maps to lower edge cost (negation direction check) so the arborescence selects high-association edges
- The pipeline runs to completion without error on the nonparanormal-transformed alsData$exprs matrix

## Limitations

- Weights come from bivariate (edge-wise) fits, so they do not condition on the full multivariate SEM and may reflect indirect dependence
- The input directed graph constrains which edges are scored; edge directions in the reference network bias which causal orderings can be recovered
- Non-Gaussian expression data should be nonparanormal-transformed first; skipping the transform violates the normality assumption the method relaxes

## Evidence

- [abstract] Active subnetworks from five methods are converted into directed trees using causal additive trees based on Chu–Liu–Edmonds' algorithm: [evidence span withheld — anchor status: not_found]
- [methods] Nonparanormal transform prior to edge scoring: "we apply a nonparanormal transform to relax the normality assumption."
- [readme] SEMgraph handles tree (arborescence) construction from data: [evidence span withheld — anchor status: external_doc]
- [readme] ALS dataset used for training the workflow: [evidence span withheld — anchor status: external_doc]
