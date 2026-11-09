---
name: nye-similarity-tree-comparison
description: Use when comparing two or more phylogenetic/lineage trees (e.g., reconstructed
  vs. ground-truth cell division trees) and you need a branch-clade-split based similarity
  that is normalized to [0,1], as in the LinRace benchmark.
when_to_use_negative:
- Input trees have unmatched leaf labels (different cell sets) — the metric requires
  the same leaves in both trees
- You need an actual distance for clustering/averaging — convert via 1 - NyeSimilarity
  or use RF.dist(normalize = TRUE) instead
edam_operation: http://edamontology.org/operation_0556
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3173
tools:
- name: TreeDist
  role: Provides NyeSimilarity(..., normalize = TRUE) used to score reconstructed
    vs. ground-truth lineage trees
- name: ape
  role: Supplies RF.dist(normalize = TRUE) and tree handling (multi2di) for the trees
    being compared
- name: LinRace
  role: Produces the reconstructed cell-division trees that are scored against the
    truth with Nye similarity
  repo: https://github.com/ZhangLabGT/LinRace
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:a91e72065b688e75cc1ca6ae099d3cafb4cd838dd9b599efff0622e2f1d0a5d0
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
  - Startle
  - TedSim
  - TreeDist
  - ape
  - ggplot2
  - phangorn
  edam_operation: http://edamontology.org/operation_0556
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# nye-similarity-tree-comparison

## Summary

Scores similarity between reconstructed lineage/cell-division trees using the normalized NyeSimilarity metric from TreeDist, complementing RF distance and CID in the LinRace benchmark on TedSim-simulated and C. elegans L1 datasets.

## When to use

Use when comparing two or more phylogenetic/lineage trees (e.g., reconstructed vs. ground-truth cell division trees) and you need a branch-clade-split based similarity that is normalized to [0,1], as in the LinRace benchmark.

## When NOT to use

- Input trees have unmatched leaf labels (different cell sets) — the metric requires the same leaves in both trees
- You need an actual distance for clustering/averaging — convert via 1 - NyeSimilarity or use RF.dist(normalize = TRUE) instead

## Inputs

- Reconstructed lineage tree(s) in ape/phylo format (e.g., from LinRace NJ_asym_Dif or nj on dist.hamming)
- Ground-truth cell division tree (e.g., TedSim-simulated phyla or CeLaVi C. elegans L1 Newick tree)

## Outputs

- Normalized Nye similarity score(s) in [0,1] per tree comparison
- Pairwise NyeSimilarity matrix across method trees (Nye_mutual)
- Benchmark results table (e.g., fig3c_l1_benchmark.csv) aggregating scores by method x mutation rate x dropout condition

## How to apply

After reconstructing trees from lineage barcode data (LinRace NJ_asym_Dif with max_Iter=500, lambda1=10, lambda2=1, plus baselines NJ on dist.hamming, DCLEAR, Cassiopeia, Startle, LinTIMaT), compute NyeSimilarity(tree, truth, normalize = TRUE) via TreeDist in R. Aggregate scores per method x mutation rate (mu in c(0.025, 0.05, 0.075)) x dropout condition, and optionally build a full pairwise NyeSimilarity matrix among method trees (as in the zebrafish vignette). Higher values indicate better agreement with the ground-truth tree; unlike RF distance, Nye similarity is a similarity (not distance) so interpret sign accordingly.

## Related tools

- **TreeDist** (Provides NyeSimilarity(..., normalize = TRUE) used to score reconstructed vs. ground-truth lineage trees)
- **ape** (Supplies RF.dist(normalize = TRUE) and tree handling (multi2di) for the trees being compared)
- **LinRace** (Produces the reconstructed cell-division trees that are scored against the truth with Nye similarity) — https://github.com/ZhangLabGT/LinRace

## Examples

```
Nye_sim <- NyeSimilarity(tree_linrace, tree_ct, normalize = TRUE)
```

## Evaluation signals

- NyeSimilarity returns a value normalized to [0,1]; verify normalize = TRUE was set so scores are comparable across trees of different sizes
- LinRace trees should achieve higher Nye similarity than all baselines across swept mutation rates (0.025-0.075) and both dropout conditions
- Aggregated CSV values should match the deposited Fig. 2 / Fig. 3c Source Data references
- Self-comparison sanity check: NyeSimilarity(tree, tree, normalize = TRUE) must equal 1

## Limitations

- Nye similarity is a similarity (higher = better), opposite in polarity to RF distance; naive aggregation or averaging with RF/CID without sign correction will invert rankings
- Benchmark results depend on barcode technology quality; the paper notes results 'depend on future improvements in lineage barcode technology (high diversity and coverage)'
- No software versions for TreeDist/ape are stated in the methods, so exact reproducibility of scores may vary across package versions

## Evidence

- [full_text] pairwise NyeSimilarity (normalize = TRUE) among method trees via TreeDist and CID metrics: [evidence span withheld — anchor status: not_found]
- [full_text] LinRace outperforms baselines on C. elegans with Nye similarity: [evidence span withheld — anchor status: not_found]
- [methods] vignette Nye comparison code: "Nye_mutual[i,j] <- NyeSimilarity(tree_list[[i]], tree_list[[j]], normalize = TRUE)"
- [abstract] LinRace integrates barcodes and expression via NJ + ML: "which integrates lineage barcode and gene expression data using asymmetric cell division model and infers cell lineages and ancestral cell states using Neighbor-Joining and maximum-likelihood"
