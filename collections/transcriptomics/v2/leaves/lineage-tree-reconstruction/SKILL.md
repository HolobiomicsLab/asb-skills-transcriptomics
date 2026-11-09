---
name: lineage-tree-reconstruction
description: Use when you have paired single-cell lineage barcode data (character/allele
  matrix) and scRNA-seq expression for the same cells, and need a cell division tree
  with inferred ancestral cell states.
when_to_use_negative:
- Cell state transitions are reversible in your system — LinRace assumes irreversible
  state transitions
- Most ancestral cell states are absent from the scRNA-seq data (e.g., fast-changing
  or synchronous differentiation systems) — LinRace assumes current-day cells contain
  the states observed during development
- Only barcode data with no paired expression is available — a barcode-only method
  (DCLEAR, Cassiopeia, Startle, NJ on Hamming distance) is more appropriate
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3293
tools:
- name: LinRace
  role: 'Primary reconstruction method: joint NJ + maximum-likelihood inference of
    the lineage tree and ancestral states via NJ_asym_Dif'
  repo: https://github.com/ZhangLabGT/LinRace
- name: TedSim
  role: Simulates lineage barcodes (SimulateCIFs) and paired expression counts (CIF2Truecounts)
    on a ground-truth tree for benchmarking
- name: DCLEAR
  role: Distance-based baseline lineage reconstruction method for comparison
- name: Cassiopeia
  role: Parsimony-based baseline lineage reconstruction method for comparison
- name: Startle
  role: Barcode-only baseline enforcing non-modifiability of CRISPR/Cas9 mutations
- name: LinTIMaT
  role: Combined-likelihood integrated baseline using local search; also used as runtime
    comparison
- name: phangorn
  role: Barcode-only NJ baseline via dist.hamming + nj, and tree distances
- name: TreeDist
  role: Nye similarity computation for tree comparison
- name: ape
  role: RF.dist with normalization and internal node (Nnode) counting for evaluation
provenance:
  source_task_ids:
  - task_002
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:f6b1436c9c92c135c50ccb29e051dac56aa5383cd8410384f22acd4aa56cb9b0
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
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# lineage-tree-reconstruction

## Summary

Reconstruct single-cell lineage (cell division) trees by integrating lineage barcode character matrices with gene expression data under an asymmetric cell division model, as implemented by LinRace using joint Neighbor-Joining and maximum-likelihood heuristics.

## When to use

Use when you have paired single-cell lineage barcode data (character/allele matrix) and scRNA-seq expression for the same cells, and need a cell division tree with inferred ancestral cell states.

## When NOT to use

- Cell state transitions are reversible in your system — LinRace assumes irreversible state transitions
- Most ancestral cell states are absent from the scRNA-seq data (e.g., fast-changing or synchronous differentiation systems) — LinRace assumes current-day cells contain the states observed during development
- Only barcode data with no paired expression is available — a barcode-only method (DCLEAR, Cassiopeia, Startle, NJ on Hamming distance) is more appropriate

## Inputs

- Lineage barcode character matrix (cells x CRISPR target sites, e.g., cm_ZF1_F3 from scGESTALT)
- Paired gene expression matrix (e.g., ZF1_F3_DrImpute count matrix)
- Cluster/state labels derived from expression (k-means on top 50 PCs, centers = 20)
- State lineages from Slingshot trajectory inference on the UMAP layout
- Optional: ground-truth lineage tree (Newick) and TedSim-simulated barcodes for benchmarking

## Outputs

- Reconstructed lineage/cell division tree (phylo object / Newick) with cell tip labels
- Inferred ancestral cell states along the tree
- Benchmark score table per method x mutation rate x dropout condition (e.g., fig3c_l1_benchmark.csv) with RF distance, Nye similarity, and CID

## How to apply

Build a character matrix from the barcode data (e.g., cm_ZF1_F3 for scGESTALT zebrafish), compute PCA on the scaled expression matrix and cluster cells with k-means (centers = 20) on the top 50 PCs, embed with UMAP (min_dist = 0.3), and run Slingshot on the UMAP layout to derive state lineages. Call LinRace's NJ_asym_Dif on the character matrix, cluster labels, expression, and state lineages with max_Iter = 500 (and, for benchmarks, lambda1 = 10, lambda2 = 1), then restore original cell tip labels. For simulated ground-truth testing, generate barcodes on a known tree with TedSim SimulateCIFs (e.g., N_char = 16, p_a = 0.8, n_CIF = 30, Sigma = 0.5, max_walk = 8, sweeping mu = 0.025/0.05/0.075) and expression counts with CIF2Truecounts (ngenes = 500, ge_prob = 0.3). Score reconstructions against the ground truth using normalized Robinson-Foulds distance (RF.dist, normalize = TRUE), normalized Nye similarity, and CID. The rationale is that the asymmetric division model explains real-data state divergence/convergence and enables ancestral state inference, yielding more accurate trees than barcode-only baselines (DCLEAR, Cassiopeia, Startle) or LinTIMaT.

