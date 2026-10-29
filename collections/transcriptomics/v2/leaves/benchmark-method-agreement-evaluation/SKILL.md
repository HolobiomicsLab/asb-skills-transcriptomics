---
name: benchmark-method-agreement-evaluation
description: Use when multiple active subnetwork detection methods have each recovered
  a subnetwork from the same seed (disease) genes or gene P-values and you must quantify
  method-to-method structural overlap via pairwise Jaccard similarity.
when_to_use_negative:
- You need edge- or topology-level agreement (e.g., graph edit distance) rather than
  node-set overlap — this skill compares only vertex sets
- Only one subnetwork has been recovered, so there is no pair against which to compute
  Jaccard similarity
- The subnetworks come from heterogeneous benchmark designs (different seeds or reference
  networks per method), which would confound the overlap interpretation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SEMgraph
  role: Provides the graph-object framework (SEMtree()-based tree structure learning)
    in which the recovered subnetworks are held and harmonized for comparison
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: igraph
  role: Graph representation from which the vertex (node) set of each subnetwork is
    extracted for Jaccard computation
- name: R
  role: Environment in which the SEMtree()/SEMgraph pipeline and the pairwise Jaccard
    computation are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:dd6759f2d4f5958de3d7fb7e18c9fec60b186128d183f4b9adaea3b7e4ee1926
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMgraph
  - SEMtree()
  - igraph
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# benchmark-method-agreement-evaluation

## Summary

Quantifies agreement among active subnetwork detection methods (pathfindR, BioNet, STr2z, etc.) by computing pairwise Jaccard similarity on the node sets of their recovered benchmark subnetworks, after harmonizing all outputs as directed trees.

## When to use

Use when multiple active subnetwork detection methods have each recovered a subnetwork from the same seed (disease) genes or gene P-values and you must quantify method-to-method structural overlap via pairwise Jaccard similarity.

## When NOT to use

- You need edge- or topology-level agreement (e.g., graph edit distance) rather than node-set overlap — this skill compares only vertex sets
- Only one subnetwork has been recovered, so there is no pair against which to compute Jaccard similarity
- The subnetworks come from heterogeneous benchmark designs (different seeds or reference networks per method), which would confound the overlap interpretation

## Inputs

- Seven recovered benchmark subnetwork edge lists (one per active subnetwork detection method: pathfindR, BioNet, STr2z, plus four others)
- Lists of seed (disease) genes or gene P-values used to generate each subnetwork
- Subnetwork graphs represented as SEMgraph/igraph graph objects

## Outputs

- jaccard_similarity_matrix.csv — symmetric 7x7 Jaccard similarity matrix with unit diagonal
- 21 pairwise Jaccard index values (|intersection|/|union| of node sets) for all method pairs
- Method-agreement summary distinguishing overlapping (pathfindR–BioNet, pathfindR–STr2z ≈ 0.3) versus distinct recovered network structures, as in Supplementary Table S2

## How to apply

Load the seven recovered benchmark subnetwork edge lists, each generated from a list of seed (disease) genes or gene P-values by one of the seven methods (pathfindR, BioNet, STr2z, and the remaining four). Because five of the methods produce perturbed modules with undirected edges, convert these undirected active subnetworks into directed trees using causal additive trees based on Chu–Liu–Edmonds' algorithm so every method is compared on the same directed active subnetwork representation. Extract each subnetwork's vertex (node) set from the SEMgraph/igraph graph objects, then compute the Jaccard index |intersection|/|union| for all 21 method pairs. Assemble the 21 values into a symmetric 7x7 Jaccard similarity matrix with unit diagonal and write it to jaccard_similarity_matrix.csv; validate against Supplementary Table S2 (with ST and WGM_FC excluded), where pathfindR–BioNet and pathfindR–STr2z show coefficients around 0.3 while the other methods recover distinct structures.

## Related tools

- **SEMgraph** (Provides the graph-object framework (SEMtree()-based tree structure learning) in which the recovered subnetworks are held and harmonized for comparison) — https://github.com/fernandoPalluzzi/SEMgraph
- **igraph** (Graph representation from which the vertex (node) set of each subnetwork is extracted for Jaccard computation)
- **R** (Environment in which the SEMtree()/SEMgraph pipeline and the pairwise Jaccard computation are executed)

## Examples

```
jac <- length(intersect(V(net1)$name, V(net2)$name)) / length(union(V(net1)$name, V(net2)$name))
```

## Evaluation signals

- Output matrix must be exactly 7x7 with diagonal equal to 1 and symmetric off-diagonal entries
- pathfindR–BioNet and pathfindR–STr2z entries must be approximately 0.3, matching Supplementary Table S2 (with ST and WGM_FC excluded)
- All remaining off-diagonal entries must be substantially lower than 0.3, indicating distinct recovered network structures
- Exactly 21 unique pairwise comparisons must be present (7 methods, choose 2)
- Every compared subnetwork must be in directed-tree form (converted via causal additive trees, Chu–Liu–Edmonds) so undirected and directed methods are not mixed

## Limitations

- The reported benchmark table (Supplementary Table S2) excludes the ST (Steiner tree) and WGM_FC methods, so the 7-method matrix cannot be validated against those two entries
- Similarity is assessed on node sets only, not on edge direction or topology, so methods with identical node recruitment but different structure score identically
- Five of the methods generate modules with undirected edges, so the comparison depends on the Chu–Liu–Edmonds causal additive tree conversion, which imposes a directed-tree assumption on undirected modules

## Evidence

- [abstract] from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods.: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [abstract] This conversion allows to compare the methods in terms of directed active subnetworks.: "This conversion allows to compare the methods in terms of directed active subnetworks."
- [methods] Interchangeable model representation as either an **igraph** object or the corresponding SEM in **lavaan** syntax: "Interchangeable model representation as either an **igraph** object or the corresponding SEM in **lavaan** syntax"
- [readme] High-throughput sequencing data, pathways and reference networks to get trained with the **R** package [**SEMgraph**]: [evidence span withheld — anchor status: external_doc]
