---
name: dirichlet-multinomial-mixture-modeling
description: Use when raw UMI count matrices from multiple individuals (e.g., 10x
  Chromium droplet scRNA-seq) must be jointly clustered without prior batch correction,
  and posterior per-cell clustering uncertainty is needed.
when_to_use_negative:
- Data are already normalized, batch-corrected, or merged without library adjustment
  — the model expects raw counts and handles heterogeneity/batch effects itself
- The goal is trajectory or cell-cycle/developmental-stage analysis — the model assumes
  each cell belongs to one distinct cluster and posterior probabilities measure clustering
  uncertainty only
- Identifying very rare cell types — genes with excessive zeros are filtered out,
  which the authors note may be problematic for rare cell type identification
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_2229
tools:
- name: BAMMSC
  role: R package implementing the Bayesian hierarchical Dirichlet multinomial mixture
    model with Gibbs sampling for multi-individual clustering
  repo: https://github.com/lichen-lab/BAMMSC
- name: DIMM-SC
  role: Single-individual Dirichlet multinomial variant called by BAMMSC when X is
    a single matrix or option='DIMMSC'
  repo: https://github.com/lichen-lab/BAMMSC
- name: dirmult
  role: Dependency providing Dirichlet multinomial support
- name: Rcpp / RcppArmadillo
  role: C++ back-end dependencies for the sampler implementation
- name: mclust
  role: Provides adjustedRandIndex for validating recovered labels against truth labels
provenance:
  source_task_ids:
  - task_006
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:9bc14ce8596a85f6e1cfc0df32823f492bdd4a281515c2ff5405c37f92475822
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - 10x Genomics Chromium system
  - BAMM-SC
  - BAMMSC
  - BAMMSC R package
  - DIMM-SC
  - Rcpp
  - Rcpp / RcppArmadillo
  - RcppArmadillo
  - SC3
  - TSCAN
  - dirmult
  - mclust
  - scran
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# dirichlet-multinomial-mixture-modeling

## Summary

Fit a Bayesian hierarchical Dirichlet multinomial mixture model (as implemented in the BAMMSC R package) to cluster droplet-based UMI count data from multiple individuals simultaneously, explicitly modeling variability across genes, cell types, and individuals.

## When to use

Use when raw UMI count matrices from multiple individuals (e.g., 10x Chromium droplet scRNA-seq) must be jointly clustered without prior batch correction, and posterior per-cell clustering uncertainty is needed.

## When NOT to use

- Data are already normalized, batch-corrected, or merged without library adjustment — the model expects raw counts and handles heterogeneity/batch effects itself
- The goal is trajectory or cell-cycle/developmental-stage analysis — the model assumes each cell belongs to one distinct cluster and posterior probabilities measure clustering uncertainty only
- Identifying very rare cell types — genes with excessive zeros are filtered out, which the authors note may be problematic for rare cell type identification

## Inputs

- List of per-individual gene-by-cell UMI count matrices from droplet-based scRNA-seq (e.g., 10x Chromium), genes filtered to remove those with excessive zeros
- Number of clusters K (predefined or selected over a range using AIC/BIC)
- Optionally, a single count matrix (Routed to DIMMSC for single-individual analysis)

## Outputs

- mem: list of per-individual cell cluster membership vectors
- delta: list of per-individual [#cell, #cluster, #iteration] posterior probability matrices for vague-cell assessment
- alpha and sigma2: [#gene, #cluster, #iteration] parameter estimates (alpha averaged post burn-in)
- loglik: MCMC log-likelihood trace for convergence inspection; AIC/BIC for cluster-number choice
- pi: cluster proportion estimates; ARI via mclust::adjustedRandIndex when truth labels exist

## How to apply

Assemble the data as a list of gene-by-cell UMI count matrices, one per individual (same genes across individuals; cell counts may differ). Install the BAMMSC package via install_github('lichen-lab/BAMMSC') (depends on dirmult, Rcpp, RcppArmadillo) and run BAMMSC(X, K) with the multinomial-Dirichlet likelihood carrying a log-normal LN(mu_ik, sigma_ik^2) prior on alpha_ilk; the Gibbs sampler alternates alpha and cell-label z_jl updates with nBurn=400 burn-in, maxIter.BAMMSC=500 MCMC iterations, and likTol.BAMMSC=1e-10 convergence tolerance. Choose K from prior knowledge or via the returned AIC/BIC, and try a range of K values since cluster number is treated as a continuum. Extract mem (per-individual memberships), loglik (for MCMC convergence checks), and delta (per-cell posterior probability matrices [#cell, #cluster, #iteration]) to flag vague cells with maximum posterior probability < 0.95. Averaging alpha over the post-burn-in chain gives gene-by-cluster-by-individual parameter estimates.

## Related tools

- **BAMMSC** (R package implementing the Bayesian hierarchical Dirichlet multinomial mixture model with Gibbs sampling for multi-individual clustering) — https://github.com/lichen-lab/BAMMSC
- **DIMM-SC** (Single-individual Dirichlet multinomial variant called by BAMMSC when X is a single matrix or option='DIMMSC') — https://github.com/lichen-lab/BAMMSC
- **dirmult** (Dependency providing Dirichlet multinomial support)
- **Rcpp / RcppArmadillo** (C++ back-end dependencies for the sampler implementation)
- **mclust** (Provides adjustedRandIndex for validating recovered labels against truth labels)

## Examples

```
library(BAMMSC); data("data_BAMMSC"); result=BAMMSC(data_BAMMSC, K=4, nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10)
```

## Evaluation signals

- MCMC log-likelihood trace (result$loglik) plateaus within the 500-iteration budget after 400 burn-in iterations
- adjustedRandIndex(unlist(result$mem), true_labels) is high on simulated data (e.g., data_BAMMSC with known K=4 labels) and exceeds baselines such as DIMM-SC, SC3, TSCAN, Seurat
- Fraction of vague cells (max delta < 0.95) is a small subset of all cells, and t-SNE shows them at cluster boundaries rather than cluster cores
- Output objects match documented schema: mem is a list of [#cell] vectors (one per individual), delta is a list of [#cell, #cluster, #iteration] matrices
- AIC/BIC decrease over a range of tried K values around the expected cluster number

## Limitations

- Genes with excessive zeros are filtered out; this may be problematic for rare cell type identification
- Zero-inflation is not explicitly modeled, which may or may not affect clustering accuracy
- Each cell is assumed to belong to one distinct cluster; posterior probabilities measure clustering uncertainty, not cell cycle or developmental stage
- In the 10% cluster-label disagreement scenario, BAMM-SC did not achieve the highest ARI in all cases (Supplementary Fig. 16)
- Further speed-up via parallel computing or GPU is not yet implemented; ~1.5–4.5 h runtimes on the three real datasets, ~30 min on a 10-individual simulated dataset

## Evidence

- [methods] Bayesian hierarchical Dirichlet multinomial mixture model with log-normal priors: [evidence span withheld — anchor status: not_found]
- [readme] MCMC configuration: "nBurn: number of iterations discarded at the initial portion of the Markov chain, default is 400"
- [readme] Posterior output for uncertainty assessment: "delta: a list of 3-D matrix. Each matrix [#cell, #cluster, #iteration] is the poster probability estimates for each individual across all iterations"
- [discussion] Gene filtering step: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [results] Vague-cell threshold: [evidence span withheld — anchor status: not_found]
