---
name: dropout-perturbation-analysis
description: Use when simulating CRISPR lineage barcodes (e.g., TedSim SimulateCIFs)
  and you need to measure how dropout events degrade barcode quality — via Qr/Qh —
  or how they change relative performance of reconstruction methods like LinRace,
  DCLEAR, or Cassiopeia.
when_to_use_negative:
- Barcode data contains no simulated or known ground-truth lineage tree, so Qr/Qh
  and RF/Nye/CID evaluation against truth cannot be computed
- The expression data are being analyzed for cell-state dynamics rather than barcode
  data-quality benchmarking — this skill is a perturbation/benchmark workflow, not
  a biological discovery analysis
- Dropout is negligible (p_d=0 and empirically absent) and the question is only about
  absolute mutation-rate effects without a paired no-dropout control
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: TedSim
  role: Simulates ground-truth lineage trees and barcodes with tunable mutation rate
    mu and dropout probability p_d for the paired conditions
  repo: github.com/ZhangLabGT/LinRace
- name: LinRace
  role: Reconstructs lineage trees from barcodes plus expression under dropout and
    no-dropout conditions (NJ_asym_Dif) for benchmark comparison
  repo: github.com/ZhangLabGT/LinRace
- name: DCLEAR
  role: Distance-based baseline reconstruction method evaluated across dropout conditions
- name: Cassiopeia
  role: Parsimony-based baseline reconstruction method evaluated across dropout conditions
- name: LinTIMaT
  role: Likelihood-based baseline reconstruction method evaluated across dropout conditions
- name: Startle
  role: Baseline method enforcing non-modifiability of CRISPR/Cas9 mutations, evaluated
    across dropout conditions
- name: phangorn
  role: Provides dist.hamming and nj for the barcode-only NJ baseline tree reconstruction
- name: TreeDist
  role: Provides NyeSimilarity (normalize=TRUE) for scoring reconstructed trees against
    ground truth
- name: ape
  role: Provides RF.dist (normalize=TRUE) and multi2di for tree comparison
- name: ggplot2
  role: Plots the method-by-mutation-rate comparison under dropout vs. no-dropout
    conditions
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44173-3
    title: 'LinRace: cell division history reconstruction of single cells using paired
      lineage barcode and gene expression data'
  source_content_hash: sha256:4c3eed85f639a84b1523a1b8e3775e5ed81f4ac473d7bbb761e1681ef52f3ee9
schema_version: 0.2.0
metadata:
  license_tier: open
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# dropout-perturbation-analysis

## Summary

A benchmark-design skill for quantifying how lineage barcode dropout events perturb data quality and lineage-reconstruction accuracy, by sweeping dropout probability (p_d) alongside mutation rate in TedSim simulations and tracking both downstream method performance (RF distance, Nye similarity, CID) and barcode-quality metrics (reconstruction potential Qr, homoplasy edge ratio Qh).

## When to use

Use when simulating CRISPR lineage barcodes (e.g., TedSim SimulateCIFs) and you need to measure how dropout events degrade barcode quality — via Qr/Qh — or how they change relative performance of reconstruction methods like LinRace, DCLEAR, or Cassiopeia.

## When NOT to use

- Barcode data contains no simulated or known ground-truth lineage tree, so Qr/Qh and RF/Nye/CID evaluation against truth cannot be computed
- The expression data are being analyzed for cell-state dynamics rather than barcode data-quality benchmarking — this skill is a perturbation/benchmark workflow, not a biological discovery analysis
- Dropout is negligible (p_d=0 and empirically absent) and the question is only about absolute mutation-rate effects without a paired no-dropout control

## Inputs

- TedSim SimulateCIFs configuration (ncells, N_char, p_a, n_CIF, Sigma, max_walk, mu grid, p_d)
- Ground-truth lineage tree (tree_ct, Newick with cell_* tip labels)
- Leaf barcode matrix (muts_leaves, first ncells rows of simulated alleles)
- Measured gene-expression matrix (e.g., C. elegans L1, 363 cells x 93 genes) for integrated reconstruction runs

## Outputs

- Qr/Qh table per mutation-rate x dropout condition (qr_qh_vs_mutation_rate.csv)
- Reconstruction scores per method x mutation-rate x dropout (fig3c_l1_benchmark.csv) with RF distance, Nye similarity, CID
- Comparison plots reproducing Fig. 3c and Supplementary Fig. 10 (ggplot2)

## How to apply

