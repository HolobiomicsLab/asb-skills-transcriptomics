---
name: gene-expression-network-structure-learning
description: Use when you have a directed reference graph (e.g., the 32-node/47-edge
  ALS KEGG graph or KEGG/Reactome interactomes) plus an RNA-seq expression matrix,
  and need to recover or convert it into a valid directed spanning tree (arborescence).
when_to_use_negative:
- When the hypothesis requires latent-variable confounding adjustment or bow-free
  covariances rather than a pure tree structure (use SEMbap-style BAP deconfounding
  / hidden LV models instead).
- When the input graph is undirected and you do not need to orient it into an arborescence
  (tree conversion via Chu–Liu–Edmonds is only meaningful for directed recovery).
- When the expression data cannot meet the modeling assumptions even after the nonparanormal
  transform (e.g., extreme non-normality not relaxed by transformData).
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SEMtree()
  role: Tree-based structure discovery algorithms combining graph and statistically
    interpretable parameters; the SEMtree() function wraps the CAT/CLE tree recovery
    over SEM-scored graphs
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgraph
  role: R package providing SEMrun, transformData (nonparanormal), and igraph/lavaan
    model representation used to score and fit the tree-structured SEM
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMdata
  role: Companion data package supplying the ALS/COVID-19 RNA-seq datasets and KEGG
    reference networks (alsData$graph, alsData$exprs) used as skill inputs
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: mgcv
  role: 'CAT phase one: fits bivariate additive models y_j ~ s(y_i) with penalized
    regression splines (mgcv::gam) from which edge weights are derived'
- name: RBGL
  role: 'CAT phase two: runs RBGL::edmondsOptimumBranching (Chu–Liu–Edmonds) on the
    weighted digraph to extract the minimum-weight spanning arborescence'
- name: igraph
  role: Interchangeable model representation holding the input digraph and the recovered
    arborescence
  repo: https://igraph.org/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:60192c0fb7fab473d24defe070e0587f5f64d8d4524dc03afdbf7673dfd2ea05
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
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# gene-expression-network-structure-learning

## Summary

Learn a directed tree (arborescence) structure over gene-expression data using the SEMtree()/SEMgraph tree-based structure discovery framework, combining SEM-interpretable graph and statistical parameters with Chu–Liu–Edmonds' causal additive tree (CAT) algorithm to convert scored digraphs or undirected active subnetworks into directed trees.

## When to use

Use when you have a directed reference graph (e.g., the 32-node/47-edge ALS KEGG graph or KEGG/Reactome interactomes) plus an RNA-seq expression matrix, and need to recover or convert it into a valid directed spanning tree (arborescence).

## When NOT to use

- When the hypothesis requires latent-variable confounding adjustment or bow-free covariances rather than a pure tree structure (use SEMbap-style BAP deconfounding / hidden LV models instead).
- When the input graph is undirected and you do not need to orient it into an arborescence (tree conversion via Chu–Liu–Edmonds is only meaningful for directed recovery).
- When the expression data cannot meet the modeling assumptions even after the nonparanormal transform (e.g., extreme non-normality not relaxed by transformData).

## Inputs

- Directed reference graph as an igraph object (e.g., alsData$graph: 32 nodes, 47 directed edges; kegg: 5007 nodes, 44755 edges)
- Raw gene-expression count matrix (e.g., alsData$exprs, 160 x 17695 RNA-seq matrix; GEO accessions GSE124439, GSE172114, GSE53740)
- Optional binary group vector of experimental conditions (e.g., alsData$group)
- List of seed (disease) genes or gene P-values for active subnetwork detection

## Outputs

- Directed spanning tree (arborescence) as an igraph object with 31 edges on the 32-node input graph
- Edge list (parent, child, weight) exportable to cat_arborescence.csv
- Tree-structured SEM with statistically interpretable node, directed-edge, and directed-path perturbation parameters (e.g., 16 DRNs and 3 DREs at p<0.05)

## How to apply

