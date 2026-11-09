---
name: phylogenetic-tree-breadth-first-clustering
description: Use when you have a single-cell lineage tree (Newick/phylo with cells
  as leaves) and need to partition tip cells into a fixed number of subpopulations
  that each correspond to a clade of the reconstructed phylogeny.
when_to_use_negative:
- You need expression-driven (transcriptional) clusters rather than lineage/clade-defined
  subpopulations — this skill partitions by tree topology, not expression similarity
- The tree has not had unifurcations collapsed; running on a tree with single-child
  internal nodes breaks the assumption that every internal node branches properly
- You need continuous lineage metrics such as relative fitness, which the article
  notes 'are not directly supported within our software'
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3308
tools:
- name: PhyloVision
  role: Provides the Cluster-Phylogeny breadth-first clustering over internal nodes
    and the VISION_Clusters_Tree metadata output
  repo: https://github.com/YosefLab/VISION
- name: ape
  role: Loads the Newick phylogeny as a phylo object and collapses unifurcations with
    collapse.singles before clustering
- name: R
  role: Language in which PhyloVision's clustering is implemented
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:22942d973cf7a5269192c477fabc4ca18125e7b609066ccc183b70bf50e4e1d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - PhyloVision
  - R
  - ape
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# phylogenetic-tree-breadth-first-clustering

## Summary

Clusters cells on a single-cell phylogeny (ape phylo object) by a breadth-first search over internal nodes, collecting the leaves beneath selected nodes into exactly `target` (default 10) phylogenetically coherent subpopulation labels such as VISION_Clusters_Tree / meta$DepthClusters.

## When to use

Use when you have a single-cell lineage tree (Newick/phylo with cells as leaves) and need to partition tip cells into a fixed number of subpopulations that each correspond to a clade of the reconstructed phylogeny.

## When NOT to use

- You need expression-driven (transcriptional) clusters rather than lineage/clade-defined subpopulations — this skill partitions by tree topology, not expression similarity
- The tree has not had unifurcations collapsed; running on a tree with single-child internal nodes breaks the assumption that every internal node branches properly
- You need continuous lineage metrics such as relative fitness, which the article notes 'are not directly supported within our software'

## Inputs

- Single-cell phylogeny in Newick format with cell barcodes as tip labels, loaded as an ape phylo object
- Target cluster count (default target := 10)

## Outputs

- Leaf-to-cluster assignment stored as the VISION_Clusters_Tree metadata variable (e.g., meta$DepthClusters)
- Two-column cluster_assignments.csv table: cell identifier and cluster label
- Exactly `target` (default 10) distinct cluster labels

## How to apply

Load the Newick lineage tree with ape and run collapse.singles so every internal node branches properly (no unifurcations). Run PhyloVision's Cluster-Phylogeny algorithm (target := 10 by default): initialize a priority queue keyed on the maximum child-clade size, insert the root, and repeatedly pop the node with the largest child clade while pushing its immediate children until the queue holds the target number of internal nodes — this is the breadth-first search over internal nodes described in the methods. Group the leaves beneath each queued node into candidate clusters, then while more clusters than the target remain, take the smallest cluster and merge it with its phylogenetically nearest cluster by cophenetic distance until exactly the target number of clusters is obtained. Store the final leaf-to-cluster assignment as the VISION_Clusters_Tree metadata variable (e.g., writing into meta$DepthClusters) and export it as a two-column table (cell identifier, cluster label) to cluster_assignments.csv.

## Related tools

- **PhyloVision** (Provides the Cluster-Phylogeny breadth-first clustering over internal nodes and the VISION_Clusters_Tree metadata output) — https://github.com/YosefLab/VISION
- **ape** (Loads the Newick phylogeny as a phylo object and collapses unifurcations with collapse.singles before clustering)
- **R** (Language in which PhyloVision's clustering is implemented)

## Examples

```
tree <- collapse.singles(read.tree("phylogeny.nwk")); meta$DepthClusters <- clusterByDepth(tree, target = 10); write.csv(data.frame(cell = names(meta$DepthClusters), cluster = meta$DepthClusters), "cluster_assignments.csv", row.names = FALSE)
```

## Evaluation signals

- Output cluster_assignments.csv contains exactly the target number of distinct cluster labels (10 with default target := 10)
- Every leaf cell in the input phylogeny appears exactly once in the assignment table (a bijective cell-to-label mapping)
- Each cluster's members form the complete leaf set beneath a single internal node from the breadth-first queue (clades are monophyletic in the input tree)
- collapse.singles has removed all unifurcations, i.e., every internal node in the preprocessed tree has >= 2 children
- When cluster count exceeds the target, merges join the smallest cluster to its cophenetic-distance-nearest cluster, so final clusters remain phylogenetically adjacent

## Limitations

- Default target is 10 clusters; other targets must be requested explicitly via the target parameter of Cluster-Phylogeny
- PhyloVision is implemented in R, which the authors note 'may provide challenges for utilizing separate software available in Python, like Cassiopeia'
- The dynamic phylogeny viewer was only tested on trees of up to ~4,000 leaves, suggesting practical scale limits for very large lineages

## Evidence

- [methods] Breadth-first clustering over internal nodes: "PhyloVision clusters cells on a phylogeny by performing a breadth-first search over internal nodes"
- [methods] Default target cluster count: "The algorithmic pseudocode is detailed as follows: Cluster-Phylogeny (target := 10)"
- [methods] Unifurcation collapsing prerequisite: "collapsed using the collapse.singles function in the R package ape"
- [intro] PhyloVision as the integrating lineage/expression tool: "we introduce PhyloVision: an open-source, interactive analysis and visualization tool that is expressly built for integrating single-cell gene expression and lineage data"
- [other] Software availability for reproducing the clustering: "PhyloVision is distributed publicly on Github at https://github.com/YosefLab/VISION"
