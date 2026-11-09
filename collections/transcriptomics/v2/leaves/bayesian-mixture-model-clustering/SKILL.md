---
name: bayesian-mixture-model-clustering
description: Use when raw UMI count matrices from several individuals (e.g. 10x Genomics
  Chromium libraries) show batch effects or individual-level heterogeneity and single-individual
  methods like DIMM-SC or naive merging produce unreliable clusters.
when_to_use_negative:
- Single-individual analysis where no cross-individual heterogeneity exists — use
  DIMMSC (call DIMMSC(X, K) or pass a plain matrix to BAMMSC)
- Rare cell type discovery, since genes with excessive zeros are filtered out and
  the supervised strategy for large datasets may potentially miss rare clusters
- Trajectory, cell-cycle, or developmental-stage interpretation — the model assumes
  each cell belongs to one distinct cluster and posterior probabilities measure clustering
  uncertainty only
- Data requiring explicit zero-inflation modeling, which BAMM-SC does not provide
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3361
tools:
- name: BAMMSC
  role: Core Bayesian hierarchical Dirichlet multinomial mixture model that clusters
    cells across multiple individuals simultaneously; installed via install_github("lichen-lab/BAMMSC")
  repo: github.com/lichen-lab/BAMMSC
- name: DIMMSC
  role: Single-individual Dirichlet multinomial mixture mode invoked via option="DIMMSC"
    or when X is a single matrix; used as the clustering baseline/extension within
    BAMMSC
  repo: https://github.com/lichen-lab/BAMMSC/blob/master/10XGenomics.md
- name: mclust
  role: Provides adjustedRandIndex to score BAMMSC memberships against true/reference
    cell labels
- name: dirmult
  role: Dependency loaded for the Dirichlet multinomial computations underlying the
    model
- name: devtools
  role: Used to install the BAMMSC package from GitHub
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:b59d58b7be038ce4a49a5b61f08a5ec52e72efd989dfcebece86ab51b632a9f1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BAMM-SC
  - BAMMSC
  - CellTree
  - DIMM-SC
  - DIMMSC
  - MetaNeighbor
  - SC3
  - SIMLR
  - TSCAN
  - devtools
  - dirmult
  - mclust
  - scran
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 20
---

# bayesian-mixture-model-clustering

## Summary

Clustering droplet-based scRNA-seq UMI count data from multiple individuals with BAMM-SC, a Bayesian hierarchical Dirichlet multinomial mixture model that jointly models genes, cell types, and individual-level variability (batch effect, unbalanced sequencing depth) in a unified framework.

## When to use

Use when raw UMI count matrices from several individuals (e.g. 10x Genomics Chromium libraries) show batch effects or individual-level heterogeneity and single-individual methods like DIMM-SC or naive merging produce unreliable clusters.

## When NOT to use

- Single-individual analysis where no cross-individual heterogeneity exists — use DIMMSC (call DIMMSC(X, K) or pass a plain matrix to BAMMSC)
- Rare cell type discovery, since genes with excessive zeros are filtered out and the supervised strategy for large datasets may potentially miss rare clusters
- Trajectory, cell-cycle, or developmental-stage interpretation — the model assumes each cell belongs to one distinct cluster and posterior probabilities measure clustering uncertainty only
- Data requiring explicit zero-inflation modeling, which BAMM-SC does not provide

## Inputs

- List of UMI count matrices (one per individual; rows = genes, columns = cells; same number of genes across individuals)
- K: number of clusters (default 4; predefine from prior knowledge or select via AIC/BIC)
- Optionally a single count matrix (matrix input routes to DIMMSC)

## Outputs

