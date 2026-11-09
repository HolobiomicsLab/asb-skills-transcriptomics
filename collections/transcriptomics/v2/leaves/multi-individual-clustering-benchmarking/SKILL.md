---
name: multi-individual-clustering-benchmarking
description: Use when you must compare clustering accuracy across methods on multi-individual
  droplet scRNA-seq simulations (e.g., varying number of individuals, sigma_ik^2 heterogeneity,
  or Splatter group/batch-effect levels) and quantify results with ARI.
when_to_use_negative:
- Input is a single individual's data or a single matrix — BAMMSC then falls back
  to DIMMSC, so multi-individual benchmarking claims do not apply
- Input data are already normalized or reduced features rather than raw UMI counts
  — BAMM-SC is designed to take raw count data as input
- The goal is trajectory/developmental-stage analysis, which the authors explicitly
  defer to future papers rather than address with this clustering benchmark
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BAMMSC
  role: Bayesian mixture model under evaluation; clusters multi-individual UMI count
    matrices simultaneously
  repo: github.com/lichen-lab/BAMMSC
- name: mclust
  role: Provides adjustedRandIndex to score predicted memberships against true labels
- name: dirmult
  role: Supports sampling from the Dirichlet multinomial generative model for simulations
- name: Splatter
  role: Independent simulation tool for mis-specification robustness tests across
    group and batch effect levels
- name: SIMLR
  role: Competing unsupervised clustering method run on the same simulated count matrices
- name: CellTree
  role: Competing unsupervised clustering method
- name: SC3
  role: Competing unsupervised clustering method
- name: TSCAN
  role: Competing unsupervised clustering method
- name: DIMM-SC
  role: Competing Dirichlet multinomial method for single-individual or pooled clustering
- name: MetaNeighbor
  role: Competing supervised method for cross-dataset cell-type assessment
- name: scran
  role: Implements MNN-based batch correction used as a competing approach
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:6fbf2358fc0478e99ac9319d31cff002a76c5a2ab588877d82d951efd70f82f2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BAMM-SC
  - BAMMSC
  - CellTree
  - DIMM-SC
  - MetaNeighbor
  - SC3
  - SIMLR
  - Splatter
  - TSCAN
  - dirmult
  - mclust
  - scran
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# multi-individual-clustering-benchmarking

## Summary

Benchmark BAMM-SC, a Bayesian hierarchical Dirichlet multinomial mixture model for clustering droplet-based scRNA-seq data from multiple individuals, against competing clustering methods on simulated multi-individual UMI count data, scoring clustering accuracy with the adjusted Rand index (ARI) via mclust.

## When to use

Use when you must compare clustering accuracy across methods on multi-individual droplet scRNA-seq simulations (e.g., varying number of individuals, sigma_ik^2 heterogeneity, or Splatter group/batch-effect levels) and quantify results with ARI.

## When NOT to use

- Input is a single individual's data or a single matrix — BAMMSC then falls back to DIMMSC, so multi-individual benchmarking claims do not apply
- Input data are already normalized or reduced features rather than raw UMI counts — BAMM-SC is designed to take raw count data as input
- The goal is trajectory/developmental-stage analysis, which the authors explicitly defer to future papers rather than address with this clustering benchmark

## Inputs

- List X of per-individual UMI count matrices (rows = genes, columns = cells; gene counts equal across individuals, cell numbers may differ)
- Simulated datasets generated from the Bayesian hierarchical Dirichlet multinomial mixture model or from Splatter (splatEstimate parameters from 10x Genomics purified B-cells)
- True cluster number K (default K=4 in the simulations)
- True simulated cell cluster labels for ARI computation

## Outputs

- Per-replicate adjustedRandIndex values for each method
- Averaged ARI table per scenario-by-method cell (e.g., simulation_ari_results.csv)
- Qualitative comparison reproducing Fig. 2 (ARI vs number of individuals) and Fig. 3 (Splatter mis-specification) patterns

## How to apply

