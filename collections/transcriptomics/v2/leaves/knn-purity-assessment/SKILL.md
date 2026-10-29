---
name: knn-purity-assessment
description: Use when benchmarking trajectory inference methods (e.g. Monocle, Slingshot,
  dynverse MST) on simulated continuous single-cell populations where gold-standard
  branch labels are available and ordering-only metrics are insufficient.
when_to_use_negative:
- The task is clustering or differential expression benchmarking rather than trajectory/branching
  assessment — the article evaluates those tasks with other metrics such as AUROC
- No gold-standard branch labels exist for the cells, so neighbor purity against true
  branches cannot be computed
- Data are UMI-based and the trajectory question concerns discrete populations rather
  than continuous EVF trajectories
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: Generates the continuous-population simulated data (SimulateTrueCounts, True2ObservedCounts,
    getTrajectoryGenes, Phyla5, gene_len_pool) that supply gold-standard branch labels
    for the purity computation
  repo: https://github.com/YosefLab/SymSim
- name: Monocle
  role: One of the trajectory inference methods whose inferred pseudotime is scored
    with cal_KNNP (version 2.6.4)
- name: Slingshot
  role: Trajectory inference method scored with cal_KNNP (version 0.99.12); outperforms
    Monocle and the MST method
- name: dynmethods
  role: Provides the wrapped minimum-spanning-tree trajectory inference method (dynwrap
    0.1.0) scored with cal_KNNP
  repo: https://github.com/dynverse/dynmethods
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:d3db59093b7f68404ef4c668db5f828fefb27ac34370a9e5384f2cf1b3fd9602
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - Monocle
  - Slingshot
  - SymSim
  - dynmethods
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# knn-purity-assessment

## Summary

Assess trajectory inference quality by computing k-nearest-neighbor purity (cal_KNNP): the fraction of each cell's k nearest neighbors in the inferred pseudotime embedding that belong to the same gold-standard branch as the cell. It complements Spearman pseudotime correlation by capturing branch-assignment fidelity rather than ordering fidelity.

## When to use

Use when benchmarking trajectory inference methods (e.g. Monocle, Slingshot, dynverse MST) on simulated continuous single-cell populations where gold-standard branch labels are available and ordering-only metrics are insufficient.

## When NOT to use

- The task is clustering or differential expression benchmarking rather than trajectory/branching assessment — the article evaluates those tasks with other metrics such as AUROC
- No gold-standard branch labels exist for the cells, so neighbor purity against true branches cannot be computed
- Data are UMI-based and the trajectory question concerns discrete populations rather than continuous EVF trajectories

## Inputs

- Observed single-cell count matrix from SymSim True2ObservedCounts (nonUMI protocol)
- Gold-standard cell metadata from getTrajectoryGenes: true pseudotime (root distance) and branch labels
- Inferred trajectory from Monocle, Slingshot, or the dynmethods MST wrapper
- Sigma/alpha grid settings (e.g. alpha=0.1, sigma=0.6) defining the benchmark conditions

## Outputs

- Per-cell k-nearest-neighbor purity values aggregated into a cal_KNNP score per method and per (Sigma, alpha) setting
- Comparison table/plot of cal_KNNP and cor_pseudotime across the Sigma/alpha grid for the three trajectory methods

## How to apply

Simulate the benchmark data with SymSim's SimulateTrueCounts using evf_type='continuous' (e.g. ncells_total=500, ngenes=500, nevf=20, n_de_evf=12, vary='s', phyla=Phyla5()) and convert to observed counts with True2ObservedCounts (protocol='nonUMI', gene lengths sampled from gene_len_pool). Extract gold-standard branch labels and root-distance pseudotime via getTrajectoryGenes, then run each trajectory method (Monocle 2.6.4, Slingshot 0.99.12, dynwrap/dynmethods v0.1.0 MST). For each inferred trajectory, compute cal_KNNP by checking, per cell, how many of its k nearest neighbors lie on the same true branch, and score alongside cor_pseudotime (Spearman correlation of inferred vs. true pseudotime). Sweep the noise axes — within-population variability (Sigma) and capture efficiency (alpha_mean) — and verify that KNN purity degrades with increasing Sigma at fixed alpha and improves with increasing alpha at fixed Sigma, as reported for Fig. 7a-b.

## Related tools

- **SymSim** (Generates the continuous-population simulated data (SimulateTrueCounts, True2ObservedCounts, getTrajectoryGenes, Phyla5, gene_len_pool) that supply gold-standard branch labels for the purity computation) — https://github.com/YosefLab/SymSim
- **Monocle** (One of the trajectory inference methods whose inferred pseudotime is scored with cal_KNNP (version 2.6.4))
- **Slingshot** (Trajectory inference method scored with cal_KNNP (version 0.99.12); outperforms Monocle and the MST method)
- **dynmethods** (Provides the wrapped minimum-spanning-tree trajectory inference method (dynwrap 0.1.0) scored with cal_KNNP) — https://github.com/dynverse/dynmethods

## Examples

```
phyla <- Phyla5(); sim <- SimulateTrueCounts(ncells_total=500, ngenes=500, nevf=20, n_de_evf=12, vary='s', sigma=0.6, evf_type='continuous', phyla=phyla, randseed=1)
```

## Evaluation signals

- cal_KNNP decreases monotonically as Sigma increases at fixed alpha (e.g. alpha=0.1)
- cal_KNNP increases as alpha increases at fixed Sigma (e.g. sigma=0.6)
- Slingshot yields higher KNN purity than Monocle and the dynmethods MST method across the grid
- Trends and magnitudes reproduce the reported Fig. 7a-b values when paired with cor_pseudotime
- Every cell receives a defined purity value with no NA outputs for cells on simulated branches

## Limitations

- KNN purity is a relative metric — it requires gold-standard branch labels and is only interpretable against simulated or known reference trajectories
- SymSim's simulation pipeline omits reverse transcription and library cleanup steps, so purity values may not transfer directly to real experimental data
- Results depend on simulation parameters; Supplementary Tables 2-5 with fitted kinetic/capture parameters are referenced but not present in the main text

## Evidence

- [intro] benchmarking trajectory inference with KNN purity: "We focus on the tasks of clustering, differential expression and trajectory inference, and test a number of methods under different simulation settings of biological separability and technical noise"
- [abstract] SymSim as benchmarking tool: "We demonstrate how SymSim can be used for benchmarking methods for clustering, differential expression and trajectory inference, and for examining the effects of various parameters on their"
- [methods] generating benchmark counts with gold-standard metadata: "SimulateTrueCounts( ) and True2ObservedCounts( ) are the two major functions to generate datasets."
- [methods] gold-standard trajectory metadata extraction: "Function getTrajectoryGenes( ) returns the simulated trajectory information of cells."
- [readme] dynmethods provides wrapped TI methods for comparison: [evidence span withheld — anchor status: not_found]
