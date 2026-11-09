---
name: scrna-seq-heterogeneity-scenario-design
description: Use when benchmarking multi-individual scRNA-seq clustering methods (e.g.
  BAMM-SC vs SIMLR, SC3, DIMM-SC) and you need factorial simulation grids varying
  heterogeneity among individuals and cohort size, with ARI as the outcome.
when_to_use_negative:
- Clustering a single individual — the BAMMSC interface falls back to DIMMSC when
  X is a single matrix, so the multi-individual heterogeneity sweep does not apply
- Methods requiring preprocessing that changes input counts (e.g. normalized expression
  matrices) rather than raw UMI counts — BAMM-SC takes raw count data as input
- Trajectory or cell-cycle/developmental-stage analysis — the model assumes each cell
  belongs to one distinct cluster and such directions are explicitly deferred to future
  work
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: BAMM-SC
  role: Primary multi-individual clustering method fit on each simulated dataset with
    default MCMC settings
  repo: lichen-lab/BAMMSC
- name: SIMLR
  role: Competing unsupervised scRNA-seq clustering method run on the same simulated
    count matrices
- name: CellTree
  role: Competing unsupervised scRNA-seq clustering method
- name: SC3
  role: Competing unsupervised scRNA-seq clustering method
- name: TSCAN
  role: Competing unsupervised scRNA-seq clustering method
- name: DIMM-SC
  role: Competing single-individual Dirichlet multinomial mixture clustering method
- name: MetaNeighbor
  role: Competing supervised method assessing cell-type profile replication across
    datasets
- name: scran
  role: Provides the mutual nearest neighbors (MNN) competing method
- name: mclust
  role: Supplies adjustedRandIndex used to score membership vectors against true labels
- name: dirmult
  role: R dependency for the Dirichlet multinomial simulation model
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:4296bbb6e934ccbf21526d0235369459ea8ef90dc1863023ef81ac15c7489b6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - BAMM-SC
  - CellTree
  - DIMM-SC
  - MetaNeighbor
  - SC3
  - SIMLR
  - TSCAN
  - dirmult
  - mclust
  - scran
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# scrna-seq-heterogeneity-scenario-design

## Summary

Design simulation benchmark scenarios that sweep individual-level heterogeneity (mean sigma_ik^2) and number of individuals for multi-individual droplet scRNA-seq clustering, scoring methods with adjustedRandIndex against true simulated labels.

## When to use

Use when benchmarking multi-individual scRNA-seq clustering methods (e.g. BAMM-SC vs SIMLR, SC3, DIMM-SC) and you need factorial simulation grids varying heterogeneity among individuals and cohort size, with ARI as the outcome.

## When NOT to use

- Clustering a single individual — the BAMMSC interface falls back to DIMMSC when X is a single matrix, so the multi-individual heterogeneity sweep does not apply
- Methods requiring preprocessing that changes input counts (e.g. normalized expression matrices) rather than raw UMI counts — BAMM-SC takes raw count data as input
- Trajectory or cell-cycle/developmental-stage analysis — the model assumes each cell belongs to one distinct cluster and such directions are explicitly deferred to future work

## Inputs

- Simulated UMI count matrices (list of per-individual count matrices, genes x cells, from the hierarchical Dirichlet multinomial mixture model)
- True simulated cluster labels (membership vectors per individual)
- Grid specification of mean sigma_ik^2 heterogeneity levels and numbers of individuals

## Outputs

- ARI per replicate for each method (via mclust::adjustedRandIndex)
- Averaged ARI table per scenario-by-method cell (e.g. simulation_ari_results.csv)

## How to apply

Generate UMI count matrices by sampling from the Bayesian hierarchical Dirichlet multinomial mixture model, sweeping the grid of mean sigma_ik^2 heterogeneity levels and numbers of individuals (e.g. 10 individuals with 4000 cells each), with 100 replicates per scenario cell. Fit BAMMSC(X, K) with default MCMC settings (nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10) and run the competing methods (SIMLR, CellTree, SC3, TSCAN, DIMM-SC, MetaNeighbor, MNN via scran) on the same simulated matrices. Score every method's membership vectors against the true simulated labels with mclust::adjustedRandIndex for each replicate. Average ARI over the 100 replicates within each scenario-by-method cell and write the table (e.g. simulation_ari_results.csv). The rationale is that BAMM-SC explicitly models data heterogeneity and batch effect among multiple individuals, so its advantage should grow with more individuals while competitors stall or decrease.

## Related tools

- **BAMM-SC** (Primary multi-individual clustering method fit on each simulated dataset with default MCMC settings) — lichen-lab/BAMMSC
- **SIMLR** (Competing unsupervised scRNA-seq clustering method run on the same simulated count matrices)
- **CellTree** (Competing unsupervised scRNA-seq clustering method)
- **SC3** (Competing unsupervised scRNA-seq clustering method)
- **TSCAN** (Competing unsupervised scRNA-seq clustering method)
- **DIMM-SC** (Competing single-individual Dirichlet multinomial mixture clustering method)
- **MetaNeighbor** (Competing supervised method assessing cell-type profile replication across datasets)
- **scran** (Provides the mutual nearest neighbors (MNN) competing method)
- **mclust** (Supplies adjustedRandIndex used to score membership vectors against true labels)
- **dirmult** (R dependency for the Dirichlet multinomial simulation model)

## Examples

```
library(BAMMSC); library(mclust); result=BAMMSC(X=data_BAMMSC,K=4); adjustedRandIndex(unlist(result$mem),unlist(Z))
```

## Evaluation signals

- Confirm in the results table (e.g. simulation_ari_results.csv) that BAMM-SC attains the highest average ARI across scenarios
- Confirm BAMM-SC's ARI increases with the number of individuals while competitors' ARIs remain stable or decrease
- Verify each scenario cell averages ARI over the full 100 replicates
- Verify all methods are scored against the same true simulated labels with mclust::adjustedRandIndex on the same simulated count matrices
- Check BAMMSC convergence via its loglik vector and supplied tolerances (likTol.BAMMSC=1e-10)

## Limitations

- Filtering genes with excessive zeros prior to analysis may be problematic for rare cell type identification
- Zero-inflation is not explicitly modeled, which may or may not affect clustering accuracy
- BAMM-SC did not achieve the highest ARI in every scenario under some label-disagreement settings (e.g. the 10% cluster-label disagreement case)
- The number of clusters is treated as a continuum; the authors recommend trying a range of cluster numbers rather than a single K

## Evidence

- [abstract] BAMM-SC takes raw count data as input and accounts for data heterogeneity and batch effect among multiple individuals in a unified Bayesian hierarchical model framework: "BAMM-SC takes raw count data as input and accounts for data heterogeneity and batch effect among multiple individuals in a unified Bayesian hierarchical model"
- [intro] benchmark objective in simulations: "benchmark its performance against existing clustering methods in simulation studies, and evaluate our method for its accuracy, stability, and efficiency"
- [methods] ARI calculation from article methods: "adjustedRandIndex(unlist(Zpred),unlist(Z))"
- [readme] BAMMSC run with example data: "result=BAMMSC(X=data_BAMMSC,K=4)"
- [abstract] multi-individual advantage finding: "BAMM-SC outperformed existing clustering methods with considerable improved clustering accuracy, particularly in the presence of heterogeneity among individuals"
- [discussion] model assumes one cluster per cell: "we assume that each cell belongs to one distinct cluster. The posterior probability measures the clustering uncertainty"