Load the reference digraph and expression matrix (e.g., alsData$graph and alsData$exprs, 160 x 17695 RNA-seq) from the SEMdata package, and apply SEMgraph::transformData to the expression matrix to relax the normality assumption via a nonparanormal transform. In CAT phase one, for each directed edge i->j fit the bivariate additive model y_j ~ s(y_i) with mgcv::gam penalized regression splines and derive an edge weight from the fitted smooth (e.g., negated association strength derived from residual variances) so stronger dependence maps to lower cost. Assemble the scored weighted digraph as an igraph object, then in phase two run RBGL::edmondsOptimumBranching (Chu–Liu–Edmonds) to extract the minimum-weight spanning arborescence rooted at a designated node. Alternatively, supply undirected perturbed modules generated from seed genes or gene P-values by five active subnetwork detection methods, and convert them to directed trees via causal additive trees to compare methods in terms of directed active subnetworks. Export the resulting (parent, child, weight) edge list (e.g., to cat_arborescence.csv) and SEM-fitted node/edge perturbation p-values for downstream interpretation.

## Related tools

- **SEMtree()** (Tree-based structure discovery algorithms combining graph and statistically interpretable parameters; the SEMtree() function wraps the CAT/CLE tree recovery over SEM-scored graphs) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgraph** (R package providing SEMrun, transformData (nonparanormal), and igraph/lavaan model representation used to score and fit the tree-structured SEM) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMdata** (Companion data package supplying the ALS/COVID-19 RNA-seq datasets and KEGG reference networks (alsData$graph, alsData$exprs) used as skill inputs) — https://github.com/fernandoPalluzzi/SEMdata
- **mgcv** (CAT phase one: fits bivariate additive models y_j ~ s(y_i) with penalized regression splines (mgcv::gam) from which edge weights are derived)
- **RBGL** (CAT phase two: runs RBGL::edmondsOptimumBranching (Chu–Liu–Edmonds) on the weighted digraph to extract the minimum-weight spanning arborescence)
- **igraph** (Interchangeable model representation holding the input digraph and the recovered arborescence) — https://igraph.org/

## Examples

```
data.npn <- transformData(alsData$exprs)$data; sem0 <- SEMrun(graph = alsData$graph, data = data.npn)
```

## Evaluation signals

- Output must be a valid spanning tree of the 32-node input graph: exactly 31 directed edges, every non-root node with exactly one parent.
- The mgcv::gam bivariate additive fits (y_j ~ s(y_i)) must run to completion without error for every directed edge of the input graph.
- Edge weights must map stronger dependence to lower cost (e.g., negated association strength) before Chu–Liu–Edmonds is applied.
- RBGL::edmondsOptimumBranching must return a minimum-weight arborescence rooted at the designated node, with a (parent, child, weight) edge list of 31 rows.
- Downstream perturbation testing should yield in-range results comparable to the article (e.g., 16 DRNs and 3 DREs at p<0.05 on the ALS data).

## Limitations

- The normality assumption must be relaxed with a nonparanormal transform (transformData) before SEM fitting on RNA-seq data; skipping it risks mis-specified models.
- Tree recovery via Chu–Liu–Edmonds requires a designated root node and cannot represent bidirected/bow-free covariance edges or latent confounders.
- Edge weighting derives from residual variances of bivariate additive fits, a heuristic association strength rather than a full multivariate SEM fit per edge.
- The article's discussion section text is absent from the source document, so long-term caveats and author-stated limitations could not be verified.

## Evidence

- [abstract] tree-based structure discovery algorithms combining graph and statistically interpretable parameters: "We propose SEMtree(), a set of tree-based structure discovery algorithms, combining graph and statistically interpretable parameters together with a user-friendly R package"
- [abstract] CAT conversion of active subnetworks to directed trees: "The latter are supplied to causal additive trees based on Chu–Liu–Edmonds’ algorithm (Chow and Liu, Approximating discrete probability distributions with dependence trees."
- [abstract] active subnetwork inputs from seed genes or P-values: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [methods] nonparanormal transform on RNA-seq data: "we apply a nonparanormal transform to relax the normality assumption."
- [readme] arborescence recovery from correlation matrices in SEMgraph: [evidence span withheld — anchor status: external_doc]
- [results] SEMtree captures biologically relevant subnetworks: "Compared to existing methods, SEMtree() is able to capture biologically relevant subnetworks with simple visualization of directed paths, good perturbation extraction, and classifier performance."
