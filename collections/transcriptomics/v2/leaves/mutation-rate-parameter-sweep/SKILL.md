---
name: mutation-rate-parameter-sweep
description: Use when validating lineage-reconstruction tools on real expression data
  paired with simulated barcodes, and you need robustness evidence across mutation
  rates (e.g., mu in 0.025/0.05/0.075) and dropout settings.
when_to_use_negative:
- You have real, experimentally acquired lineage barcodes of known diversity — sweep
  simulation parameters like mu only apply when barcodes are simulated on a known
  ground-truth tree
- The ground-truth lineage tree is unknown, since RF distance, Nye similarity, and
  CID all require comparing against a reference tree
- Evaluating a method on a single fixed mutation rate without robustness claims —
  the sweep's value is demonstrating consistent outperformance across rates and dropout
  conditions
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0662
tools:
- name: TedSim
  role: Simulates cell identity fields and lineage barcodes on the true tree (SimulateCIFs)
    at swept mutation rates mu and dropout rates p_d
  repo: https://github.com/ZhangLabGT/LinRace
- name: LinRace
  role: Reconstructs lineage trees from simulated barcodes plus measured expression
    via NJ_asym_Dif (max_Iter=500, lambda1=10, lambda2=1); the method under evaluation
  repo: https://github.com/ZhangLabGT/LinRace
- name: phangorn
  role: Builds the barcode-only NJ baseline with phyDat-encoded unique allele states,
    dist.hamming, and nj (multi2di, uniform edge lengths)
- name: ape
  role: Computes normalized Robinson-Foulds distance between reconstructed and ground-truth
    trees (RF.dist, normalize=TRUE)
- name: TreeDist
  role: Computes normalized Nye similarity (NyeSimilarity, normalize=TRUE) as a tree-accuracy
    metric
- name: ggplot2
  role: Plots the method comparisons reproducing Fig. 3c and Supplementary Fig. 10
- name: DCLEAR
  role: Distance-based baseline method for lineage reconstruction on the simulated
    barcodes
- name: Cassiopeia
  role: Parsimony-based baseline method minimizing the number of mutations on the
    reconstructed tree
- name: LinTIMaT
  role: Combined-likelihood baseline using a local search framework on barcode and
    expression data
- name: Startle
  role: Baseline inferring cell division trees while enforcing the non-modifiability
    property of CRISPR/Cas9 mutations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:07a520a6ff4faf968e69119c7f0c7ebff31d7bdecc06fb5c63ae0dd7f5df948c
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
  - http://edamontology.org/topic_0662
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# mutation-rate-parameter-sweep

## Summary

Benchmark lineage-reconstruction methods by simulating lineage barcodes on a known true tree (TedSim SimulateCIFs) while sweeping CRISPR mutation rates and dropout conditions, then scoring every reconstructed tree against ground truth with RF distance, Nye similarity, and CID.

## When to use

Use when validating lineage-reconstruction tools on real expression data paired with simulated barcodes, and you need robustness evidence across mutation rates (e.g., mu in 0.025/0.05/0.075) and dropout settings.

## When NOT to use

- You have real, experimentally acquired lineage barcodes of known diversity — sweep simulation parameters like mu only apply when barcodes are simulated on a known ground-truth tree
- The ground-truth lineage tree is unknown, since RF distance, Nye similarity, and CID all require comparing against a reference tree
- Evaluating a method on a single fixed mutation rate without robustness claims — the sweep's value is demonstrating consistent outperformance across rates and dropout conditions

## Inputs

- Measured single-cell gene-expression matrix (e.g., 363 cells x 93 genes C. elegans L1 matrix from ref. 35)
- True lineage tree in Newick format (e.g., C. elegans L1 tree from CeLaVi)
- TedSim simulation parameters per condition (mu, N_char, p_a, n_CIF, Sigma, max_walk, p_d, ncells)
- 5 replicates per mutation-rate x dropout condition

## Outputs

- results data frame per method x mutation-rate x dropout condition containing RF distance, Nye similarity, and CID
- fig3c_l1_benchmark.csv with tabulated scores
- ggplot2 method-comparison plots reproducing Fig. 3c and Supplementary Fig. 10

## How to apply

