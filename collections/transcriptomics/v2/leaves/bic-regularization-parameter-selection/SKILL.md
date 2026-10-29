---
name: bic-regularization-parameter-selection
description: 'Use when fitting scLink''s weighted graphical lasso (QUIC) and a sparsity
  level must be chosen objectively: run sclink_net over lda = seq(0.5, 0.1, -0.05)
  and pick the lambda minimizing BIC from the summary output.'
when_to_use_negative:
- When the goal is exploratory inspection of correlation structure only — use sclink_cor
  instead of fitting penalized networks across a lambda grid
- When a sparsity level must be fixed a priori by design (e.g., benchmarking at a
  fixed edge budget) rather than selected by BIC
- When comparing methods evaluated at a pre-specified lambda — re-selecting by BIC
  would confound the comparison
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: sclink_net fits the penalized Gaussian graphical model over the lambda grid
    and returns per-lambda BIC, nedge, Sigma, and adj used for BIC-based selection
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Environment in which the scLink R package is installed from CRAN and the BIC
    comparison over the summary list is performed
  repo: https://github.com/Vivianstats/scLink
- name: QUIC
  role: Underlying solver for the weighted graphical lasso evaluated at each candidate
    lambda
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:1b6ad1d94a1c14e3111564fea4aacd5aeb00b895e314c990700d1c750c3b3794
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - QUIC
  - R
  - parallel
  - scLink
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bic-regularization-parameter-selection

## Summary

Selects the regularization parameter lambda for scLink's penalized Gaussian graphical model by fitting networks over a candidate lambda grid and choosing the model with minimum BIC, avoiding ad hoc tuning of network sparsity.

## When to use

Use when fitting scLink's weighted graphical lasso (QUIC) and a sparsity level must be chosen objectively: run sclink_net over lda = seq(0.5, 0.1, -0.05) and pick the lambda minimizing BIC from the summary output.

## When NOT to use

- When the goal is exploratory inspection of correlation structure only — use sclink_cor instead of fitting penalized networks across a lambda grid
- When a sparsity level must be fixed a priori by design (e.g., benchmarking at a fixed edge budget) rather than selected by BIC
- When comparing methods evaluated at a pre-specified lambda — re-selecting by BIC would confound the comparison

## Inputs

- Normalized single-cell expression matrix from sclink_norm (library size 10^6, log10 scale)
- Vector of candidate regularization parameters lda = seq(0.5, 0.1, -0.05)
- Optional gene list (e.g., the 51 lineage marker genes) restricting the network

## Outputs

- Per-lambda summary list with BIC and edge counts (nedge)
- Selected lambda (minimum BIC)
- Concentration matrix (Sigma) and adjacency matrix (adj) at the selected lambda

## How to apply

Fit scLink's penalized likelihood network with sclink_net (ncores=1) over the candidate regularization sequence lda = seq(0.5, 0.1, -0.05), where the penalty on each edge is weighted by (1 - |r_j1j2|) from scLink's robust correlations. For each candidate lambda, sclink_net returns a summary element containing the BIC and the edge count (nedge). Scan the summary list and select the lambda with the minimum BIC rather than fixing lambda by hand. Then extract the concentration matrix (Sigma) and symmetric adjacency (adj) at the selected lambda. This same grid-and-BIC procedure applies to ablation variants (glasso-r, glasso-f, plain glasso) so that method comparisons on simulated power-law or hub networks and real time-course data (e.g., GSE75748 across 0/12/24/36/72/96 h) are made at comparably selected sparsity.

## Related tools

- **scLink** (sclink_net fits the penalized Gaussian graphical model over the lambda grid and returns per-lambda BIC, nedge, Sigma, and adj used for BIC-based selection) — https://github.com/Vivianstats/scLink
- **R** (Environment in which the scLink R package is installed from CRAN and the BIC comparison over the summary list is performed) — https://github.com/Vivianstats/scLink
- **QUIC** (Underlying solver for the weighted graphical lasso evaluated at each candidate lambda)

## Examples

```
fit <- sclink_net(exprsclink, ngenes = nrow(exprsclink), lda = seq(0.5, 0.1, -0.05), ncores = 1); best <- which.min(sapply(fit$summary, function(s) s$BIC))
```

## Evaluation signals

- The summary element of sclink_net output contains a finite BIC and nedge for every lambda in seq(0.5, 0.1, -0.05)
- The selected lambda equals the argmin of BIC over the candidate grid
- The adjacency at the selected lambda is sparse: nedge <= 5% of p(p-1)/2 possible edges
- nedge matches the nonredundant off-diagonal nonzero entries of the selected adjacency matrix
- The same grid-BIC procedure applied across conditions (e.g., all six GSE75748 time points) yields a fitted network at each condition

## Limitations

- No software version for the scLink R package is specified in the article, so exact numerical BIC values may vary across package versions
- BIC selection is restricted to the candidate grid lda = seq(0.5, 0.1, -0.05); the true optimum may lie outside this range
- Selection depends on the upstream robust correlation/covariance step; a poor robust correlation matrix propagates into the BIC-selected network

## Evidence

- [full_text] Solve the optimization with the QUIC solver over the candidate regularization parameter grid lda = seq(0.5, 0.1, -0.05) via sclink_net in the scLink R package with ncores = 1.: [evidence span withheld — anchor status: not_found]
- [full_text] Compute BIC and edge counts (nedge) for each candidate lambda and select the lambda with minimum BIC from the summary list.: [evidence span withheld — anchor status: not_found]
- [intro] scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge based on the observed co-expression: "scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge based on the observed co-expression"
- [methods] use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [readme] scLink has three main functions: sclink_norm for pre-processing gene expression data, sclink_cor for calculating the co-expression matrix, sclink_net for constructing the gene co-expression network: "`sclink_net` for constructing the gene co-expression network by
scLink"
