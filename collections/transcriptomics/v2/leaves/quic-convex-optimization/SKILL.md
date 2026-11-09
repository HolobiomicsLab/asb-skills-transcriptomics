---
name: quic-convex-optimization
description: Use when inferring a sparse gene co-expression network from normalized
  single-cell expression data for a fixed gene set, after computing scLink's robust
  correlation matrix, and needing to select lambda by BIC.
when_to_use_negative:
- The goal is directed or regulatory edges — scLink edges are undirected statistical
  dependencies and expression data alone cannot distinguish regulatory from co-regulatory
  edges
- Genes have not been filtered/normalized first (sclink_norm with library size 10^6,
  gene filtering by detection rate or mean expression) — the penalized likelihood
  assumes the preprocessed matrix
- A simple co-expression strength measure suffices — use sclink_cor correlations rather
  than the penalized network step
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: R package whose sclink_net function formulates the weighted graphical lasso
    objective, applies the positive semidefinite covariance correction, and calls
    QUIC over the lambda grid
  repo: https://github.com/Vivianstats/scLink
- name: QUIC
  role: Convex optimization solver used to maximize the penalized log-likelihood over
    positive definite concentration matrices Theta
- name: R
  role: Execution environment for the scLink package (install.packages('scLink') from
    CRAN)
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:81369629b0f95fae705f272d569adda7586ef40c153e03b7e4b997dde7e0eb6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - QUIC
  - R
  - scLink
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# quic-convex-optimization

## Summary

Solves scLink's data-adaptive penalized-likelihood estimation of the Gaussian graphical model concentration matrix using the QUIC solver, with penalty weights (1 - |r_j1j2|) derived from robust correlations and a positive semidefinite covariance correction S_p = S + |min{0,tau}|I.

## When to use

Use when inferring a sparse gene co-expression network from normalized single-cell expression data for a fixed gene set, after computing scLink's robust correlation matrix, and needing to select lambda by BIC.

## When NOT to use

- The goal is directed or regulatory edges — scLink edges are undirected statistical dependencies and expression data alone cannot distinguish regulatory from co-regulatory edges
- Genes have not been filtered/normalized first (sclink_norm with library size 10^6, gene filtering by detection rate or mean expression) — the penalized likelihood assumes the preprocessed matrix
- A simple co-expression strength measure suffices — use sclink_cor correlations rather than the penalized network step

## Inputs

- Robust correlation matrix for the filtered gene set (output of sclink_cor)
- Robust covariance matrix S for the same genes (positive semidefinite-corrected to S_p internally)
- Regularization parameter grid lda = seq(0.5, 0.1, -0.05)

## Outputs

- Concentration (precision) matrix Theta at the selected lambda (sclink_sigma.csv)
- Symmetric adjacency matrix adj of nonzero off-diagonal edges at the selected lambda
- sclink_edges.csv listing nonzero off-diagonal edges
- Per-lambda BIC and edge counts (nedge) from the summary list

## How to apply

Load the robust correlation matrix (from sclink_cor / task_001) and compute per-edge penalty weights (1 - |r_j1j2|) so strongly co-expressed gene pairs receive weaker penalties. Formulate the penalized Gaussian graphical model objective argmax_{Theta≻0} logdet(Theta) - tr(S_p Theta) - lambda * sum (1-|r_j1j2|) theta_j1j2, where S_p is the positive semidefinite approximation S + |min{0,tau}|I of the robust covariance. Solve with the QUIC solver over the lambda grid lda = seq(0.5, 0.1, -0.05) via sclink_net (ncores = 1), then compute BIC and edge count (nedge) per lambda and select the minimum-BIC model. Extract the concentration matrix (Sigma) and symmetric adjacency (adj) at the selected lambda, writing the concentration matrix to sclink_sigma.csv and nonzero off-diagonal edges to sclink_edges.csv.

## Related tools

- **scLink** (R package whose sclink_net function formulates the weighted graphical lasso objective, applies the positive semidefinite covariance correction, and calls QUIC over the lambda grid) — https://github.com/Vivianstats/scLink
- **QUIC** (Convex optimization solver used to maximize the penalized log-likelihood over positive definite concentration matrices Theta)
- **R** (Execution environment for the scLink package (install.packages('scLink') from CRAN)) — https://github.com/Vivianstats/scLink

## Examples

```
install.packages("scLink"); net <- sclink_net(expr = norm_dat, genes = genes, ncores = 1, lda = seq(0.5, 0.1, -0.05))
```

## Evaluation signals

- sclink_edges.csv exists and its row count equals nedge reported for the selected lambda
- nedge matches the number of nonredundant off-diagonal nonzero entries of the symmetric adjacency matrix adj
- Selected network is sparse: edges <= 5% of the p(p-1)/2 possible gene pairs
- The chosen lambda is the grid point seq(0.5, 0.1, -0.05) with minimum BIC in the summary list
- The returned concentration matrix corresponds to a positive definite Theta (log-likelihood term logdet(Theta) is defined)

## Limitations

- Edges are undirected statistical dependencies; regulatory, co-regulatory, and functionally coordinated edge types cannot be distinguished from expression data alone
- No software version for the scLink R package is documented in the article, complicating exact reproduction
- Imputed single-cell data may remain much sparser than bulk data, so bulk-data network methods (and results on imputed matrices) can perform poorly

## Evidence

- [intro] scLink adapts the Gaussian graphical model [29] to distinguish direct associations between genes from indirect ones and leads to sparse networks: "scLink adapts the Gaussian graphical model [29] to distinguish direct associations between genes from indirect ones and leads to sparse networks"
- [intro] Penalized likelihood with data-adaptive penalty weights: "scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge based on the observed co-expression"
- [methods] sclink_net computes robust correlations and the sparse network: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network"
- [discussion] Edges are undirected dependencies: "the edges identified by scLink are based on statistical dependencies and do not have directions"
- [readme] README lists sclink_net among the three main functions: "`sclink_net` for constructing the gene co-expression network by
scLink"
