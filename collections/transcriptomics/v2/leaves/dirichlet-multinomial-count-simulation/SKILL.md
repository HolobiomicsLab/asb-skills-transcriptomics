---
name: dirichlet-multinomial-count-simulation
description: 'Use when you need controlled simulation benchmarks for multi-individual
  scRNA-seq clustering: you must vary individual-level heterogeneity and donor count
  to test whether a method (e.g., BAMM-SC vs SIMLR/SC3/DIMM-SC) benefits from more
  individuals.'
when_to_use_negative:
- You are clustering real droplet scRNA-seq data rather than benchmarking — use BAMMSC(X,
  K) directly on the observed count matrices instead of simulated ones
- Input is a single individual (a matrix, not a list) — then DIMMSC, not the multi-individual
  BAMMSC simulation/benchmark path, applies
- The scientific question concerns zero-inflation patterns or trajectory/developmental-stage
  assignment, which BAMM-SC's Dirichlet multinomial simulation does not model
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BAMM-SC
  role: Clustering of each simulated count matrix via BAMMSC(X, K) with default MCMC
    settings, and the generative model being simulated from
  repo: github.com/lichen-lab/BAMMSC
- name: dirmult
  role: Sampling UMI counts from the hierarchical Dirichlet multinomial model
- name: mclust
  role: adjustedRandIndex to score method memberships against true simulated labels
- name: SIMLR
  role: Competing scRNA-seq clustering method evaluated on the same simulated matrices
- name: CellTree
  role: Competing scRNA-seq clustering method evaluated on the same simulated matrices
- name: SC3
  role: Competing scRNA-seq clustering method evaluated on the same simulated matrices
- name: TSCAN
  role: Competing scRNA-seq clustering method evaluated on the same simulated matrices
- name: DIMM-SC
  role: Competing scRNA-seq clustering method evaluated on the same simulated matrices
- name: MetaNeighbor
  role: Competing method evaluated on the same simulated matrices
- name: scran
  role: Competing MNN-based method evaluated on the same simulated matrices
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:76673ec3167061e3cd9888df3b7120feefaeb85254f12f8ad785d4857b11e362
schema_version: 0.2.0
metadata:
  license_tier: open
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# dirichlet-multinomial-count-simulation

## Summary