## Related tools

- **LinRace** (Primary reconstruction method: joint NJ + maximum-likelihood inference of the lineage tree and ancestral states via NJ_asym_Dif) — https://github.com/ZhangLabGT/LinRace
- **TedSim** (Simulates lineage barcodes (SimulateCIFs) and paired expression counts (CIF2Truecounts) on a ground-truth tree for benchmarking)
- **DCLEAR** (Distance-based baseline lineage reconstruction method for comparison)
- **Cassiopeia** (Parsimony-based baseline lineage reconstruction method for comparison)
- **Startle** (Barcode-only baseline enforcing non-modifiability of CRISPR/Cas9 mutations)
- **LinTIMaT** (Combined-likelihood integrated baseline using local search; also used as runtime comparison)
- **phangorn** (Barcode-only NJ baseline via dist.hamming + nj, and tree distances)
- **TreeDist** (Nye similarity computation for tree comparison)
- **ape** (RF.dist with normalization and internal node (Nnode) counting for evaluation)

## Examples

```
tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF), cluster_id, expr_ZF, state_lineages, max_Iter = 500)
```

## Evaluation signals

- Normalized Robinson-Foulds distance (RF.dist, normalize = TRUE) against the ground-truth tree is lowest compared to DCLEAR, Cassiopeia, Startle, LinTIMaT, and barcode-only NJ across swept mutation rates (mu = 0.025, 0.05, 0.075), with and without dropouts
- Results table matches deposited Source Data values (e.g., fig3c_l1_benchmark.csv) for the C. elegans L1 benchmark
- Reconstructed tree has greater maximum root-to-tip depth and more internal nodes (ape Nnode) than Cassiopeia and LinTIMaT trees on ZF1-F3, consistent with Fig. 4b
- NJ baseline sanity check: RF.dist(TreeNJ_h, tree_ct, normalize = TRUE) computed from dist.hamming on phyDat-encoded unique allele states
- LinRace runtime is lower than LinTIMaT's at the largest simulated cell count (1024/2048 cells), consistent with Supplementary Fig. 9

## Limitations

- LinRace assumes irreversible cell state transitions; reversible transitions are not modeled
- Assumes all cell states during development exist in current-day cells; ancestral states are restricted to cell states present in the scRNA-seq data, so fast-changing systems may have uncaptured intermediate states
- Most applicable to systems with asynchronous differentiation; results also depend on lineage barcode technology with high diversity and coverage
- Raw matrix.mtx.gz files for the zebrafish vignette are not uploaded due to GitHub LFS quota issues; datasets must be obtained from GSE117542

## Evidence

- [abstract] Here, we present LinRace (Lineage Reconstruction with asymmetric cell division model), which integrates lineage barcode and gene expression data: "Here, we present LinRace (Lineage Reconstruction with asymmetric cell division model), which integrates lineage barcode and gene expression data"
- [discussion] LinRace adopts the asymmetric division model, which explains phenomena in real data, increases reconstruction accuracy, and allows for the estimation of ancestral cell states.: "LinRace adopts the asymmetric division model, which explains phenomena in real data, increases reconstruction accuracy, and allows for the estimation of ancestral cell states."
- [methods] tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500): "tree_linrace <- NJ_asym_Dif(as.data.frame(cm_ZF),cluster_id,expr_ZF,state_lineages,max_Iter = 500)"
- [discussion] LinRace assumes that cell state transitions are irreversible, which is generally true under homeostatic conditions. However, reversible state transitions are also known to exist.: "LinRace assumes that cell state transitions are irreversible, which is generally true under homeostatic conditions. However, reversible state transitions are also known to exist."
- [readme] LinRace is a method that integrates the lineage barcode and gene expression data using the asymmetric cell division model and infers cell lineage: "LinRace is a method that integrates the lineage barcode and gene expression data using the asymmetric cell division model and infers cell lineage"
