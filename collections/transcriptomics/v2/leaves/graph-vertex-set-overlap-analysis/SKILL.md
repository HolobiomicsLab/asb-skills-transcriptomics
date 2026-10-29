---
name: graph-vertex-set-overlap-analysis
description: Use when two or more subnetworks (e.g., edge lists recovered by different
  active subnetwork detection methods such as pathfindR, BioNet, STr2z) must be compared
  on node membership to decide which methods find overlapping vs. distinct network
  structures.
when_to_use_negative:
- Only a single subnetwork is available — pairwise Jaccard requires at least two recovered
  networks to compare
- The comparison target is edge-level or topological similarity (path direction, perturbation
  routes) rather than node membership overlap
- Vertex identifiers are not harmonized across methods (e.g., mixed Entrez IDs and
  symbols) — gene ID conversion (e.g., org.Hs.eg.db) must precede the overlap computation
edam_topics:
- http://edamontology.org/topic_0611
tools:
- name: R
  role: Environment in which the Jaccard computation and matrix assembly are scripted
- name: igraph
  role: Provides the graph objects whose vertex sets V() are extracted for |intersection|/|union|
    computation
- name: SEMgraph
  role: Holds the recovered subnetworks as interchangeable igraph objects used to
    extract node sets for the Jaccard comparison
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMtree()
  role: Implemented in SEMgraph; supplies the tree-based subnetwork recovery context
    in which the recovered benchmark subnetworks are compared
  repo: https://github.com/fernandoPalluzzi/SEMgraph
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:fcf8b0a241ddbbfee4e69b0a3358b58b5b78f3270823aabd566a2b109ffbc171
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMgraph
  tools:
  - R
  - SEMgraph
  - SEMtree()
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0611
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# graph-vertex-set-overlap-analysis

## Summary

Quantifies overlap between subnetworks recovered by different active subnetwork detection methods by computing pairwise Jaccard similarity coefficients on their vertex (node) sets. In the SEMtree benchmark it distinguishes which of seven methods recover overlapping versus distinct COVID-19 subnetwork structures.

## When to use

Use when two or more subnetworks (e.g., edge lists recovered by different active subnetwork detection methods such as pathfindR, BioNet, STr2z) must be compared on node membership to decide which methods find overlapping vs. distinct network structures.

## When NOT to use

- Only a single subnetwork is available — pairwise Jaccard requires at least two recovered networks to compare
- The comparison target is edge-level or topological similarity (path direction, perturbation routes) rather than node membership overlap
- Vertex identifiers are not harmonized across methods (e.g., mixed Entrez IDs and symbols) — gene ID conversion (e.g., org.Hs.eg.db) must precede the overlap computation

## Inputs

- Edge lists of subnetworks recovered from seed (disease) gene lists or gene P-values by active subnetwork detection methods (e.g., pathfindR, BioNet, STr2z, and four others)
- SEMgraph/igraph graph objects representing the recovered subnetworks
- Undirected perturbed modules to be converted to directed trees via Chu–Liu–Edmonds causal additive trees

## Outputs

- Symmetric 7x7 Jaccard similarity matrix with unit diagonal (jaccard_similarity_matrix.csv)
- 21 pairwise Jaccard index values on subnetwork node sets

## How to apply

Load each recovered subnetwork's edge list, ensuring all subnetworks come from the same benchmark input (seed disease genes or gene P-values). If subnetworks are undirected modules, first convert them to directed trees via causal additive trees based on Chu–Liu–Edmonds' algorithm so all methods are comparable as directed active subnetworks. Extract each subnetwork's vertex set from the SEMgraph/igraph graph objects, then compute the Jaccard index |intersection|/|union| for every method pair (21 pairs for 7 methods). Assemble the values into a symmetric similarity matrix with unit diagonal and write it to jaccard_similarity_matrix.csv. In the SEMtree COVID-19 benchmark (Supplementary Table S2, with ST and WGM_FC excluded), pathfindeR–BioNet and pathfindR–STr2z show Jaccard ≈ 0.3 while remaining pairs show low overlap.

## Related tools

- **R** (Environment in which the Jaccard computation and matrix assembly are scripted)
- **igraph** (Provides the graph objects whose vertex sets V() are extracted for |intersection|/|union| computation)
- **SEMgraph** (Holds the recovered subnetworks as interchangeable igraph objects used to extract node sets for the Jaccard comparison) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMtree()** (Implemented in SEMgraph; supplies the tree-based subnetwork recovery context in which the recovered benchmark subnetworks are compared) — https://github.com/fernandoPalluzzi/SEMgraph

## Examples

```
R -e 'J <- length(intersect(V(g1)$name, V(g2)$name))/length(union(V(g1)$name, V(g2)$name))'
```

## Evaluation signals

- Output matrix is exactly 7x7 with unit diagonal and symmetric off-diagonal entries
- pathfindeR–BioNet and pathfindR–STr2z entries are approximately 0.3, matching Supplementary Table S2
- All other off-diagonal entries are substantially lower than 0.3, confirming distinct recovered network structures
- All subnetworks are represented as directed trees (Chu–Liu–Edmonds conversion) before comparison, so pairs are compared on a common directed representation
- Result reproduces Supplementary Table S2 with ST and WGM_FC excluded

## Limitations

- Jaccard on vertex sets ignores edge direction and topology, so networks with identical nodes but different directed paths are scored as maximally similar
- Undirected modules must first be converted to directed trees via causal additive trees (Chu–Liu–Edmonds); skipping this makes method pairs incomparable
- The ≈0.3 finding is tied to this specific COVID-19 benchmark (Supplementary Table S2, with ST and WGM_FC excluded) and may not generalize to other seeds or reference networks

## Evidence

- [results] A Jaccard similarity coefficient around 0.3 is observed between pathfindeR and BioNet, and pathfindeR and STr2z, while the other methods recovered distinct network structures: [evidence span withheld — anchor status: not_found]
- [abstract] Active subnetworks from five methods are converted into directed trees using causal additive trees based on Chu–Liu–Edmonds' algorithm: [evidence span withheld — anchor status: not_found]
- [methods] Interchangeable model representation as either an igraph object or the corresponding SEM in lavaan syntax: "Interchangeable model representation as either an **igraph** object or the corresponding SEM in **lavaan** syntax"
- [readme] Estimate causal relations in network or in complex systems with Structural Equation Modeling (SEM) using as input a directed graph: [evidence span withheld — anchor status: external_doc]
