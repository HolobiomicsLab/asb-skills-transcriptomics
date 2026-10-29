---
name: r-phylogenetics-programming-with-ape
description: Use when a Newick-format single-cell phylogeny must be loaded as a phylo
  object, cleaned of unifurcations, and traversed in R to cluster tip cells or compute
  tree-based metadata such as DepthClusters or VISION_Clusters_Tree.
when_to_use_negative:
- Phylogenies were reconstructed in Python (e.g., with Cassiopeia) and the whole pipeline
  will stay in Python — the article notes the R implementation complicates interoperability
  with Python tools
- The task is gene-module discovery or signature autocorrelation (use Hotspot / VISION
  clustering instead of raw tree cutting)
- The input is a pairwise distance matrix without an explicit tree structure, since
  the traversal requires internal nodes of a phylo object
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_3308
tools:
- name: ape
  role: R package providing phylo objects and the collapse.singles function used to
    remove unifurcations before tree-based clustering
- name: PhyloVision
  role: Reference R implementation of the Cluster-Phylogeny breadth-first search that
    clusters cells on the collapsed phylogeny (target := 10)
  repo: https://github.com/YosefLab/VISION
- name: Cassiopeia
  role: Upstream Python package that reconstructs the single-cell phylogenies consumed
    as Newick input
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:6e6f2efa186d3b2b680e7d2f69c69992c4dc213b1942de6c2692598a7ceb0544
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - Cassiopeia
  - PhyloVision
  - R
  - ape
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# R phylogenetics programming with ape

## Summary

Programmatic manipulation of single-cell phylogenies in R using the ape package's phylo objects — reading trees, collapsing unifurcations with collapse.singles, and traversing internal nodes (breadth-first) to derive cell-to-cluster assignments as implemented in PhyloVision's Cluster-Phylogeny algorithm.

## When to use

Use when a Newick-format single-cell phylogeny must be loaded as a phylo object, cleaned of unifurcations, and traversed in R to cluster tip cells or compute tree-based metadata such as DepthClusters or VISION_Clusters_Tree.

## When NOT to use

- Phylogenies were reconstructed in Python (e.g., with Cassiopeia) and the whole pipeline will stay in Python — the article notes the R implementation complicates interoperability with Python tools
- The task is gene-module discovery or signature autocorrelation (use Hotspot / VISION clustering instead of raw tree cutting)
- The input is a pairwise distance matrix without an explicit tree structure, since the traversal requires internal nodes of a phylo object

## Inputs

- Newick-format phylogeny file with one leaf per cell (leaf names matching cell barcodes)
- phylo object (R, ape package)
- User-defined tree-cut depth or target cluster count (default target := 10)

## Outputs

- Collapsed phylo object with no unifurcations
- Leaf-to-cluster assignment vector (e.g., VISION_Clusters_Tree metadata variable)
- Two-column cluster_assignments.csv (cell identifier, cluster label)

## How to apply

Load the Newick tree with ape's read.tree into a phylo object and run collapse.singles so every internal node branches properly (removing degree-2 nodes), as PhyloVision does before clustering. Implement the Cluster-Phylogeny traversal: perform a breadth-first search over internal nodes — using a priority queue keyed on the largest child clade — until the target cluster count is reached (target := 10 by default). Group the leaf labels beneath each queued internal node into candidate clusters; if more clusters remain than the target, merge the smallest cluster with its phylogenetically nearest cluster (by cophenetic distance) until exactly 10 clusters remain. Store the leaf-to-cluster assignment as a metadata variable (e.g., meta$DepthClusters / VISION_Clusters_Tree) and export it as a two-column table (cell identifier, cluster label). The rationale is that cells sharing recent ancestry inherit expression programs, so tree topology provides a biologically meaningful grouping for downstream Hotspot/VISION analyses.

## Related tools

- **ape** (R package providing phylo objects and the collapse.singles function used to remove unifurcations before tree-based clustering)
- **PhyloVision** (Reference R implementation of the Cluster-Phylogeny breadth-first search that clusters cells on the collapsed phylogeny (target := 10)) — https://github.com/YosefLab/VISION
- **Cassiopeia** (Upstream Python package that reconstructs the single-cell phylogenies consumed as Newick input)

## Examples

```
library(ape); tr <- read.tree("phylogeny.nwk"); tr <- collapse.singles(tr)
```

## Evaluation signals

- Output contains exactly 10 distinct cluster labels (matching Cluster-Phylogeny target := 10)
- Every leaf cell in the input tree appears exactly once in the assignment table (no duplicates, no drops)
- Running collapse.singles leaves no internal nodes with a single child in the phylo object
- Cluster labels partition tips only by descent: each cluster's members form the complete leaf set below one internal node at the cut depth
- The exported cluster_assignments.csv parses as a two-column table (cell identifier, cluster label) consistent with the metadata variable

## Limitations

- PhyloVision is implemented in R, which 'may provide challenges for utilizing separate software available in Python, like Cassiopeia'
- The priority-queue descent prioritizes nodes by the largest child clade, so cluster boundaries depend on the queue order, not just the user-defined depth
- When candidate clusters exceed the target, smallest-cluster merging by cophenetic distance can override purely depth-based groupings

## Evidence

- [methods] collapse.singles: "collapsed using the collapse.singles function in the R package ape"
- [methods] breadth-first clustering: "PhyloVision clusters cells on a phylogeny by performing a breadth-first search over internal nodes"
- [methods] Cluster-Phylogeny pseudocode: "The algorithmic pseudocode is detailed as follows: Cluster-Phylogeny (target := 10)"
- [discussion] R/Python interoperability caveat: "our implementation of PhyloVision in the R programming language may provide challenges for utilizing separate software available in Python, like Cassiopeia"
