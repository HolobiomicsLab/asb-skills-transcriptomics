---
name: stochastic-network-edge-weight-normalization
description: Use when a per-cell gene-gene network (e.g. from c-CSN) must be converted
  into a stochastic flow matrix and summarized into a single entropy-based potency
  score per cell, e.g. to order Wang NPC Day 0-30 or Yang E10/E17 differentiation
  stages.
when_to_use_negative:
- The goal is clustering or dimension reduction of cells — those use the CNDM (conditional
  network degree matrix) with PCA/t-SNE and K-means/SC3/SIMLR, not NFE on stochastic
  matrices.
- Input is raw (unnormalized) gene expression without an associated per-cell network;
  edge weights must be derived from a CSN/CCSN, not directly from the expression matrix.
- The analysis targets mature cells only with no progenitor comparison, since NFE
  resolves potency by contrasting entropy across differentiation stages.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0092
tools:
- name: c-CSN
  role: Builds the conditional cell-specific network (CCSN) per cell that supplies
    the adjacency from which stochastic edge weights and NFE are computed
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:5de4533739af6bcac025baea994c24c99b6f40b37f7ca0fa1f73219eec53f1bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# stochastic-network-edge-weight-normalization

## Summary

Converts a cell-specific gene-gene network (CSN/CCSN) into a weighted stochastic matrix P whose rows sum to 1, then computes network flow entropy (NFE) = -sum(i,j) xi*pij*log(xi*pij) as a per-cell differentiation-potency score. Higher NFE identifies more potent (progenitor) cells along differentiation trajectories.

## When to use

Use when a per-cell gene-gene network (e.g. from c-CSN) must be converted into a stochastic flow matrix and summarized into a single entropy-based potency score per cell, e.g. to order Wang NPC Day 0-30 or Yang E10/E17 differentiation stages.

## When NOT to use

- The goal is clustering or dimension reduction of cells — those use the CNDM (conditional network degree matrix) with PCA/t-SNE and K-means/SC3/SIMLR, not NFE on stochastic matrices.
- Input is raw (unnormalized) gene expression without an associated per-cell network; edge weights must be derived from a CSN/CCSN, not directly from the expression matrix.
- The analysis targets mature cells only with no progenitor comparison, since NFE resolves potency by contrasting entropy across differentiation stages.

## Inputs

- Per-cell CSN or CCSN adjacency matrix (from the c-CSN conditional-independence hypothesis test on gene pairs)
- Normalized gene expression profile of each cell (e.g. Wang NPC scRNA-seq, 483 cells across Day 0/1/5/7/10/30; Yang mouse hepatoblast E10/E17)

## Outputs

- Per-cell NFE scores (one entropy value per cell)
- Row-normalized stochastic matrix P for each cell's weighted network
- Wilcoxon rank-sum test statistics and P values comparing NFE between progenitor and mature stages (e.g. nfe_wilcoxon_results.csv; expected P = 2.062E-12 Wang, P = 3.756E-19 Yang)

## How to apply

Starting from the adjacency/CCSN of a cell, weight each edge proportional to the product of the normalized expression of the connected genes (pij proportional to xi*xj, with sum_j pij = 1), then row-normalize the weighted network into the stochastic matrix P with pij = xj*A*xi. Compute NFE = -sum(i,j) xi*pij*log(xi*pij) for every cell using the normalized gene expression profile. Compare NFE across stages with one-sided Wilcoxon rank-sum tests (e.g. Wang Day 0/Day 1 progenitors vs Day 30 mature neurons; Yang E10 vs E17). The rationale is that progenitor cells have higher network flow entropy, so NFE quantifies differentiation potency and resolves trajectory direction without pseudotime inference.

## Related tools

- **c-CSN** (Builds the conditional cell-specific network (CCSN) per cell that supplies the adjacency from which stochastic edge weights and NFE are computed) — https://github.com/LinLi-0909/c-CSN

## Examples

```
# in Python after c-CSN: row-normalize weighted adjacency and compute NFE
P = A * (x[:, None] * x[None, :]); P = P / P.sum(axis=1, keepdims=True); nfe = -np.sum(x[:, None] * P * np.log(x[:, None] * P + 1e-12))
```

## Evaluation signals

- Each row of the stochastic matrix P sums to 1 (sum_j pij = 1) after row normalization.
- Edge weights are proportional to the product of normalized expression of connected genes (pij proportional to xi*xj).
- Wilcoxon rank-sum P values on Wang and Yang datasets agree with the reported P = 2.062E-12 (Wang Day 0/Day 1 vs Day 30) and P = 3.756E-19 (Yang E10 vs E17).
- NFE is systematically higher in progenitors (Wang Day 0/Day 1, Yang E10) than in mature cells (Day 30 neurons, E17 hepatoblasts).
- Violin plots per differentiation stage reproduce the Figure 4 pattern of decreasing NFE along the trajectory.

## Limitations

- NFE is an association-based (not causal) network summary: c-CSN does not construct causal gene association networks, and directions of gene associations cannot be obtained.
- Constructing CCSNs with G conditional genes incurs a G-fold computational cost increase and no parallel computation scheme is provided; the authors state a parallel scheme is desired.
- Potency interpretation depends on having labeled stage groups (e.g. Day 0 vs Day 30, E10 vs E17) for the one-sided Wilcoxon comparison.

## Evidence

- [abstract] CCSN-based NFE helps resolving the direction of differentiation trajectories by quantifying the potency of each cell.: "CCSN-based NFE helps resolving the direction of differentiation trajectories by quantifying the potency of each cell."
- [methods] we normalize the weighted network as a stochastic matrix, P = (pij): "we normalize the weighted network as a stochastic matrix, P = (pij)"
- [methods] we developed NFE to estimate the differentiation potency of a cell from its CSN or CCSN: "we developed NFE to estimate the differentiation potency of a cell from its CSN or CCSN"
- [intro] by defining the network flow entropy (NFE) on the gene-gene association data of each cell based on c-CSN, we estimate the differentiation potency of individual cells: [evidence span withheld — anchor status: not_found]
