---
name: directed-tree-conversion-chu-liu-edmonds
description: Use when subnetworks recovered from seed (disease) genes or gene P-values
  by different methods are undirected modules and must be expressed as directed trees
  for a like-for-like comparison of directed active subnetworks (e.g. 21 pairwise
  Jaccard values across 7 methods).
when_to_use_negative:
- The subnetworks are already directed trees/DAGs and a direction assignment step
  would only alter the topology
- The goal is edge-level (not node-level) similarity of already-commensurate network
  structures
- The input is a raw expression matrix or gene list rather than a recovered subnetwork
  — run an active subnetwork detection method first
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0602
tools:
- name: SEMtree()
  role: Provides the tree-based structure discovery algorithms that convert undirected
    perturbed modules into directed trees via causal additive trees (Chu–Liu–Edmonds'
    algorithm)
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgraph
  role: R package implementing model management (graph-to-SEM conversion, graph conversion
    to DAG, tree/arborescence from correlation matrices) in which the converted subnetworks
    are manipulated
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: igraph
  role: Graph representation used to hold the converted directed trees and extract
    vertex sets for Jaccard overlap computation
  repo: https://igraph.org/
- name: R
  role: Environment in which SEMgraph/SEMtree and igraph are run
  repo: https://CRAN.R-project.org/package=SEMgraph
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:14e20b7705568b45763539bd450c6b0bdb3c214430cb832f676ab4c88c28470a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMgraph
  - SEMtree()
  - igraph
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# Directed tree conversion of undirected active subnetworks via causal additive trees (Chu–Liu–Edmonds)

## Summary

Converts undirected perturbed modules produced by active subnetwork detection methods into directed trees using causal additive trees based on Chu–Liu–Edmonds' algorithm, so that heterogeneous methods (pathfindR, BioNet, STr2z, etc.) can be compared on a common directed representation, e.g. via node-set Jaccard similarity.

## When to use

Use when subnetworks recovered from seed (disease) genes or gene P-values by different methods are undirected modules and must be expressed as directed trees for a like-for-like comparison of directed active subnetworks (e.g. 21 pairwise Jaccard values across 7 methods).

## When NOT to use

- The subnetworks are already directed trees/DAGs and a direction assignment step would only alter the topology
- The goal is edge-level (not node-level) similarity of already-commensurate network structures
- The input is a raw expression matrix or gene list rather than a recovered subnetwork — run an active subnetwork detection method first

## Inputs

- Recovered subnetwork edge lists from active subnetwork detection methods (e.g. pathfindR, BioNet, STr2z, plus four others), each derived from seed (disease) genes or gene P-values
- Subnetworks as SEMgraph/igraph graph objects (undirected perturbed modules)
- SEMs and gene data represented in SEMgraph as igraph objects or lavaan syntax

## Outputs

- Directed tree (arborescence) representation of each active subnetwork
- Vertex (node) sets of the converted subnetworks for overlap analysis
- Symmetric 7x7 Jaccard similarity matrix (unit diagonal) written to jaccard_similarity_matrix.csv

## How to apply

Load the recovered subnetwork edge lists (e.g. the seven benchmark subnetworks generated from seed genes or gene P-values by pathfindR, BioNet, STr2z and the remaining four methods) as SEMgraph/igraph graph objects. Apply causal additive trees based on Chu–Liu–Edmonds' algorithm to supply each undirected module with a directed tree (arborescence) structure, exploiting SEMgraph's graph-to-tree model management. Rationale: the five state-of-the-art active subnetwork detection methods generate perturbed modules with undirected edges, so a common directed representation is required before computing node-set overlaps. Extract each converted subnetwork's vertex set with igraph, then compute |intersection|/|union| for every method pair (21 pairs for 7 methods) and assemble the values into a symmetric 7x7 Jaccard similarity matrix with unit diagonal, written to jaccard_similarity_matrix.csv.

## Related tools

- **SEMtree()** (Provides the tree-based structure discovery algorithms that convert undirected perturbed modules into directed trees via causal additive trees (Chu–Liu–Edmonds' algorithm)) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgraph** (R package implementing model management (graph-to-SEM conversion, graph conversion to DAG, tree/arborescence from correlation matrices) in which the converted subnetworks are manipulated) — https://github.com/fernandoPalluzzi/SEMgraph
- **igraph** (Graph representation used to hold the converted directed trees and extract vertex sets for Jaccard overlap computation) — https://igraph.org/
- **R** (Environment in which SEMgraph/SEMtree and igraph are run) — https://CRAN.R-project.org/package=SEMgraph

## Examples

```
devtools::install_github("fernandoPalluzzi/SEMgraph"); # then convert modules: causal additive trees (Chu–Liu–Edmonds) via SEMtree() in SEMgraph
```

## Evaluation signals

- Each converted subnetwork is a directed tree (arborescence) rather than an undirected module, i.e. every node except a root has exactly one parent
- The Jaccard similarity output is a 7x7 symmetric matrix with diagonal equal to 1
- The pathfindR–BioNet and pathfindR–STr2z pairwise entries are approximately 0.3, matching Supplementary Table S2
- All other off-diagonal Jaccard entries are substantially lower than 0.3, indicating distinct recovered network structures
- All 21 method-pair values are finite and in the valid Jaccard range [0, 1]

## Limitations

- The Chu–Liu–Edmonds conversion imposes a directed tree topology that may not reflect the original undirected module edges; similarity conclusions are node-set based
- ST and WGM_FC methods were excluded from Supplementary Table S2, so comparison coverage is not exhaustive across all seven methods
- Discussion section text is absent from the source document, limiting interpretation guidance beyond the reported benchmark
- The benchmark shows most methods recovered distinct network structures, so only specific pairs (pathfindR–BioNet, pathfindR–STr2z) show high overlap (~0.3)

## Evidence

- [abstract] Active subnetworks from five methods are converted into directed trees using causal additive trees based on Chu–Liu–Edmonds' algorithm: [evidence span withheld — anchor status: not_found]
- [abstract] from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [methods] Interchangeable model representation as either an igraph object or the corresponding SEM in lavaan syntax: "Interchangeable model representation as either an **igraph** object or the corresponding SEM in **lavaan** syntax"
- [readme] graph conversion to DAG (Directed Acyclic Graph), and tree (arborescence) from correlation matrices: [evidence span withheld — anchor status: external_doc]
