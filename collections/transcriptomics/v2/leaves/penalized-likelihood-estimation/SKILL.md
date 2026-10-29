---
name: penalized-likelihood-estimation
description: Use when you have a robust correlation/covariance matrix from single-cell
  gene expression data and need to distinguish direct gene-gene associations from
  indirect ones via a sparse concentration matrix estimate.
when_to_use_negative:
- The goal is a raw pairwise co-expression summary rather than conditional (direct)
  dependencies — use sclink_cor alone instead of the network step.
- The gene list is very large; the article suggests first dividing genes into major
  modules based on scLink's correlation and applying scLink separately per module.
- The input is bulk-RNA-seq-like or already densely measured data, where bulk network
  methods may suffice; imputed single-cell data can also still be too sparse for reliable
  graphical model fitting.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: R package providing sclink_net, which computes the robust correlation matrix
    and fits the penalized Gaussian graphical model network
  repo: https://github.com/Vivianstats/scLink
- name: QUIC
  role: solver for the regularized Gaussian log-determinant (graphical lasso) optimization
    inside sclink_net
- name: R
  role: runtime environment in which the scLink package is installed and the QUIC-based
    network fitting, BIC selection, and edge export are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:2c036a417824dc64691b7569b264014596f48bc652cba85f6c095ad93c64e3f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - QUIC
  - R
  - scLink
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# penalized-likelihood-estimation

## Summary

Reconstructs sparse gene co-expression networks from single-cell expression data by maximizing a data-adaptively penalized Gaussian graphical model likelihood (weighted graphical lasso solved with QUIC), as implemented in scLink's sclink_net.

## When to use

Use when you have a robust correlation/covariance matrix from single-cell gene expression data and need to distinguish direct gene-gene associations from indirect ones via a sparse concentration matrix estimate.

## When NOT to use

- The goal is a raw pairwise co-expression summary rather than conditional (direct) dependencies — use sclink_cor alone instead of the network step.
- The gene list is very large; the article suggests first dividing genes into major modules based on scLink's correlation and applying scLink separately per module.
- The input is bulk-RNA-seq-like or already densely measured data, where bulk network methods may suffice; imputed single-cell data can also still be too sparse for reliable graphical model fitting.

## Inputs

- robust correlation matrix from the sclink_cor step (task_001)
- robust covariance matrix S for the p selected genes (e.g., 500 genes after sclink_norm with library size 10^6)
- vector of gene names used in downstream analysis

## Outputs

- sclink_sigma.csv: the estimated concentration matrix (Sigma) at the selected lambda
- sclink_edges.csv: nonzero off-diagonal edges of the symmetric adjacency matrix
- per-lambda summary of BIC and edge counts (nedge) used for model selection

## How to apply

First compute the data-adaptive penalty weight (1 - |r_j1j2|) for each gene pair from the robust correlation matrix so strongly co-expressed pairs receive weaker penalties. Formulate the penalized log-likelihood argmax over Theta ≻ 0 of logdet(Theta) - tr(S_p Theta) - lambda * sum_j1<j2 (1 - |r_j1j2|) * |theta_j1j2|, applying the positive-semidefinite correction S_p = S + |min{0, tau}| I to the robust covariance S. Solve with the QUIC solver over the regularization grid lda = seq(0.5, 0.1, -0.05) via sclink_net (ncores = 1). Compute BIC and edge counts (nedge) for each candidate lambda, select the lambda with minimum BIC, then extract the concentration matrix (Sigma) and symmetric adjacency matrix (adj) at the selected lambda.

## Related tools

- **scLink** (R package providing sclink_net, which computes the robust correlation matrix and fits the penalized Gaussian graphical model network) — https://github.com/Vivianstats/scLink
- **QUIC** (solver for the regularized Gaussian log-determinant (graphical lasso) optimization inside sclink_net)
- **R** (runtime environment in which the scLink package is installed and the QUIC-based network fitting, BIC selection, and edge export are executed)

## Examples

```
library(scLink); fit <- sclink_net(expr, type = 'count', scales = NULL, lda = seq(0.5, 0.1, -0.05), ncores = 1)
```

## Evaluation signals

- sclink_edges.csv exists and its row count matches the nonredundant off-diagonal nonzero entries of the adjacency matrix (adj)
- Selected lambda is the one with minimum BIC across the grid lda = seq(0.5, 0.1, -0.05)
- Resulting network is sparse: edge count <= 5% of the p(p-1)/2 possible edges
- Estimated concentration matrix Theta is positive definite (Optimizer converged with Theta ≻ 0)
- Edge weights in the penalty reflect (1 - |r_j1j2|), so pairs with strong robust correlation receive weaker penalties

## Limitations

- Edges identified by scLink are based on statistical dependencies and do not have directions, so regulatory vs. co-regulatory edge types cannot be distinguished from expression data alone.
- No software version for the scLink R package is specified in the article, hindering exact reproducibility.
- Gene filtering by detection rate or mean expression is recommended before network inference; unfiltered sparse genes can degrade the fit.

## Evidence

- [intro] penalized likelihood with data-adaptive weights: "scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge based on the observed co-expression"
- [methods] sclink_net computes network: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [intro] GGM objective: "scLink adapts the Gaussian graphical model [29] to distinguish direct associations between genes from indirect ones and leads to sparse networks"
- [discussion] edges undirected: "the edges identified by scLink are based on statistical dependencies and do not have directions"
- [readme] README quick start: [evidence span withheld — anchor status: external_doc]
