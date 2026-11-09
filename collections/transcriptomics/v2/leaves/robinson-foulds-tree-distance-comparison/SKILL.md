---
name: robinson-foulds-tree-distance-comparison
description: Use when evaluating inferred lineage/cell division trees (e.g., from
  LinRace, DCLEAR, Cassiopeia, Startle, LinTIMaT, or NJ-on-Hamming baselines) against
  a known ground-truth tree across simulation conditions such as mutation rate or
  dropout.
when_to_use_negative:
- No ground-truth or reference tree is available (e.g., purely exploratory real-data
  trees like the zebrafish scGESTALT ZF1-F3 reconstruction) — use Nye similarity between
  method pairs or qualitative subtree inspection instead of RF-vs-truth scoring
- Trees being compared have different leaf sets (cell label mismatch) — RF.dist requires
  corresponding taxa, so relabel/intersect leaves first
- You need branch-length-aware or ancestral-state agreement measures; RF/Nye/CID capture
  only topology split agreement, not inferred ancestral cell states
edam_operation: http://edamontology.org/operation_0558
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_0091
tools:
- name: ape
  role: Computes normalized Robinson-Foulds distance between reconstructed and ground-truth
    trees (RF.dist, normalize = TRUE)
- name: TreeDist
  role: Computes normalized Nye similarity for pairwise tree comparison (NyeSimilarity,
    normalize = TRUE)
- name: phangorn
  role: Builds the NJ baseline tree from barcode Hamming distances (dist.hamming +
    nj) that is subsequently scored
- name: LinRace
  role: Primary lineage-reconstruction method whose output trees are scored against
    the truth
  repo: https://github.com/ZhangLabGT/LinRace
- name: ggplot2
  role: Plots the aggregated benchmark scores to reproduce Fig. 3c / Supplementary
    Fig. 10
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:f04320c46232c3fa5b749de4b9efaa37a22be5c21f041c49e3d82e37cd6c67b3
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
  edam_operation: http://edamontology.org/operation_0558
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# robinson-foulds-tree-distance-comparison

## Summary

Benchmark lineage-reconstruction methods by scoring each inferred cell division tree against the ground-truth tree using normalized Robinson-Foulds distance (ape::RF.dist, normalize=TRUE), complemented by normalized Nye similarity (TreeDist::NyeSimilarity) and CID, as done in the LinRace paper to compare LinRace against DCLEAR, Cassiopeia, LinTIMaT, Startle, and a barcode-only NJ baseline.

## When to use

Use when evaluating inferred lineage/cell division trees (e.g., from LinRace, DCLEAR, Cassiopeia, Startle, LinTIMaT, or NJ-on-Hamming baselines) against a known ground-truth tree across simulation conditions such as mutation rate or dropout.

## When NOT to use

- No ground-truth or reference tree is available (e.g., purely exploratory real-data trees like the zebrafish scGESTALT ZF1-F3 reconstruction) — use Nye similarity between method pairs or qualitative subtree inspection instead of RF-vs-truth scoring
- Trees being compared have different leaf sets (cell label mismatch) — RF.dist requires corresponding taxa, so relabel/intersect leaves first
- You need branch-length-aware or ancestral-state agreement measures; RF/Nye/CID capture only topology split agreement, not inferred ancestral cell states

## Inputs

- Reconstructed lineage tree(s) in Newick/phylo format from each method (LinRace NJ_asym_Dif output, DCLEAR, Cassiopeia, LinTIMaT, Startle, NJ-on-Hamming via dist.hamming + nj + multi2di)
- Ground-truth cell division tree (TedSim-simulated phyla or true C. elegans L1 lineage tree Newick from CeLaVi)
- Condition metadata: mutation rate (mu), dropout condition (p_d), cell count (ncells), replicate index

## Outputs

- Per-tree metric scores: normalized RF distance, normalized Nye similarity, CID
- Benchmark results data frame / CSV (e.g., fig3c_l1_benchmark.csv) keyed by method x mutation-rate x dropout condition
- ggplot2 comparison plots reproducing Fig. 3c and Supplementary Fig. 10

## How to apply

