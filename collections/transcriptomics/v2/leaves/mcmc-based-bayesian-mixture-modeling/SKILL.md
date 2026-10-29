---
name: mcmc-based-bayesian-mixture-modeling
description: Use when clustering raw UMI count matrices from multiple individuals
  (each a genes x cells matrix in a list) where individual-level heterogeneity/batch
  effects exist, or when a trained model must predict cluster labels for held-out
  cells via a training-proportion split.
when_to_use_negative:
- Data consist of normalized, batch-corrected expression matrices rather than raw
  UMI counts — BAMM-SC explicitly takes raw count data and models library/depth variation
  itself
- Single-cell data are plate-based/Smart-seq2 rather than droplet-based (10x Chromium)
  scRNA-seq from population studies
- Identifying rare cell clusters is critical — the authors note the supervised strategy
  'may potentially miss some rare clusters', and genes with excessive zeros are filtered
  out, which is problematic for rare cell type identification
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BAMM-SC (BAMMSC R package)
  role: Core Bayesian hierarchical Dirichlet multinomial mixture model run across
    multiple individuals via BAMMSC(), with supervised prediction via BAMMSC.predict(X,
    K, p, seed)
  repo: https://github.com/lichen-lab/BAMMSC
- name: DIMMSC
  role: Initial per-individual (or combined single-individual) Dirichlet multinomial
    mixture clustering step called internally or directly as DIMMSC(data, K); invoked
    automatically when X is a matrix
  repo: https://github.com/lichen-lab/BAMMSC
- name: mclust
  role: Supplies adjustedRandIndex to compare predicted/clustering memberships against
    true labels, e.g. adjustedRandIndex(unlist(Zpred), unlist(Z))
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:06c17401826f43f6d3c6f89f848ddca5d8ddcabc743a606308f24359ee0c9865
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - BAMM-SC
  - BAMM-SC (BAMMSC R package)
  - DIMMSC
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# mcmc-based-bayesian-mixture-modeling

## Summary

BAMM-SC clusters droplet-based scRNA-seq UMI count data from multiple individuals simultaneously using a Bayesian hierarchical Dirichlet multinomial mixture model fitted by MCMC, explicitly modeling gene, cell-type, and individual-level variability to handle batch effects such as unbalanced sequencing depths.

## When to use

Use when clustering raw UMI count matrices from multiple individuals (each a genes x cells matrix in a list) where individual-level heterogeneity/batch effects exist, or when a trained model must predict cluster labels for held-out cells via a training-proportion split.

## When NOT to use

- Data consist of normalized, batch-corrected expression matrices rather than raw UMI counts — BAMM-SC explicitly takes raw count data and models library/depth variation itself
- Single-cell data are plate-based/Smart-seq2 rather than droplet-based (10x Chromium) scRNA-seq from population studies
- Identifying rare cell clusters is critical — the authors note the supervised strategy 'may potentially miss some rare clusters', and genes with excessive zeros are filtered out, which is problematic for rare cell type identification

## Inputs

- List of raw UMI count matrices, one per individual (rows = genes, columns = cells; same number of genes across individuals, cell numbers may differ) — e.g. data_BAMMSC with 10 individuals, 100 genes, 400 cells each
- Number of clusters K (predefined from prior knowledge or selected via AIC/BIC)
- Training proportion p for supervised prediction (e.g. 0.1–1.0) and seed for BAMMSC.predict

## Outputs

