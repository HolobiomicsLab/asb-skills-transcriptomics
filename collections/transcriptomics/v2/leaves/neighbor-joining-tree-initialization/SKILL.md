---
name: neighbor-joining-tree-initialization
description: 'Use when starting LinRace lineage reconstruction for a barcode group:
  you have a character matrix of mutated barcode loci and need an initial binary topology
  before likelihood-based local search.'
when_to_use_negative:
- Cells do not share an identical barcode group and fine-scale subtree resolution
  is not needed (barcode data alone already resolves them)
- Barcode data quality is too low (short barcodes, dropout, non-optimal mutation rates)
  such that Hamming distances are uninformative — the article notes these challenges
  degrade lineage inference
- You only need the final ML-refined tree and no initial-tree control; use the full
  NJ_asym_Dif pipeline instead of running NJ manually
edam_operation: http://edamontology.org/operation_0557
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_2229
tools:
- name: LinRace
  role: Provides MutsTransform, distH, and NJ_asym_Dif which wrap the NJ initialization
    inside the full likelihood-based lineage reconstruction pipeline
  repo: github.com/ZhangLabGT/LinRace
- name: phangorn
  role: Provides dist.hamming for computing pairwise Hamming distances on transformed
    barcode characters
- name: ape
  role: Provides nj for Neighbor Joining tree construction, multi2di to resolve the
    NJ tree to binary, and write.tree to serialize the final tree
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:497a115a7ae3477c6750535537d419e86850bd0700f8e549cf417df7c1ef332f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LinRace
  - R
  - ape
  - phangorn
  edam_operation: http://edamontology.org/operation_0557
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# neighbor-joining-tree-initialization

## Summary

Initializes a candidate cell lineage tree for LinRace by computing pairwise Hamming distances on transformed lineage barcode characters and running Neighbor Joining, providing a starting tree that the rSS maximum-likelihood local search subsequently refines.

## When to use

Use when starting LinRace lineage reconstruction for a barcode group: you have a character matrix of mutated barcode loci and need an initial binary topology before likelihood-based local search.

## When NOT to use

- Cells do not share an identical barcode group and fine-scale subtree resolution is not needed (barcode data alone already resolves them)
- Barcode data quality is too low (short barcodes, dropout, non-optimal mutation rates) such that Hamming distances are uninformative — the article notes these challenges degrade lineage inference
- You only need the final ML-refined tree and no initial-tree control; use the full NJ_asym_Dif pipeline instead of running NJ manually

## Inputs

- Lineage barcode character matrix (e.g., cm_ZF1_F3.txt, tab-separated, row names = cell IDs) for cells sharing an identical barcode
- Cell-state cluster labels (e.g., kmeans clusters on top 50 PCs of expression data)
- Gene-expression count matrix (e.g., zebrafish scGESTALT ZF1_F3_DrImpute counts)
- Cell state tree / state lineage specification

## Outputs

- Binary starting lineage tree (newick/phylo object) with all edge lengths set to 1
- Seed tree input to NJ_asym_Dif for subsequent maximum-likelihood subtree-swap search

## How to apply

First transform the raw barcode mutation matrix (MutsTransform), then compute the pairwise Hamming distance between cells on the transformed barcodes (dist.hamming / distH). Run Neighbor Joining (nj from phangorn/ape) on this distance matrix to obtain an unrooted tree, resolve any multifurcations to a strictly binary tree with multi2di (every non-leaf node must have exactly two children), and set all edge lengths to 1. This NJ tree is passed to NJ_asym_Dif together with the expression matrix, cell-state cluster labels, and state lineages (e.g., max_Iter = 500), where it serves as the seed for subsequent LikelihoodCal scoring, AncesInfer, and hill-climbing subtree-swap (TreeLC2) refinement. The rationale is that barcode Hamming distance gives a computationally cheap, reasonably accurate starting topology so the likelihood search converges from a good initial point.

## Related tools

- **LinRace** (Provides MutsTransform, distH, and NJ_asym_Dif which wrap the NJ initialization inside the full likelihood-based lineage reconstruction pipeline) — github.com/ZhangLabGT/LinRace
- **phangorn** (Provides dist.hamming for computing pairwise Hamming distances on transformed barcode characters)
- **ape** (Provides nj for Neighbor Joining tree construction, multi2di to resolve the NJ tree to binary, and write.tree to serialize the final tree)

## Examples

```
dist_h <- dist.hamming(sim_data); Treenj = nj(dist_h); Treenj_bin = multi2di(Treenj); Treenj_bin$edge.length[] <- 1
```

## Evaluation signals

- The resulting tree is strictly binary — every non-leaf node has exactly two children (verified after multi2di)
- All edge lengths in the initialized tree equal 1
- The distance matrix is symmetric with zero diagonal, computed on MutsTransform-transformed barcodes
- Running LikelihoodCal + AncesInfer on the NJ seed tree succeeds and yields a finite log-likelihood before the TreeLC2 hill-climbing loop starts
- Final refined tree scores higher log-likelihood than the NJ initial tree, and can be compared to ground truth via normalized Robinson-Foulds distance (RF.dist normalize=TRUE)

## Limitations

- LinRace's downstream model assumes irreversible cell state transitions; reversible transitions are not modeled, so NJ initialization feeds a tree that cannot capture them
- Assumes all cell states during development exist in current-day cells; ancestral states reconstructed on the tree are limited to cell states present in the scRNA-seq data
- Barcode data alone cannot resolve cells sharing an identical barcode, so the NJ tree within such a group is only a heuristic seed that must be refined by likelihood search

## Evidence

- [methods] NJ initialization inside LinRace pipeline: "tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)"
- [methods] NJ on Hamming distance of barcode data: "dist_h <- dist.hamming(sim_data); Treenj = nj(dist_h)"
- [abstract] LinRace integrates barcode and expression with NJ + maximum-likelihood heuristics: "which integrates lineage barcode and gene expression data using asymmetric cell division model and infers cell lineages and ancestral cell states using Neighbor-Joining and maximum-likelihood"
- [readme] LinRace combines NJ and maximum-likelihood framework: "infers cell lineage under a framework combining Neighbor Joining and maximum-likelihood heuristics"
