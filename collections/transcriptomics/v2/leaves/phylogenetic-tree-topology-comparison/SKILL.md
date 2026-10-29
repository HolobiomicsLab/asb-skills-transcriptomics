---
name: phylogenetic-tree-topology-comparison
description: Use when several lineage or phylogenetic trees have been reconstructed
  from the same cell set (e.g., competing lineage-reconstruction methods) and you
  need to compare their local resolution via maximum root-to-tip depth and internal-node
  counts.
when_to_use_negative:
- When edge lengths carry biological meaning and must be preserved (this skill discards
  edge lengths to compare topology only)
- When trees cover different cell sets — node and depth counts are not comparable
  across different tips
- When accuracy against a ground-truth tree is needed — use distance metrics like
  normalized Robinson-Foulds (RF.dist) instead of local-resolution counts
edam_operation: http://edamontology.org/operation_0552
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0102
tools:
- name: ape
  role: Used to read Newick trees, resolve multifurcations with multi2di, discard
    edge lengths, and compute internal-node counts (Nnode) and maximum depth
- name: LinRace
  role: Reconstructs the lineage tree via NJ_asym_Dif with max_Iter = 500 from the
    character matrix, k-means cluster labels, and expression data; its tree is the
    focal comparison target
  repo: https://github.com/ZhangLabGT/LinRace
- name: Cassiopeia
  role: Baseline parsimony-based lineage reconstruction; its tree is loaded as Newick,
    the 'normal' tip dropped, and polytomies resolved before comparison
- name: LinTIMaT
  role: Baseline combined-likelihood lineage reconstruction; its precomputed tree
    is read as Newick for topology comparison
- name: DCLEAR
  role: Baseline distance-based lineage reconstruction (neighbor joining); its tree
    is read as Newick for topology comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:d4dcc3cca44ab0e0b2f558bc58a6c1a743f18e8da0a8609b8d87e29a4b6fc935
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/LinRace
  tools:
  - Cassiopeia
  - DCLEAR
  - LinRace
  - LinTIMaT
  - R
  - ape
  - ggplot2
  - phangorn
  edam_operation: http://edamontology.org/operation_0552
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Phylogenetic tree topology comparison

## Summary

Compares reconstructed lineage/phylogenetic trees from multiple methods on local-resolution metrics — maximum root-to-tip depth and number of internal nodes — to quantify how much hierarchical structure each reconstruction recovers, as done for LinRace vs Cassiopeia, LinTIMaT, and DCLEAR on the scGESTALT zebrafish ZF1-F3 sample.

## When to use

Use when several lineage or phylogenetic trees have been reconstructed from the same cell set (e.g., competing lineage-reconstruction methods) and you need to compare their local resolution via maximum root-to-tip depth and internal-node counts.

## When NOT to use

- When edge lengths carry biological meaning and must be preserved (this skill discards edge lengths to compare topology only)
- When trees cover different cell sets — node and depth counts are not comparable across different tips
- When accuracy against a ground-truth tree is needed — use distance metrics like normalized Robinson-Foulds (RF.dist) instead of local-resolution counts

## Inputs

- Newick tree files for each method, e.g., tree_LinTIMaT_ZF1_F3.newick, tree_cassiopeia_ZF1_F3.processed.newick, tree_DCLEAR_nj_ZF1_F3.newick, and the LinRace tree
- Optionally a character matrix (cm_ZF1_F3.txt) and expression matrix (ZF1_F3_DrImpute) if trees must be reconstructed first with NJ_asym_Dif (max_Iter = 500)

## Outputs

- Per-method table of maximum root-to-tip depth and number of internal nodes
- Verified ordering of methods by local resolution (e.g., LinRace > Cassiopeia, LinTIMaT on depth and Nnode)

## How to apply

Load each reconstructed tree (Newick format) and strip edge lengths so only topology is compared, since different methods calibrate branch lengths differently. For trees exported with multifurcations, such as the Cassiopeia tree (after dropping the 'normal' tip), resolve polytomies with ape::multi2di before counting nodes. For each tree compute two metrics: the maximum total edge length from the root to any leaf (maximum depth) and the number of internal nodes (ape Nnode). Assemble the per-method depth and Nnode values into a table and check the expected ordering — on the 750-cell ZF1-F3 scGESTALT zebrafish brain sample, LinRace should produce a deeper tree with more internal nodes than Cassiopeia and LinTIMaT, indicating higher local resolution. Optionally supplement with topological similarity measures such as normalized Nye similarity when comparing multiple reconstructions pairwise.

## Related tools

- **ape** (Used to read Newick trees, resolve multifurcations with multi2di, discard edge lengths, and compute internal-node counts (Nnode) and maximum depth)
- **LinRace** (Reconstructs the lineage tree via NJ_asym_Dif with max_Iter = 500 from the character matrix, k-means cluster labels, and expression data; its tree is the focal comparison target) — https://github.com/ZhangLabGT/LinRace
- **Cassiopeia** (Baseline parsimony-based lineage reconstruction; its tree is loaded as Newick, the 'normal' tip dropped, and polytomies resolved before comparison)
- **LinTIMaT** (Baseline combined-likelihood lineage reconstruction; its precomputed tree is read as Newick for topology comparison)
- **DCLEAR** (Baseline distance-based lineage reconstruction (neighbor joining); its tree is read as Newick for topology comparison)

## Examples

```
library(ape); tr <- multi2di(read.tree("tree_cassiopeia_ZF1_F3.processed.newick")); c(maxDepth = max(node.depth.edgelength(tr)), Nnode = Nnode(tr))
```

## Evaluation signals

- Each tree parses as a valid phylo object from Newick and contains the expected tip count (750 cells for ZF1-F3)
- Multifurcations are resolved (multi2di) before counting internal nodes, so no unresolved polytomies remain in the Cassiopeia tree
- The recomputed max-depth and Nnode ordering matches the Fig. 4b expectation: LinRace deeper with more internal nodes than Cassiopeia and LinTIMaT
- Per-method table contains one depth and one Nnode value per method with no missing entries
- Edge lengths are confirmed to be discarded so the comparison reflects topology only

## Limitations

- Comparing with edge lengths discarded means branch-length-dependent differences between methods are invisible
- Resolving multifurcations (multi2di) arbitrarily inserts resolution that the original Cassiopeia output did not contain, which can inflate internal-node counts
- Maximum depth and Nnode measure local resolution, not accuracy; a deeper tree is not necessarily more correct

## Evidence

- [readme] README definition of max-depth metric: "The max depth means the maximum total edge length going from the root to any leaf cell."
- [methods] LinRace tree reconstruction call used to produce the focal tree: "tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)"
- [methods] Pairwise topological comparison of inferred trees: "Nye_mutual[i,j] <- NyeSimilarity(tree_list[[i]], tree_list[[j]], normalize = TRUE)"
- [readme] README description of the tree-properties comparison figure: [evidence span withheld — anchor status: not_found]
