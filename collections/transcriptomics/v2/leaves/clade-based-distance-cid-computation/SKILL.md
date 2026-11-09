---
name: clade-based-distance-cid-computation
description: Use when assessing whether a lineage-reconstruction method recovers the
  true topology for cells sharing an identical barcode but belonging to different
  true clones (barcode homoplasy), via CID versus random-tree baselines.
when_to_use_negative:
- Evaluating whole-tree reconstruction accuracy against an uncollapsed ground truth
  — use normalized Robinson-Foulds distance instead (RF.dist with normalize = TRUE),
  as in the main simulation comparison
- Comparing inferred trees that all lack a ground-truth reference — use Nye similarity
  (NyeSimilarity with normalize = TRUE) rather than CID
- Barcode homoplasy is absent (each cell group's barcode maps to a single clone),
  so no subtree collapsing or homoplasy-resolution test is needed
tools:
- name: LinRace
  role: Reconstructs the homoplasy-cell subtree via NJ_asym_Dif combining the mutation
    matrix and expression counts; its tree is the test subject of the CID comparison
  repo: https://github.com/ZhangLabGT/LinRace
- name: TedSim
  role: Simulates the paired lineage-barcode / gene-expression dataset and provides
    the true cell division tree (tree_ct) used as the collapsed reference
  repo: https://github.com/ZhangLabGT/LinRace
- name: R
  role: Environment for collapsing the true tree, generating the 100 random trees,
    computing CIDs, and writing cid_comparison_table.csv
- name: ggplot2
  role: Visualization of the CID comparison consistent with the Supplementary Fig.
    8b-f pattern
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:d59d4733a8952dd7c25deb2e8007aa3078c4429388c54cee01b1cf54ab6eb080
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/LinRace
  tools:
  - LinRace
  - R
  - TedSim
  - ggplot2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# clade-based-distance-cid-computation

## Summary

Evaluates a LinRace-reconstructed subtree against a collapsed ground-truth cell division tree using clade-based distance (CID), benchmarked against 100 randomly generated trees over the same tips to establish whether the reconstruction carries real topological signal (Supplementary Fig. 8b-f pattern).

## When to use

Use when assessing whether a lineage-reconstruction method recovers the true topology for cells sharing an identical barcode but belonging to different true clones (barcode homoplasy), via CID versus random-tree baselines.

## When NOT to use

- Evaluating whole-tree reconstruction accuracy against an uncollapsed ground truth — use normalized Robinson-Foulds distance instead (RF.dist with normalize = TRUE), as in the main simulation comparison
- Comparing inferred trees that all lack a ground-truth reference — use Nye similarity (NyeSimilarity with normalize = TRUE) rather than CID
- Barcode homoplasy is absent (each cell group's barcode maps to a single clone), so no subtree collapsing or homoplasy-resolution test is needed

## Inputs

- leaf-cell mutation matrix (muts_leaves) from TedSim simulation (SimulateCIFs with N_char=16, max_walk=8, p_a=0.8, n_CIF=30, n_diff=20, cif_step=0.5, Sigma=0.5, mu=0.05, non-zero p_d)
- true cell division tree (tree_ct, e.g. cifs[[4]] from the TedSim output)
- gene expression counts from CIF2Truecounts (ngenes=500, ge_prob=0.3)
- cell state labels / state lineages (cluster assignments)
- set of leaf cells sharing one identical barcode but from different true clones

## Outputs

- LinRace-reconstructed subtree for the homoplasy cell group
- collapsed reference (true) tree over identical-barcode cells
- 100 random trees over the same tips
- table of 101 CID values (LinRace + 100 random trees), written to cid_comparison_table.csv

## How to apply

First reconstruct the subtree of interest with LinRace's NJ_asym_Dif on the leaf-cell mutation matrix and expression counts (max_Iter = 500, lambda1 = 10, lambda2 = 1) using predefined state lineages. Collapse the true cell division tree (e.g., TedSim's tree_ct from SimulateCIFs output, cifs[[4]]) over identical-barcode cells to obtain the reference tree. Compute the clade-based distance (CID) of the reconstructed subtree against this collapsed reference, then generate 100 random trees over the same tip set and compute each one's CID against the same reference. The reconstruction is judged successful only if its CID is lower than every random-tree CID (101 values total: LinRace + 100 random), matching the finding that LinRace beat all 100 random trees by a large margin.

## Related tools

- **LinRace** (Reconstructs the homoplasy-cell subtree via NJ_asym_Dif combining the mutation matrix and expression counts; its tree is the test subject of the CID comparison) — https://github.com/ZhangLabGT/LinRace
- **TedSim** (Simulates the paired lineage-barcode / gene-expression dataset and provides the true cell division tree (tree_ct) used as the collapsed reference) — https://github.com/ZhangLabGT/LinRace
- **R** (Environment for collapsing the true tree, generating the 100 random trees, computing CIDs, and writing cid_comparison_table.csv)
- **ggplot2** (Visualization of the CID comparison consistent with the Supplementary Fig. 8b-f pattern)

## Examples

```
tree_linrace <- NJ_asym_Dif(as.data.frame(muts_leaves), cluster_id, expr, state_lineages, max_Iter = 500, lambda1 = 10, lambda2 = 1)
```

## Evaluation signals

- The LinRace subtree's CID is strictly lower than all 100 random-tree CIDs (101-value table is complete and non-degenerate)
- The reference tree is correctly collapsed: every identical-barcode leaf cell is merged so the group's true clone structure is the reference topology
- The 100 random trees span exactly the same tip set as the LinRace subtree (no tip mismatch inflating or deflating CIDs)
- The CID table serializes cleanly to cid_comparison_table.csv with one row per tree and matches the Supplementary Fig. 8b-f qualitative pattern
- Sanity-check the broader pipeline by verifying tree inputs are phylo objects as consumed by NJ_asym_Dif and tree comparison routines like RF.dist(..., normalize = TRUE)

## Limitations

- LinRace assumes irreversible cell state transitions; reversible transitions are not modeled, which could bias subtree topology in homoplasy groups
- LinRace assumes all ancestral cell states exist in current-day cells (most applicable to systems with asynchronous differentiation), constraining interpretation of reconstructed subtrees
- The CID evaluation is grounded in TedSim simulations with specific parameters (N_char=16, max_walk=8, p_a=0.8, mu=0.05); performance on real homoplasy groups may differ

## Evidence

- [methods] NJ_asym_Dif reconstruction call: "tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)"
- [methods] RF comparison against ground-truth tree: "print( RF.dist(TreeNJ_h, tree_ct, normalize = TRUE) )"
- [methods] TedSim simulation call: "cifs <- SimulateCIFs(ncells,phyla,p_a = p_a,n_CIF = n_cif,n_diff = n_diff,step = cif_step,p_d = p_d, Sigma = 0.5,mu = mu, N_char = N_char, max_walk = max_walk, SIF_res = returnlist, unif_on = FALSE)"
- [readme] LinRace method description: "LinRace is a method that integrates the lineage barcode and gene expression data using the asymmetric cell division model and infers cell lineage under a framework combining Neighbor Joining and"
- [supplementary] CID homoplasy finding: [evidence span withheld — anchor status: not_found]
