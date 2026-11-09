---
name: similarity-matrix-construction
description: Use when multiple candidate subnetworks/modules have been recovered from
  the same benchmark data (seed genes or gene P-values) and you need a quantitative,
  pairwise overlap assessment across methods, e.g. Jaccard index on node sets assembled
  into a 7x7 matrix.
when_to_use_negative:
- When only a single recovered network exists and no cross-method comparison is intended
  (Jaccard pairs are undefined for one method)
- When the comparison target is edge- or path-level rather than node-set overlap —
  this skill uses vertex sets only
- When methods produce only directed outputs already and no undirected-to-directed
  conversion via Chu–Liu–Edmonds causal additive trees is needed, the tree-conversion
  step should be skipped
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: igraph
  role: Graph library underlying SEMgraph; used to hold the subnetworks as graph objects
    and extract vertex (node) sets for pairwise Jaccard computation
- name: SEMgraph
  role: R package providing the graph objects in which the seven recovered subnetworks
    are represented before node-set extraction
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: R
  role: Language used to compute the 21 pairwise Jaccard indices and write the 7x7
    matrix to CSV
  repo: https://CRAN.R-project.org/package=SEMgraph
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:d50d95845ba25005606fa907428188b6dd8721ea68d4839f0fee2e9a4cd730dc
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# similarity-matrix-construction

## Summary

Constructs a symmetric pairwise Jaccard similarity matrix comparing node sets of seven active subnetworks (pathfindR, BioNet, STr2z, and four other methods) recovered from the COVID-19 GSE172114 benchmark, to quantify which methods recover overlapping versus distinct network structures.

## When to use

Use when multiple candidate subnetworks/modules have been recovered from the same benchmark data (seed genes or gene P-values) and you need a quantitative, pairwise overlap assessment across methods, e.g. Jaccard index on node sets assembled into a 7x7 matrix.

## When NOT to use

- When only a single recovered network exists and no cross-method comparison is intended (Jaccard pairs are undefined for one method)
- When the comparison target is edge- or path-level rather than node-set overlap — this skill uses vertex sets only
- When methods produce only directed outputs already and no undirected-to-directed conversion via Chu–Liu–Edmonds causal additive trees is needed, the tree-conversion step should be skipped

## Inputs

- Seven recovered benchmark subnetwork edge lists (one per active subnetwork detection method: pathfindR, BioNet, STr2z, and four others)
- Lists of seed (disease) genes or gene P-values used to generate the subnetworks
- SEMgraph/igraph graph objects representing the subnetworks

## Outputs

- Symmetric 7x7 Jaccard similarity matrix (unit diagonal) with 21 pairwise values
- jaccard_similarity_matrix.csv file

## How to apply

Load the seven recovered benchmark subnetwork edge lists, each generated from a list of seed (disease) genes or gene P-values by an active subnetwork detection method (pathfindR, BioNet, STr2z, and the remaining four). Because some methods produce undirected perturbed modules, first convert undirected active subnetworks into directed trees via causal additive trees based on Chu–Liu–Edmonds' algorithm so all methods are compared as directed active subnetworks. Using SEMgraph/igraph graph objects, extract each subnetwork's vertex set, then compute the Jaccard index |intersection|/|union| for all 21 method pairs. Assemble the 21 values into a symmetric 7x7 matrix with unit diagonal and write it to jaccard_similarity_matrix.csv. Validate against the reported structure: pathfindR–BioNet and pathfindR–STr2z pairs should show Jaccard ≈ 0.3 while other off-diagonal pairs remain substantially lower (Supplementary Table S2, with ST and WGM_FC excluded).

## Related tools

- **igraph** (Graph library underlying SEMgraph; used to hold the subnetworks as graph objects and extract vertex (node) sets for pairwise Jaccard computation)
- **SEMgraph** (R package providing the graph objects in which the seven recovered subnetworks are represented before node-set extraction) — https://github.com/fernandoPalluzzi/SEMgraph
- **R** (Language used to compute the 21 pairwise Jaccard indices and write the 7x7 matrix to CSV) — https://CRAN.R-project.org/package=SEMgraph

## Examples

```
jac <- length(intersect(a, b)) / length(union(a, b))  # a, b are node sets extracted via V(graph) from igraph/SEMgraph subnetwork objects
```

## Evaluation signals

- Output matrix must be 7x7 (21 unique pairwise comparisons among 7 methods)
- Diagonal of the Jaccard matrix must equal 1
- pathfindR–BioNet and pathfindR–STr2z entries must be approximately 0.3, matching Supplementary Table S2
- All remaining off-diagonal entries must be substantially lower than 0.3, indicating distinct recovered network structures
- ST and WGM_FC are excluded from the verification against Supplementary Table S2

## Limitations

- Undirected modules must first be converted to directed trees via causal additive trees (Chu–Liu–Edmonds), so the comparison reflects directed active subnetworks rather than the raw undirected modules
- The reported verification excludes the ST and WGM_FC methods (Supplementary Table S2 check is performed without them)
- Discussion section text is absent from the source document, so interpretation of the similarity structure beyond the reported finding is not available

## Evidence

- [abstract] Active subnetworks from five methods are converted into directed trees using causal additive trees based on Chu–Liu–Edmonds' algorithm, allowing comparison in terms of directed active subnetworks: "The latter are supplied to causal additive trees based on Chu–Liu–Edmonds’ algorithm (Chow and Liu, Approximating discrete probability distributions with dependence trees."
- [supplementary] Jaccard similarity ≈ 0.3 for pathfindR–BioNet and pathfindR–STr2z, other methods recover distinct structures: [evidence span withheld — anchor status: not_found]
- [methods] Subnetworks held as igraph objects in SEMgraph, from which vertex sets are extracted: "Interchangeable model representation as either an **igraph** object or the corresponding SEM in **lavaan** syntax"
