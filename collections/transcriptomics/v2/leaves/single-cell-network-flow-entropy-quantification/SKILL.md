---
name: single-cell-network-flow-entropy-quantification
description: Use when scRNA-seq data along a differentiation trajectory (e.g. progenitors
  vs mature cells) must be ordered by potency of individual cells, using per-cell
  network entropy rather than expression-only scores.
when_to_use_negative:
- The goal is clustering or dimension reduction rather than potency estimation — the
  article provides separate CNDM-based PCA/t-SNE and clustering workflows for those
  tasks
- Only a gene expression matrix is available and no per-cell CSN/CCSN can be constructed,
  since NFE is defined on the CSN/CCSN-based weighted stochastic network
- Data are not ordered along a differentiation trajectory, so there is no progenitor-vs-mature
  contrast to validate the potency direction
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: c-CSN
  role: Constructs the conditional cell-specific network (CCSN) per cell whose weighted
    stochastic matrix is the substrate for NFE computation
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:3b977f20288fdf49cbcd75ceda4e3418454d21fc392ffab74482e36302bf390d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# single-cell-network-flow-entropy-quantification

## Summary

Computes network flow entropy (NFE) on the weighted stochastic matrix derived from each cell's CSN/CCSN to quantify differentiation potency of individual cells and resolve the direction of differentiation trajectories from single-cell RNA-seq data.

## When to use

Use when scRNA-seq data along a differentiation trajectory (e.g. progenitors vs mature cells) must be ordered by potency of individual cells, using per-cell network entropy rather than expression-only scores.

## When NOT to use

- The goal is clustering or dimension reduction rather than potency estimation — the article provides separate CNDM-based PCA/t-SNE and clustering workflows for those tasks
- Only a gene expression matrix is available and no per-cell CSN/CCSN can be constructed, since NFE is defined on the CSN/CCSN-based weighted stochastic network
- Data are not ordered along a differentiation trajectory, so there is no progenitor-vs-mature contrast to validate the potency direction

## Inputs

- Normalized gene expression matrix of a scRNA-seq differentiation dataset (e.g. Wang NPC dataset, 483 cells, stages Day 0/1/5/7/10/30; Yang mouse hepatoblast E10 vs E17)
- Per-cell CSN/CCSN network from the c-CSN conditional-independence test
- Cell stage/condition labels for grouping cells into progenitor vs mature classes

## Outputs

- Per-cell NFE scores (network flow entropy per cell)
- Wilcoxon rank-sum test statistics and P values comparing NFE between progenitor and mature stages (nfe_wilcoxon_results.csv)
- Figure 4-style violin plots of NFE per differentiation stage

## How to apply

First construct the per-cell CCSN with c-CSN (e.g. one conditional gene chosen from top-degree hub genes for the Wang NPC dataset, unconditional CSNs for the Yang hepatoblast dataset), testing each gene pair with the conditional-independence hypothesis test where H0 is rejected when the normalized statistic exceeds the standard-normal quantile at the significance level. Then weight each edge proportional to the product of the normalized expression of the connected genes (pij ∝ xi*xj, sum_j pij = 1) and row-normalize the weighted network into the stochastic matrix P. Compute NFE = -sum(i,j) xi*pij*log(xi*pij) for every cell, yielding one potency score per cell. Validate potency ordering with one-sided Wilcoxon rank-sum tests comparing progenitors versus mature cells; the article expects higher NFE in progenitors, reproducing P = 2.062E-12 for Wang Day 0/Day 1 vs Day 30 and P = 3.756E-19 for Yang E10 vs E17. Tabulate per-cell NFE scores and test statistics (e.g. nfe_wilcoxon_results.csv) and draw stage-wise violin plots in the Figure 4 style.

## Related tools

- **c-CSN** (Constructs the conditional cell-specific network (CCSN) per cell whose weighted stochastic matrix is the substrate for NFE computation) — https://github.com/LinLi-0909/c-CSN

## Examples

```
from cCSN import build_ccsn, compute_nfe; A = build_ccsn(expr, cond_gene=top_hub_gene); P = row_normalize(A * expr); nfe = compute_nfe(P, expr); scipy.stats.mannwhitneyu(nfe[progenitors], nfe[mature], alternative='greater')
```

## Evaluation signals

- NFE values are higher in progenitors than mature cells (e.g. Wang Day 0/Day 1 > Day 30; Yang E10 > E17)
- Wilcoxon rank-sum P values agree with the reported benchmarks: P = 2.062E-12 (Wang) and P = 3.756E-19 (Yang)
- Row-normalized matrix P satisfies stochasticity: for each cell, sum_j pij = 1 with pij ∝ xi*xj
- One NFE score is produced per cell (483 cells in the Wang dataset), tabulated alongside test statistics in nfe_wilcoxon_results.csv
- The potency ordering resolves the direction of the differentiation trajectory ( progenitor → mature ) as expected

## Limitations

- c-CSN carries a G-fold computational cost increase for G conditional genes, and no parallel computation scheme is provided; the authors state 'a parallel computation scheme is desired to reduce the computation time'
- c-CSN does not yield directional (causal) gene associations: 'c-CSN is not designed to construct the causal gene association networks, and the directions of the gene associations cannot be obtained'

## Evidence

- [abstract] CCSN-based NFE helps resolving the direction of differentiation trajectories by quantifying the potency of each cell.: "CCSN-based NFE helps resolving the direction of differentiation trajectories by quantifying the potency of each cell."
- [methods] we normalize the weighted network as a stochastic matrix, P = (pij): "we normalize the weighted network as a stochastic matrix, P = (pij)"
- [methods] we developed NFE to estimate the differentiation potency of a cell from its CSN or CCSN: "we developed NFE to estimate the differentiation potency of a cell from its CSN or CCSN"
- [methods] we choose the top GG≥1 largest 'importance' genes as the conditional genes: [evidence span withheld — anchor status: not_found]
- [discussion] a parallel computation scheme is desired to reduce the computation time: "a parallel computation scheme is desired to reduce the computation time"
