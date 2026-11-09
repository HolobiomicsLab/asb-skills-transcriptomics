---
name: single-cell-lineage-cluster-assignment
description: Use when you have a single-cell phylogeny (Newick/phylo object with cells
  as leaves) and need to group cells into a target number of clade-coherent subpopulations,
  e.g. to define tumor subclones or lineage labels for signature-scoring.
when_to_use_negative:
- The input is a flat expression matrix or annotation table with no phylogenetic tree
  — clustering must instead use expression-based methods (e.g. UMAP/Hotspot modules)
- Clusters need not correspond to connected clades (e.g. purely transcriptional communities)
  — cophenetic merging will force lineage-coherent groups
- The tree has unresolved unifurcations and you cannot preprocess it — the algorithm
  assumes every internal node branches properly after collapse.singles
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: PhyloVision
  role: Implements the breadth-first Cluster-Phylogeny cell-clustering over internal
    nodes and the VISION_Clusters_Tree metadata output
  repo: https://github.com/YosefLab/VISION
- name: ape
  role: R package whose collapse.singles function removes unifurcations from the phylo
    tree before clustering
- name: R
  role: Language in which PhyloVision and the clustering implementation are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:53833c44a54d02e435135183f222dad7d369a720d91456575564e2d3b5a3b764
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - PhyloVision
  - R
  - ape
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# single-cell-lineage-cluster-assignment

## Summary

Partition the tip cells of a reconstructed single-cell phylogeny into a fixed number of phylogenetically coherent subpopulation clusters using PhyloVision's Cluster-Phylogeny algorithm, a breadth-first search over internal nodes with cophenetic-distance-based merging. This yields a leaf-to-cluster assignment (VISION_Clusters_Tree) usable as lineage-aware metadata in downstream VISION/Hotspot analyses.

## When to use

Use when you have a single-cell phylogeny (Newick/phylo object with cells as leaves) and need to group cells into a target number of clade-coherent subpopulations, e.g. to define tumor subclones or lineage labels for signature-scoring.

## When NOT to use

- The input is a flat expression matrix or annotation table with no phylogenetic tree — clustering must instead use expression-based methods (e.g. UMAP/Hotspot modules)
- Clusters need not correspond to connected clades (e.g. purely transcriptional communities) — cophenetic merging will force lineage-coherent groups
- The tree has unresolved unifurcations and you cannot preprocess it — the algorithm assumes every internal node branches properly after collapse.singles

## Inputs

- Single-cell phylogeny in Newick format (cells as leaves), loadable as an R ape 'phylo' object
- Target cluster count (default target := 10)

## Outputs

- Leaf-to-cluster assignment stored as the VISION_Clusters_Tree metadata variable
- Two-column table (cell identifier, cluster label) written to cluster_assignments.csv with exactly the target number of distinct labels

## How to apply

Load the lineage-traced phylogeny into R as a phylo object and collapse unifurcations with ape's collapse.singles so every internal node properly branches. Then run the Cluster-Phylogeny procedure (default target := 10): maintain a priority queue keyed on the maximum size of each internal node's child clades, start with the root, and repeatedly pop the largest-node while inserting its immediate children until the queue holds the target number of internal nodes. Collect the leaf cells beneath each queued node as candidate clusters, and while more clusters than the target remain, merge the smallest cluster with its phylogenetically nearest cluster by cophenetic distance until exactly the target count is reached. Write the final leaf-to-cluster mapping as the VISION_Clusters_Tree metadata variable and export it as a two-column table (cell identifier, cluster label) to cluster_assignments.csv.

## Related tools

- **PhyloVision** (Implements the breadth-first Cluster-Phylogeny cell-clustering over internal nodes and the VISION_Clusters_Tree metadata output) — https://github.com/YosefLab/VISION
- **ape** (R package whose collapse.singles function removes unifurcations from the phylo tree before clustering)
- **R** (Language in which PhyloVision and the clustering implementation are executed)

## Examples

```
tree <- ape::collapse.singles(ape::read.tree("lineage.nwk")); clusters <- Cluster-Phylogeny(tree, target = 10)
```

## Evaluation signals

- cluster_assignments.csv contains exactly the target number of distinct cluster labels (default 10)
- Every leaf cell present in the input phylo tree appears exactly once in the assignment table
- Each cluster's cell set equals the complete leaf set of one internal node selected by the breadth-first traversal (clade-coherent partitions)
- Merged clusters are adjacent by cophenetic distance — the smallest cluster is always merged to its nearest cluster, so post-merge no orphan singleton cluster smaller than necessary remains
- Assignment round-trips into PhyloVision as the VISION_Clusters_Tree metadata variable without missing or extra cells

## Limitations

- PhyloVision's dynamic phylogeny viewer was tested on trees of up to ~4,000 leaves; much larger trees may strain performance
- Because the implementation is in R, integrating with Python software such as Cassiopeia (used to reconstruct the phylogenies) may be challenging
- Related phylogeny-derived quantities such as relative fitness are not directly supported within the software
- Analyses of lineage data with this pipeline (e.g. the 1,127-cell metastatic lung adenocarcinoma clone) still lack experimental validation of generated hypotheses

## Evidence

- [methods] Breadth-first clustering over internal nodes: "PhyloVision clusters cells on a phylogeny by performing a breadth-first search over internal nodes"
- [methods] Target cluster count default: "The algorithmic pseudocode is detailed as follows: Cluster-Phylogeny (target := 10)"
- [methods] Tree preprocessing with ape: "collapsed using the collapse.singles function in the R package ape"
- [results] Scale of trees supported: "allowing low-latency selections and subtree collapsing for large trees (we tested trees of up to ∼4,000 leaves)"
- [results] Demonstration dataset scale: "we apply it to a clone of 1,127 cells from a recent CRISPR-Cas9 lineage-tracing dataset investigating metastatic spread in a mouse model of lung adenocarcinoma"