Load the measured expression matrix (e.g., C. elegans L1, 363 cells x 93 genes) and the true lineage tree (Newick) from CeLaVi, then simulate barcodes on the true tree with TedSim's SimulateCIFs using fixed parameters (N_char=16, p_a=0.8, n_CIF=30, Sigma=0.5, max_walk=8) while sweeping mu in c(0.025, 0.05, 0.075) for ncells in c(1024, 2048), 5 replicates per condition, under both dropout-free (p_d=0) and dropout conditions. Reconstruct trees per replicate with LinRace (NJ_asym_Dif, max_Iter=500, lambda1=10, lambda2=1) using the simulated barcodes plus measured expression, and with each baseline (DCLEAR, Cassiopeia, LinTIMaT, Startle, and a barcode-only NJ from phyDat-encoded allele states via dist.hamming + nj with multi2di and uniform edge lengths). Score every reconstructed tree against the ground truth with normalized Robinson-Foulds (RF.dist, normalize=TRUE), normalized Nye similarity (NyeSimilarity, normalize=TRUE), and CID, assembling method x mutation-rate x dropout results. The rationale is that mutation rate directly controls barcode diversity and collision frequency, so conclusions about a method's advantage (e.g., LinRace's asymmetric-division model over barcode-only baselines) only hold if demonstrated across the swept range rather than a single rate.

## Related tools

- **TedSim** (Simulates cell identity fields and lineage barcodes on the true tree (SimulateCIFs) at swept mutation rates mu and dropout rates p_d) — https://github.com/ZhangLabGT/LinRace
- **LinRace** (Reconstructs lineage trees from simulated barcodes plus measured expression via NJ_asym_Dif (max_Iter=500, lambda1=10, lambda2=1); the method under evaluation) — https://github.com/ZhangLabGT/LinRace
- **phangorn** (Builds the barcode-only NJ baseline with phyDat-encoded unique allele states, dist.hamming, and nj (multi2di, uniform edge lengths))
- **ape** (Computes normalized Robinson-Foulds distance between reconstructed and ground-truth trees (RF.dist, normalize=TRUE))
- **TreeDist** (Computes normalized Nye similarity (NyeSimilarity, normalize=TRUE) as a tree-accuracy metric)
- **ggplot2** (Plots the method comparisons reproducing Fig. 3c and Supplementary Fig. 10)
- **DCLEAR** (Distance-based baseline method for lineage reconstruction on the simulated barcodes)
- **Cassiopeia** (Parsimony-based baseline method minimizing the number of mutations on the reconstructed tree)
- **LinTIMaT** (Combined-likelihood baseline using a local search framework on barcode and expression data)
- **Startle** (Baseline inferring cell division trees while enforcing the non-modifiability property of CRISPR/Cas9 mutations)

## Examples

```
cifs <- SimulateCIFs(ncells=2048, phyla=tree, p_a=0.8, n_CIF=30, p_d=0, Sigma=0.5, mu=0.05, N_char=16, max_walk=8)
```

## Evaluation signals

- The results CSV matches the deposited Fig. 3c Source Data values
- LinRace achieves the lowest RF distance across the swept mutation-rate (0.025, 0.05, 0.075) and dropout conditions
- All tree scores use normalized metrics (RF.dist normalize=TRUE, NyeSimilarity normalize=TRUE) so values are comparable across trees of different sizes
- Every condition (mu x ncells x dropout/no-dropout) contains exactly 5 replicates per method in the results data frame
- Baseline trees built via nj(dist_h) are made binary via multi2di with uniform edge lengths before scoring, otherwise RF/Nye comparisons are invalid

## Limitations

- Conclusions are conditional on the simulation model: TedSim-generated barcodes on the known tree, with fixed N_char=16, p_a=0.8, Sigma=0.5, max_walk=8; other barcode-design regimes are not covered
- LinRace assumes irreversible cell state transitions, so sweep results do not transfer to systems with reversible state transitions
- LinRace assumes all ancestral cell states exist in current-day cells, making results most applicable to systems with asynchronous differentiation
- Reproducibility depends on lineage barcode technology offering high diversity and coverage; low-quality barcodes may not support the swept mutation-rate conclusions

## Evidence

- [methods] SimulateCIFs with swept mu and p_d: "p_d = p_d, Sigma = 0.5,mu = mu, N_char = N_char, max_walk = max_walk"
- [methods] RF distance evaluation against ground truth: "print( RF.dist(TreeNJ_h, tree_ct, normalize = TRUE) )"
- [methods] Nye similarity metric used for comparison: "Nye_mutual[i,j] <- NyeSimilarity(tree_list[[i]], tree_list[[j]], normalize = TRUE)"
- [methods] Barcode-only NJ baseline construction: "dist_h <- dist.hamming(sim_data); Treenj = nj(dist_h)"
- [readme] LinRace method description grounding why it is the tool under sweep: "integrates the lineage barcode and gene expression data using the asymmetric cell division model and infers cell lineage under a framework"
