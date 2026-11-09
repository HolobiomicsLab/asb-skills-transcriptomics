---
name: active-subnetwork-jaccard-similarity-comparison
description: Use when several active subnetworks are recovered from the same seed
  genes or gene P-values (e.g., on COVID-19 benchmark data) and you need to quantify
  node-set overlap between methods via pairwise Jaccard similarity.
when_to_use_negative:
- Only a single subnetwork is available (pairwise Jaccard requires at least two method-recovered
  networks)
- The comparison should reflect edge-level or causal-structure agreement rather than
  node-set overlap — Jaccard here is computed on vertex sets only
- Subnetworks were derived from different seed gene sets or different input data,
  so overlap reflects differing inputs rather than method behavior
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0602
tools:
- name: SEMgraph
  role: R package whose graph objects hold the recovered subnetworks and provide the
    vertex sets for Jaccard computation
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMtree()
  role: Tree-based structure discovery function; supplies causal additive trees used
    to convert undirected active subnetworks into directed trees for comparison
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: igraph
  role: Graph representation used to extract the vertex (node) set V() of each subnetwork
    before computing Jaccard indices
- name: R
  role: Analysis environment in which the pairwise Jaccard matrix is computed and
    written to jaccard_similarity_matrix.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:0b542852f74af118584bb5fd22edbe294de9187279e7c012095f242cf9973b5f
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
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# active-subnetwork-jaccard-similarity-comparison

## Summary

Quantifies how similar active subnetworks recovered by different structure-learning methods are, by converting each undirected perturbed module to a directed tree and computing pairwise Jaccard indices on node sets. In the SEMtree benchmark it distinguishes methods that recover overlapping structures (pathfindR–BioNet and pathfindR–STr2z at Jaccard ≈ 0.3) from those recovering distinct ones.

## When to use

Use when several active subnetworks are recovered from the same seed genes or gene P-values (e.g., on COVID-19 benchmark data) and you need to quantify node-set overlap between methods via pairwise Jaccard similarity.

## When NOT to use

- Only a single subnetwork is available (pairwise Jaccard requires at least two method-recovered networks)
- The comparison should reflect edge-level or causal-structure agreement rather than node-set overlap — Jaccard here is computed on vertex sets only
- Subnetworks were derived from different seed gene sets or different input data, so overlap reflects differing inputs rather than method behavior

## Inputs

- Seven recovered benchmark subnetwork edge lists, each generated from a list of seed (disease) genes or gene P-values by an active subnetwork detection method (e.g., pathfindR, BioNet, STr2z)
- igraph/SEMgraph graph objects representing each subnetwork (directed after causal additive tree conversion)
- Seed (disease) gene lists or gene P-value lists used for subnetwork recovery (e.g., from COVID-19 RNA-seq data, GSE172114)

## Outputs

- Symmetric 7x7 Jaccard similarity matrix with unit diagonal (jaccard_similarity_matrix.csv)
- 21 pairwise Jaccard index values (|intersection|/|union| of node sets) for method-pair comparison
- Interpretation of overlapping vs. distinct recovered network structures per method pair

## How to apply

Load the recovered subnetwork edge lists, each generated from a list of seed (disease) genes or gene P-values by a different active subnetwork detection method (pathfindR, BioNet, STr2z, and the remaining methods). Where subnetworks are undirected modules, convert them to directed trees using causal additive trees based on Chu–Liu–Edmonds' algorithm, so all methods can be compared as directed active subnetworks. Extract the vertex (node) set of each subnetwork from the SEMgraph/igraph graph objects. Compute the Jaccard index |intersection|/|union| for all 21 method pairs on their node sets and assemble the values into a symmetric 7x7 similarity matrix with unit diagonal (here excluding ST and WGM_FC), written to jaccard_similarity_matrix.csv. Validate against Supplementary Table S2: the pathfindR–BioNet and pathfindR–STr2z pairs should show Jaccard ≈ 0.3 while all other off-diagonal pairs are substantially lower.

## Related tools

- **SEMgraph** (R package whose graph objects hold the recovered subnetworks and provide the vertex sets for Jaccard computation) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMtree()** (Tree-based structure discovery function; supplies causal additive trees used to convert undirected active subnetworks into directed trees for comparison) — https://github.com/fernandoPalluzzi/SEMgraph
- **igraph** (Graph representation used to extract the vertex (node) set V() of each subnetwork before computing Jaccard indices)
- **R** (Analysis environment in which the pairwise Jaccard matrix is computed and written to jaccard_similarity_matrix.csv)

## Examples

```
V1 <- V(g1)$name; V2 <- V(g2)$name; jaccard <- length(intersect(V1, V2)) / length(union(V1, V2))
```

## Evaluation signals

- Output matrix is exactly 7x7 with all diagonal entries equal to 1
- Matrix is symmetric (J[i,j] == J[j,i]) with 21 unique pairwise values covering all method pairs
- pathfindR–BioNet and pathfindR–STr2z entries are approximately 0.3, matching Supplementary Table S2
- All remaining off-diagonal entries are substantially lower than 0.3, indicating distinct recovered network structures
- Every subnetwork was converted to a directed representation (via Chu–Liu–Edmonds causal additive trees) before node-set extraction, so comparisons are between directed active subnetworks

## Limitations

- Jaccard is computed on node sets only, so agreement in edges or directionality is not captured
- Two methods (ST and WGM_FC) were excluded from the 7x7 comparison reported in Supplementary Table S2
- Results are benchmark-specific: the ≈0.3 overlap values were observed on subnetworks recovered from COVID-19 benchmark data (GSE172114) and may differ for other datasets or seed gene lists
- Comparability depends on first converting undirected modules to directed trees; skipping this conversion makes node-set comparison between methods inconsistent

## Evidence

- [abstract] Active subnetworks from five methods are converted into directed trees using causal additive trees based on Chu–Liu–Edmonds' algorithm, allowing comparison in terms of directed active subnetworks: "The latter are supplied to causal additive trees based on Chu–Liu–Edmonds’ algorithm (Chow and Liu, Approximating discrete probability distributions with dependence trees."
- [abstract] Undirected perturbed modules are generated from seed genes or gene P-values by state-of-the-art active subnetwork detection methods: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [supplementary] Jaccard ≈ 0.3 between pathfindR and BioNet, and pathfindR and STr2z, while other methods recover distinct structures (task_007 finding, Supplementary Table S2): [evidence span withheld — anchor status: not_found]
- [methods] igraph objects serve as the interchangeable graph representation from which vertex sets are extracted: "Interchangeable model representation as either an **igraph** object or the corresponding SEM in **lavaan** syntax"
