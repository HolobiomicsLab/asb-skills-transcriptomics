---
name: cophenetic-distance-neighbor-merging
description: Use when a breadth-first clustering of leaves over internal nodes of
  a single-cell phylogeny yields more candidate clusters than the user-defined target
  (default target := 10) and the count must be reduced while preserving phylogenetic
  coherence.
when_to_use_negative:
- The tree is not a single-cell phylogeny with cells as labeled tips (e.g., a species
  tree) — the breadth-first internal-node grouping assumes cell leaves
- The number of candidate clusters from the breadth-first search is already at or
  below the target count — no merging pass is needed
- Transcriptomic (non-phylogenetic) clustering is intended — this skill groups cells
  strictly by tree topology and cophenetic distance, not expression
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: PhyloVision
  role: Implements the Cluster-Phylogeny algorithm whose merging step joins surplus
    clusters by cophenetic distance; results stored as VISION_Clusters_Tree
  repo: https://github.com/YosefLab/VISION
- name: ape
  role: R package used to collapse unifurcations in the input phylogeny (collapse.singles)
    prior to clustering
- name: R
  role: Language in which the PhyloVision clustering implementation is written
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:ca64e0acec1c3ede5c0b0b4784b76e03ba3a04467d62a2898b12be591245c30e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - PhyloVision
  - R
  - ape
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Cophenetic-distance neighbor merging

## Summary

A post-processing step of PhyloVision's Cluster-Phylogeny algorithm: after breadth-first grouping of leaves under internal nodes, surplus clusters are reduced to the target count (default := 10) by iteratively merging the smallest cluster with its phylogenetically nearest cluster as measured by cophenetic distance.

## When to use

Use when a breadth-first clustering of leaves over internal nodes of a single-cell phylogeny yields more candidate clusters than the user-defined target (default target := 10) and the count must be reduced while preserving phylogenetic coherence.

## When NOT to use

- The tree is not a single-cell phylogeny with cells as labeled tips (e.g., a species tree) — the breadth-first internal-node grouping assumes cell leaves
- The number of candidate clusters from the breadth-first search is already at or below the target count — no merging pass is needed
- Transcriptomic (non-phylogenetic) clustering is intended — this skill groups cells strictly by tree topology and cophenetic distance, not expression

## Inputs

- A single-cell phylogeny as a phylo object (Newick format) with each cell as a tip, collapsed of unifurcations
- Target cluster count (default target := 10)

## Outputs

- Leaf-to-cluster assignment stored as the VISION_Clusters_Tree metadata variable
- Two-column table (cell identifier, cluster label) written to cluster_assignments.csv

## How to apply

First cluster leaves by a breadth-first search over internal nodes of the phylo tree (collapsed of unifurcations with ape's collapse.singles), assigning each internal node in the queue the set of descendant tip labels as a candidate cluster. If the number of candidate clusters exceeds the target (default target := 10), compute phylogenetic distances between clusters (cophenetic distance on the tree), select the smallest cluster, and merge it with its nearest neighbor cluster; repeat until exactly the target number of clusters remains. Store the final leaf-to-cluster assignment as the VISION_Clusters_Tree metadata variable and write it as a two-column table (cell identifier, cluster label) to cluster_assignments.csv. The rationale is that merging by cophenetic distance removes the smallest, least-supported subpopulations without cutting across deep phylogenetic splits.

## Related tools

- **PhyloVision** (Implements the Cluster-Phylogeny algorithm whose merging step joins surplus clusters by cophenetic distance; results stored as VISION_Clusters_Tree) — https://github.com/YosefLab/VISION
- **ape** (R package used to collapse unifurcations in the input phylogeny (collapse.singles) prior to clustering)
- **R** (Language in which the PhyloVision clustering implementation is written)

## Examples

```
tree <- ape::collapse.singles(ape::read.tree("phylogeny.nwk")); clusters <- ClusterPhylogeny(tree, target := 10)
```

## Evaluation signals

- The final assignment contains exactly 10 distinct cluster labels (matching target := 10)
- Every leaf cell in the input tree appears exactly once in the cluster_assignments.csv table
- The assignment is stored as the VISION_Clusters_Tree metadata variable
- Merged clusters are the smallest ones and are joined to their cophenetically nearest cluster at each iteration

## Limitations

- The R implementation of PhyloVision may pose interoperability challenges for combining with separate Python software like Cassiopeia
- Relative fitness quantification and other phylogeny-derived characteristics are not directly supported within the software

## Evidence

- [methods] Breadth-first internal-node clustering: "PhyloVision clusters cells on a phylogeny by performing a breadth-first search over internal nodes"
- [methods] Cluster-Phylogeny pseudocode with target := 10: "The algorithmic pseudocode is detailed as follows: Cluster-Phylogeny (target := 10)"
- [methods] Unifurcation collapsing with ape: "collapsed using the collapse.singles function in the R package ape"
- [results] PhyloVision pipeline scope: "PhyloVision is an integrated pipeline for interactive analysis of single-cell expression and lineage profiles"