For each reconstructed tree, load the ground-truth cell division tree (e.g., the TedSim-simulated tree or the true C. elegans L1 Newick from CeLaVi) and compute the normalized Robinson-Foulds distance with ape's RF.dist(tree_inferred, tree_truth, normalize = TRUE); lower RF distance means better reconstruction. Compute complementary metrics so topological agreement is not judged on RF alone: pairwise normalized Nye similarity via TreeDist::NyeSimilarity(tree, truth, normalize = TRUE) (higher is better) and CID. Aggregate scores into a data frame keyed by method x mutation rate (e.g., mu in c(0.025, 0.05, 0.075)) x dropout condition (p_d = 0 vs dropout) x replicate (5 replicates per condition), then write a results CSV (e.g., fig3c_l1_benchmark.csv) and plot method comparisons with ggplot2 to reproduce Fig. 2/3c and Supplementary Fig. 10. Validate by checking that the CSV matches the deposited Source Data values and that the expected winner (LinRace/LinRace-TST/IST) attains the lowest RF distance across swept conditions.

## Related tools

- **ape** (Computes normalized Robinson-Foulds distance between reconstructed and ground-truth trees (RF.dist, normalize = TRUE))
- **TreeDist** (Computes normalized Nye similarity for pairwise tree comparison (NyeSimilarity, normalize = TRUE))
- **phangorn** (Builds the NJ baseline tree from barcode Hamming distances (dist.hamming + nj) that is subsequently scored)
- **LinRace** (Primary lineage-reconstruction method whose output trees are scored against the truth) — https://github.com/ZhangLabGT/LinRace
- **ggplot2** (Plots the aggregated benchmark scores to reproduce Fig. 3c / Supplementary Fig. 10)

## Examples

```
rf <- RF.dist(tree_linrace, tree_ct, normalize = TRUE); nye <- NyeSimilarity(tree_linrace, tree_ct, normalize = TRUE)
```

## Evaluation signals

- RF.dist is called with normalize = TRUE so scores fall in [0,1]; 0 indicates identical splits to the ground-truth tree
- NyeSimilarity is normalized (higher = better) and should anti-correlate with RF distance for the same tree pair
- Aggregated per-condition scores reconcile with the deposited Fig. 2 / Fig. 3c Source Data values
- Expected ordering: LinRace (or LinRace-TST/IST) achieves the lowest RF distance across the swept mutation rates (mu in 0.025-0.075) and both dropout conditions
- Every method x condition x replicate cell (5 replicates per condition) has a non-missing score in the results CSV

## Limitations

- RF distance only compares bipartition splits and ignores edge lengths and ancestral cell states, so trees can differ biologically yet score identically; Nye similarity and CID are used as complements
- Metrics require a known ground-truth tree, so on real data (e.g., zebrafish scGESTALT) evaluation relies on comparisons among methods (pairwise NyeSimilarity) rather than accuracy vs truth
- Scores are sensitive to unrooted-vs-rooted representation and polytomies; baseline NJ trees are collapsed with multi2di to binary before comparison

## Evidence

- [methods] print( RF.dist(TreeNJ_h, tree_ct, normalize = TRUE) ): "print( RF.dist(TreeNJ_h, tree_ct, normalize = TRUE) )"
- [methods] Nye_mutual[i,j] <- NyeSimilarity(tree_list[[i]], tree_list[[j]], normalize = TRUE): "Nye_mutual[i,j] <- NyeSimilarity(tree_list[[i]], tree_list[[j]], normalize = TRUE)"
- [abstract] On both simulated and real data, LinRace outputs more accurate cell division trees than existing methods: "On both simulated and real data, LinRace outputs more accurate cell division trees than existing methods"
- [methods] neighbor joining on the barcode Hamming distance (dist.h then nj with multi2di): "dist_h <- dist.hamming(sim_data); Treenj = nj(dist_h)"
- [readme] LinRace is a method that integrates the lineage barcode and gene expression data using the asymmetric cell division model: "LinRace is a method that integrates the lineage barcode and gene expression data using the asymmetric cell division model and infers cell lineage under a framework combining Neighbor Joining and"
