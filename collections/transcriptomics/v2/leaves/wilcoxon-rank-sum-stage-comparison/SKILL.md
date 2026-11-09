---
name: wilcoxon-rank-sum-stage-comparison
description: Use when per-cell NFE (or similar potency) scores have been computed
  for cells grouped into developmental stages and you need a non-parametric statistical
  test of whether earlier-stage cells are systematically more potent than mature-stage
  cells.
when_to_use_negative:
- The input is already a cell-level gene expression matrix without any network/entropy
  transformation — apply c-CSN to build CCSN and compute NFE before testing
- Stages are not ordered along a differentiation trajectory and a two-sided or non-directional
  test is required
- The goal is clustering or dimension reduction (CNDM analyses), not potency comparison
  across stages
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: c-CSN
  role: Upstream tool that constructs the conditional cell-specific network (CCSN)
    per cell, from which the stochastic matrix P and NFE are derived before running
    the Wilcoxon rank-sum test
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:4d41c64fc2511e35d4e169d09f2ec681f92029928bed3506056288549b7dc684
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# wilcoxon-rank-sum-stage-comparison

## Summary

Statistical comparison of c-CSN-derived network flow entropy (NFE) scores between differentiation stages using one-sided Wilcoxon rank-sum tests, to verify that NFE is higher in progenitors (e.g. Wang Day 0/Day 1 vs Day 30 mature neurons) and thus resolves differentiation trajectory direction.

## When to use

Use when per-cell NFE (or similar potency) scores have been computed for cells grouped into developmental stages and you need a non-parametric statistical test of whether earlier-stage cells are systematically more potent than mature-stage cells.

## When NOT to use

- The input is already a cell-level gene expression matrix without any network/entropy transformation — apply c-CSN to build CCSN and compute NFE before testing
- Stages are not ordered along a differentiation trajectory and a two-sided or non-directional test is required
- The goal is clustering or dimension reduction (CNDM analyses), not potency comparison across stages

## Inputs

- Per-cell NFE scores computed from c-CSN/CSN weighted stochastic matrices for each cell in the dataset
- Stage labels for each cell (Wang NPC: Day 0/1/5/7/10/30; Yang mouse hepatoblast: E10/E17)
- Gene expression matrices (scRNA-seq) for the Wang and Yang datasets, normalized before CCSN construction

## Outputs

- Per-cell NFE scores with test statistics in a tabular result file (e.g. nfe_wilcoxon_results.csv)
- Wilcoxon rank-sum P values per dataset-stage comparison (expected: Wang P=2.062E-12, Yang P=3.756E-19)
- Stage-wise violin plots of NFE distributions (Figure 4-style)

## How to apply

First compute per-cell NFE = -sum(i,j) xi*pij*log(xi*pij) from the c-CSN-derived weighted stochastic matrix P with pij = xj*A*xi and sum_j pij = 1 (row-normalized) for each cell. Group cells by stage — Wang NPC dataset by Day 0, Day 1, Day 5, Day 7, Day 10, Day 30, and Yang mouse hepatoblast dataset by E10 vs E17 — with NFE expected higher in earlier/progenitor stages. Run one-sided Wilcoxon rank-sum tests comparing progenitors versus mature cells: Wang Day 0/Day 1 progenitors vs Day 30 mature neurons, and Yang E10 vs E17 cells. The one-sided alternative tests that progenitor NFE exceeds mature NFE, since high entropy reflects a more pluripotent network state. Tabulate per-cell NFE scores and test statistics, and visualize stage-wise distributions as violin plots (Figure 4-style). Verify reported agreement with published P = 2.062E-12 (Wang) and P = 3.756E-19 (Yang).

## Related tools

- **c-CSN** (Upstream tool that constructs the conditional cell-specific network (CCSN) per cell, from which the stochastic matrix P and NFE are derived before running the Wilcoxon rank-sum test) — https://github.com/LinLi-0909/c-CSN

## Examples

```
from scipy.stats import ranksums; stat, p = ranksums(nfe_progenitors, nfe_mature, alternative='greater')
```

## Evaluation signals

- Wilcoxon P value for Wang Day 0/Day 1 vs Day 30 should agree with the reported P = 2.062E-12
- Wilcoxon P value for Yang E10 vs E17 should agree with the reported P = 3.756E-19
- Mean/median NFE must be significantly higher in progenitors than in mature cells (Day 0/Day 1 > Day 30 for Wang; E10 > E17 for Yang), consistent with higher potency at earlier stages
- Result table must contain per-cell NFE scores and Wilcoxon test statistics with no missing stages
- Stage-wise violin plots should show monotone-ish decline of NFE along the Wang Day 0 -> Day 30 differentiation axis

## Limitations

- No parallel computation scheme is provided despite the G-fold computational cost increase of building per-cell CCSNs; the authors state it is desired
- c-CSN does not give directed (causal) gene association networks, so NFE-based trajectory direction inference relies on the entropy ordering rather than causal edge orientation
- NFE depends on the prior construction of a weighted stochastic matrix from the CCSN; errors in CCSN edge inference (over- or under-estimated conditional independence) propagate to NFE and thus to the test outcome

## Evidence

- [abstract] CCSN-based NFE helps resolving the direction of differentiation trajectories by quantifying the potency of each cell: "CCSN-based NFE helps resolving the direction of differentiation trajectories by quantifying the potency of each cell."
- [methods] NFE computation from stochastic matrix: "we normalize the weighted network as a stochastic matrix, P = (pij)"
- [intro] NFE estimates differentiation potency: "by defining the network flow entropy (NFE) on the gene–gene association data of each cell based on c-CSN, we estimate the differentiation potency of individual cells"
- [supplementary] c-CSN repository: "CCSN is available at https://github.com/LinLi-0909/c-CSN."
- [abstract] c-CSN method for CCSN construction: "we propose the c-CSN method, which can construct the conditional cell-specific network (CCSN) for each cell"