- mem: list of clustering membership vectors, one per individual
- delta: per-cell posterior probability matrices (#cell, #cluster, #iteration)
- sigma2: 3-D matrix of sigma^2 heterogeneity estimates (#gene, #cluster, #iteration)
- alpha: 3-D matrices of alpha estimates per iteration (#gene, #cluster, #individuals)
- pi: cluster proportion estimates
- loglik: log-likelihood trace across MCMC iterations, plus AIC and BIC

## How to apply

Install the BAMMSC R package via devtools::install_github("lichen-lab/BAMMSC") and assemble input X as a list of UMI count matrices, one per individual, with genes in rows and cells in columns (gene counts equal across individuals, cell counts may differ). Call BAMMSC(X, K) with default MCMC settings (nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10), where K is predefined from tissue knowledge or chosen by comparing AIC/BIC across a range of K values; BAMMSC initializes clusters with kmeans (method_cluster_initial) and alpha with Ronning's method. Read cluster memberships from result$mem (one vector per individual) and clustering uncertainty from result$delta posterior probabilities; use result$loglik to check MCMC convergence. Validate assignments with mclust::adjustedRandIndex against true or reference labels, e.g. adjustedRandIndex(unlist(result$mem), unlist(Z)). For supervised extension on large datasets, run BAMMSC.predict(X, K, p=0.5) to train on a subset of cells and predict labels for the rest.

## Related tools

- **BAMMSC** (Core Bayesian hierarchical Dirichlet multinomial mixture model that clusters cells across multiple individuals simultaneously; installed via install_github("lichen-lab/BAMMSC")) — github.com/lichen-lab/BAMMSC
- **DIMMSC** (Single-individual Dirichlet multinomial mixture mode invoked via option="DIMMSC" or when X is a single matrix; used as the clustering baseline/extension within BAMMSC) — https://github.com/lichen-lab/BAMMSC/blob/master/10XGenomics.md
- **mclust** (Provides adjustedRandIndex to score BAMMSC memberships against true/reference cell labels)
- **dirmult** (Dependency loaded for the Dirichlet multinomial computations underlying the model)
- **devtools** (Used to install the BAMMSC package from GitHub)

## Examples

```
library(BAMMSC); data("data_BAMMSC"); result=BAMMSC(data_BAMMSC,K=4)
```

## Evaluation signals

- adjustedRandIndex(unlist(result$mem), unlist(Z)) computed with mclust shows high agreement with true/reference labels; in the paper's simulations BAMM-SC attained the highest average ARI and its ARI increased with the number of individuals while competitors stalled or decreased
- result$loglik across the MCMC (nBurn=400, maxIter.BAMMSC=500) plateaus within the likTol.BAMMSC=1e-10 convergence tolerance
- Structure check: mem is a list of length #individuals, each vector of length #cells for that individual, with labels in 1..K
- AIC/BIC comparison across candidate K values supports the chosen cluster number (the paper recommends trying a range of cluster numbers since K is treated as a continuum)
- On the bundled data_BAMMSC (10 individuals, 100 genes, 400 cells each) with known labels z=rep(seq(4),each=100), the ARI should be high (near 1) for K=4

## Limitations

- Genes with excessive zeros are filtered out, which may be problematic for rare cell type identification
- Zero-inflation is not explicitly modeled, which may or may not affect clustering accuracy
- Each cell is assumed to belong to one distinct cluster; posterior probabilities cannot be interpreted as cell cycle or developmental stage quantification
- The supervised strategy for large datasets (BAMMSC.predict with p) may potentially miss some rare clusters
- The number of clusters is treated as a continuum, so practitioners should try a range of K values; in ~10% cluster-label disagreement scenarios BAMM-SC did not achieve the highest ARI
- No parallel/GPU speed-up is implemented; runtimes were ~1.5-4.5 h for real datasets (human PBMC, mouse lung, human skin) and ~30 min for a simulated 10-individual, 4000-cells-each dataset

## Evidence

- [abstract] BAMM-SC takes raw count data as input and accounts for data heterogeneity and batch effect among multiple individuals in a unified Bayesian hierarchical model: "BAMM-SC takes raw count data as input and accounts for data heterogeneity and batch effect among multiple individuals in a unified Bayesian hierarchical model framework"
- [readme] X argument spec: "X: a list with each element a UMI count matrix for each individual, with row as number of genes and column as number of cells."
- [readme] MCMC default settings: "nBurn: number of iterations discarded at the initial portion of the Markov chain, default is 400"
- [discussion] How to choose K: "we can predefine the number of clusters based on prior knowledge on the tissue or determine it using some standard model checking criterion"
- [methods] ARI evaluation: [evidence span withheld — anchor status: not_found]
- [discussion] Gene filtering caveat: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
