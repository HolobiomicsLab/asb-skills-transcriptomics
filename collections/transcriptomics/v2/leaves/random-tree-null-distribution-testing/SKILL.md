---
name: random-tree-null-distribution-testing
description: Use when you must judge whether a reconstructed phylogeny (e.g., a LinRace
  subtree for cells sharing an identical barcode but from different clones) carries
  real topological signal, by comparing its distance to a reference against random
  trees over the same tip set.
when_to_use_negative:
- The reference tree has no cells that need collapsing (all leaves uniquely resolvable)
  — compare directly with RF.dist or NyeSimilarity normalization instead of a random-tree
  null
- Only one or a few random trees are feasible to generate — the null distribution
  is too sparse to support the 'lower than every random tree' criterion
edam_operation: http://edamontology.org/operation_0557
edam_topics:
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_2229
tools:
- name: LinRace
  role: Reconstructs the candidate subtree via NJ_asym_Dif on barcode mutations plus
    expression counts (max_Iter = 500, lambda1 = 10, lambda2 = 1)
  repo: https://github.com/ZhangLabGT/LinRace
- name: TedSim
  role: Simulates the paired lineage-barcode and gene-expression ground truth (SimulateCIFs,
    CIF2Truecounts) providing tree_ct, muts_leaves, and cell state labels for the
    homoplasy group
- name: R
  role: Runtime environment for tree manipulation, random-tree generation, and CID
    computation
- name: ggplot2
  role: Visualization of the CID comparison distribution (Supplementary Fig. 8b-f
    pattern)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:19fc91af47874b66dc1a5d1e6cb9754f539c60902748e71b6e8e5df43a845430
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/LinRace
  tools:
  - LinRace
  - R
  - TedSim
  - ggplot2
  edam_operation: http://edamontology.org/operation_0557
  edam_topics:
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# Random-tree null distribution testing

## Summary

Validates a lineage-tree reconstruction by computing a tree-distance metric (here, CID — common intervals distance) between the reconstructed tree and a reference tree, and comparing it against the null distribution obtained from 100 randomly generated trees over the same tips. A reconstructed tree is considered informative only if its distance to the reference falls below every (or nearly all) random-tree distances.

## When to use

Use when you must judge whether a reconstructed phylogeny (e.g., a LinRace subtree for cells sharing an identical barcode but from different clones) carries real topological signal, by comparing its distance to a reference against random trees over the same tip set.

## When NOT to use

- The reference tree has no cells that need collapsing (all leaves uniquely resolvable) — compare directly with RF.dist or NyeSimilarity normalization instead of a random-tree null
- Only one or a few random trees are feasible to generate — the null distribution is too sparse to support the 'lower than every random tree' criterion

## Inputs

- Reconstructed tree over a fixed tip set (e.g., LinRace subtree from NJ_asym_Dif on muts_leaves and expression counts)
- Reference/true tree collapsed over identical-barcode leaf cells (tree_ct from TedSim SimulateCIFs output)
- Tip labels of the cell group under test (leaf cells sharing one barcode from different true clones)
- Number of null replicates (100 random trees) over the same tips

## Outputs

- Table of 101 CID values (reconstructed tree + 100 random trees) written to cid_comparison_table.csv
- Binary verdict: reconstructed-tree CID < minimum random-tree CID (null rejected) or not

## How to apply

First reconstruct the tree under test — e.g., run LinRace's NJ_asym_Dif on the mutation matrix (muts_leaves) and expression counts of the target cell group with max_Iter = 500, lambda1 = 10, lambda2 = 1 and predefined state lineages. Next, prepare the reference: collapse the true cell division tree (tree_ct) over leaf cells that are indistinguishable (e.g., carry identical barcodes), so the reference reflects only resolvable structure. Compute the CID of the reconstructed tree against this collapsed reference, then generate 100 random trees over the same tip labels and compute CID of each against the same reference. Write the 101 CID values to a table (e.g., cid_comparison_table.csv); the reconstruction passes the null test only if its CID is lower than every random-tree CID (matching the Supplementary Fig. 8b-f pattern where LinRace outperformed all 100 random trees by a large margin).

## Related tools

- **LinRace** (Reconstructs the candidate subtree via NJ_asym_Dif on barcode mutations plus expression counts (max_Iter = 500, lambda1 = 10, lambda2 = 1)) — https://github.com/ZhangLabGT/LinRace
- **TedSim** (Simulates the paired lineage-barcode and gene-expression ground truth (SimulateCIFs, CIF2Truecounts) providing tree_ct, muts_leaves, and cell state labels for the homoplasy group)
- **R** (Runtime environment for tree manipulation, random-tree generation, and CID computation)
- **ggplot2** (Visualization of the CID comparison distribution (Supplementary Fig. 8b-f pattern))

## Examples

```
tree_linrace <- NJ_asym_Dif(as.data.frame(muts_leaves), cluster_id, expr, state_lineages, max_Iter = 500)
```

## Evaluation signals

- The reconstructed tree's CID is lower than every one of the 100 random-tree CIDs (LinRace beat all 100 random trees by a large margin in the article)
- cid_comparison_table.csv contains exactly 101 CID values computed against the same collapsed true tree
- The collapsed reference tree reflects only barcode-resolvable structure — identical-barcode cells are merged before distance computation
- All random trees span exactly the same tip labels as the reconstructed subtree
- If visualized, the LinRace CID point sits left of the entire random-tree distribution, matching Supplementary Fig. 8b-f

## Limitations

- The null is tip-set-specific: results only hold for the particular homoplasy group (identical barcode, different true clones) and collapsed reference; they do not generalize to other subtrees
- Random trees assume a uniform null over topologies; if the true tree has unusual balance properties, the null may be weak
- CID must be computed against the same collapsed true tree for both reconstructed and random trees, otherwise values are not comparable

## Evidence

- [results] On a TedSim-simulated group of cells sharing one barcode but from different true clones, the CID between the true tree and the LinRace reconstructed tree showed LinRace is superior to all 100 randomly generated trees by a large margin.: [evidence span withheld — anchor status: not_found]
- [methods] Article methods on tree building with NJ_asym_Dif.: "tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)"
- [methods] Article methods on TedSim simulation used to generate the ground truth for the homoplasy case.: "cifs <- SimulateCIFs(ncells,phyla,p_a = p_a,n_CIF = n_cif,n_diff = n_diff,step = cif_step,p_d = p_d, Sigma = 0.5,mu = mu, N_char = N_char, max_walk = max_walk, SIF_res = returnlist, unif_on = FALSE)"
- [readme] README describing LinRace's integration framework supporting the reconstruction step under test.: "LinRace is a method that integrates the lineage barcode and gene expression data using the asymmetric cell division model and infers cell lineage under a framework combining Neighbor Joining and"
- [methods] Article methods on alternative normalized tree-distance evaluation.: "print( RF.dist(TreeNJ_h, tree_ct, normalize = TRUE) )"
