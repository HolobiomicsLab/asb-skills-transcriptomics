---
name: paired-barcode-expression-lineage-tree-reconstruction
description: Use when you have paired single-cell lineage-barcode mutation matrices
  and gene expression counts and need to infer a cell division tree, or to resolve
  barcode homoplasy where identical barcodes span different true clones.
when_to_use_negative:
- Systems with reversible cell state transitions or synchronized differentiation —
  LinRace assumes irreversible state transitions and that all ancestral states exist
  in current-day cells
- Lineage barcodes with low diversity/coverage — poor barcode quality limits tree
  reconstruction regardless of expression integration
- Barcode-only reconstruction tasks where expression data are unavailable (barcode-only
  methods like DCLEAR or Cassiopeia may suffice)
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
tools:
- name: LinRace
  role: 'Core method: NJ_asym_Dif reconstructs the cell division tree from paired
    barcode mutations and expression under the asymmetric division model (max_Iter=500)'
  repo: https://github.com/ZhangLabGT/LinRace
- name: TedSim
  role: Simulator generating paired lineage-barcode and gene-expression ground-truth
    datasets (SimulateCIFs, CIF2Truecounts) for benchmarking
- name: R
  role: Implementation environment for the LinRace package and benchmarking scripts
- name: ggplot2
  role: Visualization of reconstruction results (e.g., Supplementary Fig. 8b-f CID
    comparisons)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:02f412c5d5a0897b2f31df4812b7c9f50faed41d5524e893fdf43700031a5ece
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZhangLabGT/LinRace
  tools:
  - LinRace
  - R
  - TedSim
  - ggplot2
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# paired-barcode-expression-lineage-tree-reconstruction

## Summary

Reconstruct single-cell division (lineage) trees by integrating CRISPR lineage-barcode mutation data with scRNA-seq gene expression using LinRace's asymmetric cell division model, combining Neighbor-Joining and maximum-likelihood heuristics.

## When to use

Use when you have paired single-cell lineage-barcode mutation matrices and gene expression counts and need to infer a cell division tree, or to resolve barcode homoplasy where identical barcodes span different true clones.

## When NOT to use

- Systems with reversible cell state transitions or synchronized differentiation — LinRace assumes irreversible state transitions and that all ancestral states exist in current-day cells
- Lineage barcodes with low diversity/coverage — poor barcode quality limits tree reconstruction regardless of expression integration
- Barcode-only reconstruction tasks where expression data are unavailable (barcode-only methods like DCLEAR or Cassiopeia may suffice)

## Inputs

- leaf-cell lineage-barcode mutation matrix (muts_leaves, e.g., from TedSim SimulateCIFs output or CRISPR barcode data)
- gene expression count matrix (e.g., TedSim CIF2Truecounts output with ngenes=500, ge_prob=0.3, or zebrafish scGESTALT counts)
- cell state/cluster labels (e.g., kmeans clusters on top 50 PCs)
- predefined state lineage (cell-state tree / state_lineages)
- optional: true cell division tree (tree_ct) for benchmarking

## Outputs

- reconstructed cell division tree (phylo object) with inferred ancestral cell states
- CID / Robinson-Founds / Nye similarity comparison tables against reference or random trees (e.g., cid_comparison_table.csv)

## How to apply

Run LinRace's NJ_asym_Dif on the leaf-cell mutation matrix (e.g., from TedSim SimulateCIFs with N_char=16, max_walk=8, p_a=0.8, mu=0.05) together with the expression count matrix and cell-state labels (e.g., kmeans clusters on the top 50 PCs), using predefined state lineages and max_Iter = 500 (with lambda1 = 10, lambda2 = 1 in the homoplasy analysis). The method jointly optimizes barcode-mutation likelihood and expression-based state-transition likelihood under an asymmetric division model, via preprocessing (MutsTransform, distH), likelihood calculation (LikelihoodCal), ancestral state inference (AncesInfer), and local tree search (TreeLC2/TreeLC3). Evaluate the resulting tree against ground truth using CID (for homoplasy subtrees), Robinson-Foulds, or Nye similarity; in the homoplasy benchmark the LinRace subtree CID should beat all 100 random trees.

## Related tools

- **LinRace** (Core method: NJ_asym_Dif reconstructs the cell division tree from paired barcode mutations and expression under the asymmetric division model (max_Iter=500)) — https://github.com/ZhangLabGT/LinRace
- **TedSim** (Simulator generating paired lineage-barcode and gene-expression ground-truth datasets (SimulateCIFs, CIF2Truecounts) for benchmarking)
- **R** (Implementation environment for the LinRace package and benchmarking scripts)
- **ggplot2** (Visualization of reconstruction results (e.g., Supplementary Fig. 8b-f CID comparisons))

## Examples

```
tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)
```

## Evaluation signals

- CID of the reconstructed subtree against the collapsed true tree is lower than all 100 random-tree CIDs (homoplasy benchmark, Supplementary Fig. 8b-f)
- Robinson-Foulds distance of the LinRace tree to the ground-truth tree (RF.dist with normalize=TRUE) is lower than barcode-only NJ baselines
- Pairwise Nye similarity matrix across reconstructed trees (NyeSimilarity with normalize=TRUE) shows consistency of repeated runs
- Reconstructed tree respects the asymmetric division model: state divergences/convergences are consistent with the predefined state_lineages
- 101-row cid_comparison_table.csv contains LinRace + 100 random-tree CID values with LinRace ranked lowest

## Limitations

- LinRace assumes irreversible cell state transitions; reversible transitions are not modeled
- Assumes all cell states during development exist in current-day cells; ancestral states are restricted to cell states present in the scRNA-seq data, so fast-changing systems may have uncaptured intermediate states
- Most applicable to systems with asynchronous differentiation
- Results depend on future improvements in lineage barcode technology (high diversity and coverage)
- Software versions for the R packages are not stated in the methods

## Evidence

- [abstract] integrates lineage barcode and gene expression data using asymmetric cell division model and infers cell lineages and ancestral cell states using Neighbor-Joining and maximum-likelihood: "which integrates lineage barcode and gene expression data using asymmetric cell division model and infers cell lineages and ancestral cell states using Neighbor-Joining and maximum-likelihood"
- [methods] NJ_asym_Dif call with max_Iter=500: "tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)"
- [methods] TedSim simulation parameters for paired barcode/expression data: "cifs <- SimulateCIFs(ncells,phyla,p_a = p_a,n_CIF = n_cif,n_diff = n_diff,step = cif_step,p_d = p_d, Sigma = 0.5,mu = mu, N_char = N_char, max_walk = max_walk, SIF_res = returnlist, unif_on = FALSE)"
- [discussion] Limitation: irreversible state transitions assumption: "LinRace assumes that cell state transitions are irreversible, which is generally true under homeostatic conditions. However, reversible state transitions are also known to exist."
- [readme] README installation command: [evidence span withheld — anchor status: external_doc]