Generate synthetic droplet-based scRNA-seq UMI count matrices by sampling from the Bayesian hierarchical Dirichlet multinomial mixture model (BAMM-SC's generative model), sweeping individual-level heterogeneity (sigma_ik^2) and the number of individuals to benchmark clustering methods by ARI.

## When to use

Use when you need controlled simulation benchmarks for multi-individual scRNA-seq clustering: you must vary individual-level heterogeneity and donor count to test whether a method (e.g., BAMM-SC vs SIMLR/SC3/DIMM-SC) benefits from more individuals.

## When NOT to use

- You are clustering real droplet scRNA-seq data rather than benchmarking — use BAMMSC(X, K) directly on the observed count matrices instead of simulated ones
- Input is a single individual (a matrix, not a list) — then DIMMSC, not the multi-individual BAMMSC simulation/benchmark path, applies
- The scientific question concerns zero-inflation patterns or trajectory/developmental-stage assignment, which BAMM-SC's Dirichlet multinomial simulation does not model

## Inputs

- Grid of simulation scenarios: mean sigma_ik^2 individual-level heterogeneity levels crossed with numbers of individuals
- True cluster labels Z per simulated cell (used to compute ARI)
- R environment with BAMMSC (from install_github('lichen-lab/BAMMSC')), dirmult, and mclust packages

## Outputs

- Simulated UMI count matrices: a list X with one gene-by-cell count matrix per individual (same number of genes, varying number of cells)
- Per-replicate ARI values for BAMM-SC and each competing method
- simulation_ari_results.csv: table of average ARI (over 100 replicates) per scenario-by-method cell

## How to apply

Sample UMI count matrices for multiple individuals from the Bayesian hierarchical Dirichlet multinomial mixture model using the dirmult package, sweeping a grid of mean sigma_ik^2 heterogeneity levels and numbers of individuals with 100 replicates per scenario. Cluster each simulated dataset with BAMMSC(X, K) using the default MCMC settings (nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10) and run competitors (SIMLR, CellTree, SC3, TSCAN, DIMM-SC, MetaNeighbor, MNN/scran) on the same count matrices. Score every method's membership vectors against the true simulated labels with mclust::adjustedRandIndex per replicate, then average ARI over the 100 replicates within each scenario-by-method cell and export simulation_ari_results.csv. The rationale is that the simulation reproduces exactly the three levels of variability (genes, cell types, individuals) that BAMM-SC models, so a correct benchmark shows whether pooling individuals improves accuracy.

## Related tools

- **BAMM-SC** (Clustering of each simulated count matrix via BAMMSC(X, K) with default MCMC settings, and the generative model being simulated from) — github.com/lichen-lab/BAMMSC
- **dirmult** (Sampling UMI counts from the hierarchical Dirichlet multinomial model)
- **mclust** (adjustedRandIndex to score method memberships against true simulated labels)
- **SIMLR** (Competing scRNA-seq clustering method evaluated on the same simulated matrices)
- **CellTree** (Competing scRNA-seq clustering method evaluated on the same simulated matrices)
- **SC3** (Competing scRNA-seq clustering method evaluated on the same simulated matrices)
- **TSCAN** (Competing scRNA-seq clustering method evaluated on the same simulated matrices)
- **DIMM-SC** (Competing scRNA-seq clustering method evaluated on the same simulated matrices)
- **MetaNeighbor** (Competing method evaluated on the same simulated matrices)
- **scran** (Competing MNN-based method evaluated on the same simulated matrices)

## Examples

```
library(devtools); install_github("lichen-lab/BAMMSC"); library(BAMMSC); result = BAMMSC(X_sim, K=4, nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10); mclust::adjustedRandIndex(unlist(result$mem), unlist(Z))
```

## Evaluation signals

- simulation_ari_results.csv exists with one averaged-ARI value per scenario-by-method cell (100 replicates averaged per cell)
- BAMM-SC attains the highest average ARI across the swept scenarios
- BAMM-SC's ARI increases monotonically with the number of simulated individuals while competitors' ARIs remain stable or decrease
- Each replicate's ARI is computed via mclust::adjustedRandIndex between the method's membership vector and the true simulated labels Z
- Simulated X is a list of per-individual count matrices with identical numbers of genes but possibly differing cell counts, matching BAMMSC's expected input

## Limitations

- In the 10% cluster-label disagreement scenarios, BAMM-SC did not always achieve the highest ARI (highest only in most, not all, scenarios)
- The Dirichlet multinomial model does not explicitly model zero-inflation, which may or may not affect clustering accuracy
- Filtering genes with excessive zeros before analysis may be problematic for rare cell type identification
- Runtime scales with simulated size: about 30 min for a simulated dataset of 10 individuals with 4000 cells each; no parallel/GPU speed-up implemented

## Evidence

- [full_text] as the number of simulated individuals increased, BAMM-SC achieved higher ARI, while the ARIs of the competing clustering methods either remained stable or decreased.: [evidence span withheld — anchor status: not_found]
- [readme] X: a list with each element a UMI count matrix for each individual, with row as number of genes and column as number of cells.: "X: a list with each element a UMI count matrix for each individual, with row as number of genes and column as number of cells."
- [discussion] For the simulated dataset consist of 10 individuals with 4000 cells each, the computational time for clustering is about 30 min: "For the simulated dataset consist of 10 individuals with 4000 cells each, the computational time for clustering is about 30 min"
- [methods] adjustedRandIndex(unlist(result$mem),unlist(Z)): "adjustedRandIndex(unlist(result$mem),unlist(Z))"
