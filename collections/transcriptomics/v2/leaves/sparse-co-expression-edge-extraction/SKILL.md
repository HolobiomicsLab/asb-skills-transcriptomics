---
name: sparse-co-expression-edge-extraction
description: Use when single-cell gene expression data has been normalized (library
  size 10^6), genes filtered to ~500 candidates, and a sparse gene co-expression network
  must distinguish direct from indirect gene associations via a Gaussian graphical
  model.
when_to_use_negative:
- 'Goal is to assign direction or edge type to interactions: the article states scLink''s
  edges ''do not have directions'' and it is not feasible to distinguish regulatory,
  co-regulatory, and functionally coordinated edge types using only gene expression
  data'
- 'Gene set is very large (p too big to optimize jointly): the article recommends
  first dividing genes into major modules based on scLink''s correlation and applying
  scLink separately to each module'
- 'Data are bulk rather than single-cell: imputed single-cell data may still be much
  sparser than bulk data, so bulk-data network methods (and this skill''s assumptions)
  may not transfer'
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scLink
  role: Provides sclink_net, which calculates the robust correlation matrix and identifies
    the sparse co-expression network via the penalized likelihood with QUIC
  repo: https://github.com/Vivianstats/scLink
- name: scLink (R package)
  role: R implementation installed via install.packages('scLink'); sclink_cor can
    be run first to investigate correlation structures before network inference
  repo: https://github.com/Vivianstats/scLink
- name: QUIC
  role: Optimizer used by sclink_net to solve the weighted graphical lasso objective
    over the candidate lambda grid
- name: R
  role: Execution environment for the scLink package and the BIC-based model selection
    and CSV export steps
  repo: https://github.com/Vivianstats/scLink
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:d89165a30d4dce32e7e217528eb3304a5d94f901c8a308063125951ccb209d74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - QUIC
  - R
  - scLink
  - scLink (R package)
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# sparse-co-expression-edge-extraction

## Summary

Reconstructs scLink's data-adaptive penalized-likelihood network inference: maximizing a log-determinant Gaussian graphical model objective (Eq. 8) with QUIC over a lambda grid, then selecting the sparse network by minimum BIC and extracting the concentration matrix and nonzero edges.

## When to use

Use when single-cell gene expression data has been normalized (library size 10^6), genes filtered to ~500 candidates, and a sparse gene co-expression network must distinguish direct from indirect gene associations via a Gaussian graphical model.

## When NOT to use

- Goal is to assign direction or edge type to interactions: the article states scLink's edges 'do not have directions' and it is not feasible to distinguish regulatory, co-regulatory, and functionally coordinated edge types using only gene expression data
- Gene set is very large (p too big to optimize jointly): the article recommends first dividing genes into major modules based on scLink's correlation and applying scLink separately to each module
- Data are bulk rather than single-cell: imputed single-cell data may still be much sparser than bulk data, so bulk-data network methods (and this skill's assumptions) may not transfer

## Inputs

- Normalized single-cell gene expression matrix (library size 10^6, ~500 filtered genes)
- Robust correlation matrix computed by sclink_cor (task_001 output)
- Regularization parameter grid lda = seq(0.5, 0.1, -0.05)

## Outputs

- Concentration matrix at the selected lambda (sclink_sigma.csv)
- Symmetric adjacency matrix (adj) and nonredundant off-diagonal edge list (sclink_edges.csv)
- BIC and edge counts (nedge) per candidate lambda

## How to apply

First compute the data-adaptive penalty weight (1 - |r_j1j2|) for each gene pair from the robust correlation matrix (from sclink_cor), so strongly co-expressed pairs receive weaker penalties. Formulate the penalized likelihood argmax over Theta succ 0 of logdet(Theta) - tr(S_p Theta) - lambda * sum (1-|r_j1j2|)*theta_j1j2, applying the positive-semidefinite correction S_p = S + |min{0,tau}|*I to the robust covariance S. Solve with the QUIC solver over the grid lda = seq(0.5, 0.1, -0.05) via sclink_net (ncores = 1). Compute BIC and edge count (nedge) for each candidate lambda, select the lambda with minimum BIC, then extract the concentration matrix (Sigma) and the symmetric adjacency matrix (adj); write the concentration matrix to sclink_sigma.csv and nonzero off-diagonal edges to sclink_edges.csv.

## Related tools

- **scLink** (Provides sclink_net, which calculates the robust correlation matrix and identifies the sparse co-expression network via the penalized likelihood with QUIC) — https://github.com/Vivianstats/scLink
- **scLink (R package)** (R implementation installed via install.packages('scLink'); sclink_cor can be run first to investigate correlation structures before network inference) — https://github.com/Vivianstats/scLink
- **QUIC** (Optimizer used by sclink_net to solve the weighted graphical lasso objective over the candidate lambda grid)
- **R** (Execution environment for the scLink package and the BIC-based model selection and CSV export steps) — https://github.com/Vivianstats/scLink

## Examples

```
net <- sclink_net(expr = norm_data, genes = genes, lda = seq(0.5, 0.1, -0.05), ncores = 1); summary(net)
```

## Evaluation signals

- sclink_edges.csv exists and its row count equals nedge at the selected lambda
- nedge matches the number of nonredundant off-diagonal nonzero entries of the adjacency matrix (adj)
- Selected network is sparse: edges <= 5% of the p(p-1)/2 possible edges
- Selected lambda is the one with minimum BIC across the grid seq(0.5, 0.1, -0.05)
- Concentration matrix Theta is positive definite and the sclink_sigma.csv dump matches the extracted Sigma

## Limitations

- Edges identified by scLink are based on statistical dependencies and do not have directions; regulatory vs co-regulatory vs functionally coordinated edge types cannot be distinguished from expression data alone
- No software version is specified for the scLink R package, hindering exact reproducibility
- For large gene lists, joint optimization may be impractical; the article suggests dividing genes into modules first based on scLink's correlation

## Evidence

- [intro] GGM objective with adaptive weights: "scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge based on the observed co-expression"
- [methods] sclink_net invocation: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [methods] exploratory correlation step for large gene lists: "first use `sclink_cor` to investigate the correlation structures among these genes"
- [discussion] undirected edges limitation: "the edges identified by scLink are based on statistical dependencies and do not have directions"
- [readme] package availability and installation: [evidence span withheld — anchor status: not_found]
