---
name: differentiation-potency-violin-visualization
description: Use when single-cell differentiation datasets (e.g. progenitor vs. mature
  stages) need a per-cell potency score to orient trajectory direction, and NFE distributions
  must be compared across stages with non-parametric statistics.
when_to_use_negative:
- The goal is clustering or dimension reduction rather than potency/trajectory direction
  — use the CNDM-based PCA/t-SNE/clustering workflow instead
- Data lack stage or time-point labels, so progenitor-vs-mature groups for the Wilcoxon
  test and violin plots cannot be defined
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: c-CSN
  role: Constructs the per-cell CCSN whose weighted stochastic matrix is the basis
    of the NFE potency score
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:5bdacb3739a8b05d6b7539ff7d24329c1bb5567fc0f2258a3cee967b288e91e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# differentiation-potency-violin-visualization

## Summary

Estimates differentiation potency of individual scRNA-seq cells by computing network flow entropy (NFE) on c-CSN-derived weighted stochastic networks, then visualizes per-stage NFE distributions as violin plots and validates trajectory direction with one-sided Wilcoxon rank-sum tests.

## When to use

Use when single-cell differentiation datasets (e.g. progenitor vs. mature stages) need a per-cell potency score to orient trajectory direction, and NFE distributions must be compared across stages with non-parametric statistics.

## When NOT to use

- The goal is clustering or dimension reduction rather than potency/trajectory direction — use the CNDM-based PCA/t-SNE/clustering workflow instead
- Data lack stage or time-point labels, so progenitor-vs-mature groups for the Wilcoxon test and violin plots cannot be defined

## Inputs

- Normalized scRNA-seq gene expression matrix with stage/time-point labels (e.g. Wang NPC dataset, 483 cells at Day 0/1/5/7/10/30; Yang mouse hepatoblast E10 vs E17)
- Per-cell CSN or CCSN adjacency from the c-CSN conditional-independence test (c-CSN for Wang with one conditional gene; unconditional CSN for Yang)

## Outputs

- Per-cell NFE scores
- Wilcoxon rank-sum statistics and P values per stage comparison (nfe_wilcoxon_results.csv)
- Figure 4-style violin plots of NFE per differentiation stage

## How to apply

Build the per-cell CCSN with c-CSN (conditional-independence hypothesis test; reject H0 when the normalized statistic exceeds the standard-normal quantile at the chosen significance level), then weight each edge proportionally to the product of the normalized expression of the connected genes (pij proportional to xi*xj with sum_j pij = 1) and row-normalize into the stochastic matrix P. Compute NFE = -sum(i,j) xi*pij*log(xi*pij) for every cell; progenitor-stage cells (e.g. Wang Day 0/Day 1 NPCs, Yang E10 hepatoblasts) should show higher entropy than mature cells (Wang Day 30 neurons, Yang E17). Run one-sided Wilcoxon rank-sum tests on the two stage groups and tabulate per-cell NFE scores and statistics (expected P = 2.062E-12 for Wang, 3.756E-19 for Yang). Render Figure 4-style violin plots of NFE per differentiation stage and export results (e.g. nfe_wilcoxon_results.csv) so potency ordering and direction of differentiation can be read directly from the plot.

## Related tools

- **c-CSN** (Constructs the per-cell CCSN whose weighted stochastic matrix is the basis of the NFE potency score) — https://github.com/LinLi-0909/c-CSN

## Examples

```
python compute_nfe.py --wang wang_normalized.csv --yang yang_normalized.csv --test wilcoxon --output nfe_wilcoxon_results.csv --violin figure4_violin.png
```

## Evaluation signals

- NFE is higher in progenitors than mature cells (Wang Day 0/Day 1 > Day 30; Yang E10 > E17)
- Wilcoxon P values reproduce the reported values: P = 2.062E-12 (Wang) and P = 3.756E-19 (Yang)
- Row-normalized stochastic matrix satisfies sum_j pij = 1 before entropy computation
- nfe_wilcoxon_results.csv contains per-cell NFE scores plus test statistics for every stage comparison
- Violin plots show per-stage NFE distributions consistent with the Figure 4 layout

## Limitations

- c-CSN's G-fold conditional-gene loop increases computation time and no parallel computation scheme is provided
- c-CSN does not yield causal/directed gene associations, so NFE indicates potency but not regulatory direction
- Findings were demonstrated on the Wang NPC and Yang hepatoblast datasets; transfer to other tissues or sequencing platforms is not established here

## Evidence

- [methods] NFE computation from CSN/CCSN-based weighted stochastic network: "we normalize the weighted network as a stochastic matrix, P = (pij)"
- [methods] Differentiation potency estimation via NFE: "we developed NFE to estimate the differentiation potency of a cell from its CSN or CCSN"
- [abstract] NFE resolves trajectory direction: "CCSN-based NFE helps resolving the direction of differentiation trajectories by quantifying the potency of each cell."
- [methods] Wilcoxon comparison datasets: "In Yang dataset [44], we compared the cells at embryonic day 10 (E10) with those at embryonic day 17 (E17)"