Simulate multi-individual UMI count matrices either from the paper's Bayesian hierarchical Dirichlet multinomial mixture model (sweeping mean sigma_ik^2 heterogeneity and number of individuals, 100 replicates per scenario) or with Splatter after splatEstimate on the 10x Genomics purified B-cells dataset spanning low/medium/high group and batch effect levels. Run BAMMSC(X, K) on each dataset as a list of per-individual UMI count matrices with K set to the true cluster number and default MCMC settings (nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10). Apply the competing scRNA-seq methods SIMLR, CellTree, SC3, TSCAN, and DIMM-SC (plus MetaNeighbor and MNN/scran for supervised/batch-correction comparisons) to the same simulated counts with the same true K. Score each method's membership vectors against true labels with mclust::adjustedRandIndex per replicate, average ARI over the 100 replicates within each scenario-by-method cell, and write the table (e.g., simulation_ari_results.csv). The rationale is that BAMM-SC accounts for data heterogeneity and batch effect among individuals, so its advantage should grow as individuals are added while single-pool methods stall or decline.

## Related tools

- **BAMMSC** (Bayesian mixture model under evaluation; clusters multi-individual UMI count matrices simultaneously) — github.com/lichen-lab/BAMMSC
- **mclust** (Provides adjustedRandIndex to score predicted memberships against true labels)
- **dirmult** (Supports sampling from the Dirichlet multinomial generative model for simulations)
- **Splatter** (Independent simulation tool for mis-specification robustness tests across group and batch effect levels)
- **SIMLR** (Competing unsupervised clustering method run on the same simulated count matrices)
- **CellTree** (Competing unsupervised clustering method)
- **SC3** (Competing unsupervised clustering method)
- **TSCAN** (Competing unsupervised clustering method)
- **DIMM-SC** (Competing Dirichlet multinomial method for single-individual or pooled clustering)
- **MetaNeighbor** (Competing supervised method for cross-dataset cell-type assessment)
- **scran** (Implements MNN-based batch correction used as a competing approach)

## Examples

```
result = BAMMSC(X = data_BAMMSC, K = 4); adjustedRandIndex(unlist(result$mem), unlist(Z))
```

## Evaluation signals

- simulation_ari_results.csv contains one averaged-ARI cell per scenario-by-method combination with 100 replicates per scenario
- BAMM-SC attains the highest average ARI across scenarios, and its ARI increases with the number of individuals while competitors remain stable or decrease
- Under Splatter mis-specification, BAMM-SC still achieves the top ARI in nearly all scenarios (occasional exceptions, e.g., the 10% label-disagreement case)
- BAMMSC run returns converged MCMC output including loglik, AIC, and BIC fields
- ARI computed consistently as adjustedRandIndex(unlist(predicted_memberships), unlist(true_labels))

## Limitations

- Filtering out genes with excessive zeros may be problematic for rare cell type identification
- Zero-inflation is not explicitly modeled, which may or may not affect clustering accuracy
- The supervised strategy for large datasets (train on a subset, predict the rest) may potentially miss some rare clusters
- Improvement over competitors is less substantial under Splatter mis-specification than under BAMM-SC's own generative model, and BAMM-SC did not achieve the highest ARI in some scenarios such as the 10% cluster-label disagreement case

## Evidence

- [abstract] BAMM-SC method for multi-individual clustering: "we develop a Bayesian mixture model for single-cell sequencing (BAMM-SC) method to cluster scRNA-seq data from multiple individuals simultaneously"
- [intro] Benchmarking objective against existing methods: "benchmark its performance against existing clustering methods in simulation studies, and evaluate our method for its accuracy, stability, and efficiency"
- [intro] Competing methods list: "scRNA-seq tailored unsupervised methods, such as SIMLR9, CellTree10, SC311, TSCAN12, and DIMM-SC13"
- [readme] ARI scoring pattern from README example: "result=BAMMSC(X=data_BAMMSC,K=4)
adjustedRandIndex(unlist(result$mem),unlist(Z))"
- [readme] Default MCMC settings documented in README: "nBurn: number of iterations discarded at the initial portion of the Markov chain, default is 400
* maxIter.BAMMSC: maximum number of iterations in MCMC, default is 500"
- [discussion] Runtime feasibility for large simulated datasets: "For the simulated dataset consist of 10 individuals with 4000 cells each, the computational time for clustering is about 30 min"