Simulate ground-truth lineage trees and ancestral barcodes with TedSim's SimulateCIFs (e.g., ncells=1024, N_char=16, p_a=0.8, n_CIF=30, Sigma=0.5, max_walk=8) twice per mutation rate: once with p_d=0 (dropout-free) and once with dropout events enabled, sweeping mu over a grid such as c(0.025, 0.05, 0.075) with replicates per condition. For each dataset, extract the ground-truth tree (tree_ct with cell_* tips) and the leaf barcode matrix (muts_leaves, first ncells rows), then compute the reconstruction potential Qr (fraction of recoverable mutated target sites, Eq. 13) and homoplasy edge ratio Qh (Eq. 14) per mutation-rate x dropout combination. In parallel, reconstruct trees with LinRace (NJ_asym_Dif, max_Iter=500, lambda1=10, lambda2=1) and baselines (DCLEAR, Cassiopeia, LinTIMaT, Startle, barcode-only NJ via dist.hamming + nj), scoring each against the truth with normalized RF.dist, NyeSimilarity, and CID. Tabulate scores and Qr/Qh into a per-method x mutation-rate x dropout CSV (e.g., fig3c_l1_benchmark.csv, qr_qh_vs_mutation_rate.csv) and compare dropout vs. no-dropout conditions. The rationale is that the paper found homoplasy edge ratio reaches ~0.75 under dropouts and tracks method performance trends, so Qh serves as a barcode data-quality proxy.

## Related tools

- **TedSim** (Simulates ground-truth lineage trees and barcodes with tunable mutation rate mu and dropout probability p_d for the paired conditions) — github.com/ZhangLabGT/LinRace
- **LinRace** (Reconstructs lineage trees from barcodes plus expression under dropout and no-dropout conditions (NJ_asym_Dif) for benchmark comparison) — github.com/ZhangLabGT/LinRace
- **DCLEAR** (Distance-based baseline reconstruction method evaluated across dropout conditions)
- **Cassiopeia** (Parsimony-based baseline reconstruction method evaluated across dropout conditions)
- **LinTIMaT** (Likelihood-based baseline reconstruction method evaluated across dropout conditions)
- **Startle** (Baseline method enforcing non-modifiability of CRISPR/Cas9 mutations, evaluated across dropout conditions)
- **phangorn** (Provides dist.hamming and nj for the barcode-only NJ baseline tree reconstruction)
- **TreeDist** (Provides NyeSimilarity (normalize=TRUE) for scoring reconstructed trees against ground truth)
- **ape** (Provides RF.dist (normalize=TRUE) and multi2di for tree comparison)
- **ggplot2** (Plots the method-by-mutation-rate comparison under dropout vs. no-dropout conditions)

## Examples

```
cifs <- SimulateCIFs(ncells=1024, phyla, p_a=0.8, n_CIF=30, n_diff=20, p_d=p_d, Sigma=0.5, mu=mu, N_char=16, max_walk=8)
```

## Evaluation signals

- Homoplasy edge ratio Qh reaches approximately 0.75 on TedSim-simulated data (1024 cells, 16 characters) when dropouts are present
- Qh varies with mutation rate following a similar trend to lineage-reconstruction method performance
- LinRace achieves the lowest normalized RF distance across swept mutation-rate and dropout conditions, matching deposited Fig. 3c Source Data values
- Every method x mutation-rate x dropout cell has scores for all three metrics (RF, Nye, CID) with 5 replicates per condition
- Dropout conditions consistently show worse RF distance than the matched p_d=0 conditions (cf. Fig. 3c vs. Supplementary Fig. 10)

## Limitations

- The homoplasy/dropout analysis depends on TedSim simulation assumptions; results depend on future improvements in lineage barcode technology (high diversity and coverage) not available in the current workflow
- Software versions are not stated for any of the R packages used, hindering exact reproduction
- Dropout perturbation results are reported for simulated barcodes grafted onto real expression data (C. elegans L1); conclusions may not transfer to fully observed real barcode dropouts

## Evidence

- [results] On the C. elegans dataset with simulated barcodes, LinRace consistently outperforms state-of-the-art methods for varying mutation rates, both with dropouts (Fig. 3c) and without dropouts: [evidence span withheld — anchor status: not_found]
- [results] the homoplasy edge ratio reaches as high as 0.75 when dropouts are present, and it varies with mutation rate following a similar trend to lineage reconstruction method performance: [evidence span withheld — anchor status: not_found]
- [methods] compute the reconstruction potential Qr (fraction of recoverable mutated target sites) and homoplasy edge ratio Qh per the paper's Eq. 13 and Eq. 14: [evidence span withheld — anchor status: not_found]
- [methods] SimulateCIFs(ncells,phyla,p_a = p_a,n_CIF = n_cif,n_diff = n_diff,step = cif_step,p_d = p_d, Sigma = 0.5,mu = mu: "SimulateCIFs(ncells,phyla,p_a = p_a,n_CIF = n_cif,n_diff = n_diff,step = cif_step,p_d = p_d, Sigma = 0.5,mu = mu"
- [discussion] it is also important that the data quality is improved with advances in technology, e.g., to create lineage barcodes with high diversity and coverage: "it is also important that the data quality is improved with advances in technology, e.g., to create lineage barcodes with high diversity and coverage"