- mem: list of per-individual cell cluster-membership vectors
- delta: list of 3-D matrices of posterior probability estimates [#cell, #cluster, #iteration] per individual
- alpha: list of per-iteration alpha estimates [#gene, #cluster, #individuals]; pi and sigma2 estimates
- AIC and BIC values, and the MCMC log-likelihood vector for convergence checking
- ARI (adjusted Rand index) against true labels when reference labels are available

## How to apply

Install BAMMSC via devtools::install_github('lichen-lab/BAMMSC') and pass X, a list of UMI count matrices (rows = genes, columns = cells; gene counts must match across individuals, cell counts may differ), to BAMMSC(X, K=...) with K predefined from prior knowledge or chosen via the returned AIC/BIC. The MCMC uses nBurn=400 discarded burn-in iterations, maxIter.BAMMSC=500 MCMC iterations, and log-likelihood tolerance likTol.BAMMSC=1e-10; DIMMSC defaults (maxIter.DIMMSC=400, tol.DIMMSC=1e-4, likTol.DIMMSC=1e-2) govern the initial per-individual clustering (kmeans or random initialization, Ronning or Weir alpha initialization). Extract cluster memberships from result$mem and posterior probabilities from result$delta. For large datasets, call BAMMSC.predict(X, K, p, seed=1), which trains the model on the p-fraction of cells per individual and classifies the remaining cells with the trained model (p from 0.1 to 1.0); the authors verified that with p=0.1 on a 4000-cell-per-individual simulated dataset a run completes in ~90 seconds and test ARI plateaus once the training pool reaches 4000 cells. Validate against true labels with mclust::adjustedRandIndex(unlist(Zpred), unlist(Z)).

## Related tools

- **BAMM-SC (BAMMSC R package)** (Core Bayesian hierarchical Dirichlet multinomial mixture model run across multiple individuals via BAMMSC(), with supervised prediction via BAMMSC.predict(X, K, p, seed)) — https://github.com/lichen-lab/BAMMSC
- **DIMMSC** (Initial per-individual (or combined single-individual) Dirichlet multinomial mixture clustering step called internally or directly as DIMMSC(data, K); invoked automatically when X is a matrix) — https://github.com/lichen-lab/BAMMSC
- **mclust** (Supplies adjustedRandIndex to compare predicted/clustering memberships against true labels, e.g. adjustedRandIndex(unlist(Zpred), unlist(Z)))

## Examples

```
library(BAMMSC); Zpred=BAMMSC.predict(X=data_BAMMSC,K=4,p=0.5,seed=1); adjustedRandIndex(unlist(Zpred),unlist(Z))
```

## Evaluation signals

- adjustedRandIndex(unlist(result$mem), unlist(Z)) > adjustedRandIndex of competing methods (DIMM-SC, SIMLR, etc.) for the same data, as in the paper's simulations
- BAMMSC.predict ARI at a given training proportion p should approach the full BAMMSC unsupervised ARI as p increases, with test ARI plateauing once the training pool reaches ~4000 cells
- MCMC log-likelihood vector (result$loglik) should converge: changes below likTol.BAMMSC=1e-10 (BAMMSC) / likTol.DIMMSC=1e-2 (DIMMSC) before maxIter (500/400) is reached
- Length of each vector in result$mem equals the per-individual cell count of the corresponding input matrix, and list length equals the number of individuals
- Timing sanity checks: ~1.5/2.5/4.5 h for the PBMC, mouse lung, and human skin datasets and ~30 min for 10 individuals x 4000 cells in full unsupervised mode; ~90 s for supervised prediction at p=0.1 on 4000-cell data

## Limitations

- Genes with excessive zeros are filtered out prior to analysis (lowly-expressed genes assumed uninformative), which may be problematic for rare cell type identification
- Zero-inflation is not explicitly modeled, which may or may not affect clustering accuracy
- Each cell is assumed to belong to one distinct cluster; the posterior probability measures clustering uncertainty, not cell cycle or developmental stage
- The supervised strategy for large datasets is efficient but may potentially miss some rare clusters
- The number of clusters is treated as a continuum — authors recommend trying a range of K values rather than a single value; in some 10% cluster-label-disagreement scenarios BAMM-SC did not achieve the highest ARI

## Evidence

- [readme] BAMMSC is an R package for clustering droplet-based single cell transcriptomic data from multiple individuals simultaneously.: "BAMMSC is an R package for clustering droplet-based single cell transcriptomic data from multiple individuals simultaneously. It adopts a Bayesian hierarchical Dirichlet multinomial mixture model"
- [abstract] BAMM-SC takes raw count data and accounts for heterogeneity and batch effect in a unified Bayesian hierarchical model.: "BAMM-SC takes raw count data as input and accounts for data heterogeneity and batch effect among multiple individuals in a unified Bayesian hierarchical model framework"
- [discussion] Supervised classification: train on a subset, classify the rest.: "users can first apply BAMM-SC on a small subset of single cells in each individual, and save predicted cluster labels as well as other informative parameters"
- [readme] BAMMSC.predict training proportion argument.: "p: proportion of cells used to train a BAMMSC model and the cluster labels of the rest cells are predicted by the trained label"
- [discussion] Supervised strategy limitation for rare clusters.: "although our supervised strategy is proven to work for large datasets efficiently, it may potentially miss some rare clusters"
- [readme] Convergence parameters for the MCMC.: "nBurn: number of iterations discarded at the initial portion of the Markov chain, default is 400"
