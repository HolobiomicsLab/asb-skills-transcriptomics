---
name: scrna-seq-expression-normalization
description: Use when raw scRNA-seq count matrices (e.g. Wang NPC 483-cell differentiation
  or Yang E10/E17 hepatoblast datasets) need normalized expression profiles before
  per-cell network or entropy computations.
when_to_use_negative:
- When the input is already a feature/degree matrix (e.g. a CNDM derived from CCSNs),
  not raw per-cell gene expression
- When the analysis goal is bulk RNA-seq normalization, since the method is designed
  for single-cell expression matrices with per-cell gene values
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: c-CSN
  role: Consumes the normalized expression profiles to build per-cell CCSN/CSN networks
    and compute NFE; normalization conditions the edge weights used downstream
  repo: https://github.com/LinLi-0909/c-CSN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.05.005
    title: 'c-CSN: Single-Cell RNA Sequencing Data Analysis by Conditional Cell-Specific
      Network'
  source_content_hash: sha256:c6fb4054ee51a66d4b74a0ffa13f9db2265deb04bf445db8b0bc70c8ab140394
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LinLi-0909/c-CSN
  tools:
  - c-CSN
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# scrna-seq-expression-normalization

## Summary

Normalizes raw scRNA-seq gene expression profiles so that per-cell gene expression values can be consistently compared across cells before downstream analyses such as CCSN construction, dimension reduction, or NFE-based potency estimation. In the c-CSN workflow it is the step applied immediately after loading the Wang NPC and Yang mouse hepatoblast datasets.

## When to use

Use when raw scRNA-seq count matrices (e.g. Wang NPC 483-cell differentiation or Yang E10/E17 hepatoblast datasets) need normalized expression profiles before per-cell network or entropy computations.

## When NOT to use

- When the input is already a feature/degree matrix (e.g. a CNDM derived from CCSNs), not raw per-cell gene expression
- When the analysis goal is bulk RNA-seq normalization, since the method is designed for single-cell expression matrices with per-cell gene values

## Inputs

- Raw Wang NPC differentiation scRNA-seq count matrix (483 cells; Day 0, Day 1, Day 5, Day 7, Day 10, Day 30)
- Raw Yang mouse hepatoblast scRNA-seq count matrix (E10 vs E17)

## Outputs

- Normalized gene expression profile matrix per dataset, suitable for CCSN/CSN edge weighting (pij proportional to xi*xj)

## How to apply

Load the raw scRNA-seq datasets — the Wang NPC differentiation dataset (483 cells across Day 0, Day 1, Day 5, Day 7, Day 10, Day 30) and the Yang mouse hepatoblast dataset (E10 vs E17) — and normalize the gene expression profiles so per-cell values are comparable. In the c-CSN pipeline this normalization feeds directly into the per-cell background networks built with the c-CSN approach (CCSNs for Wang cells with one conditional gene; unconditional CSNs for Yang cells), where each gene pair is tested for conditional independence against a standard-normal quantile at the chosen significance level. Downstream NFE computation requires weights pij proportional to the product of the normalized expression of connected genes (pij proportional to xi*xj, with sum_j pij = 1), so normalization directly conditions the stochastic matrix P used for entropy. Judgment: normalized profiles should yield NFE distributions that separate progenitors (Wang Day 0/Day 1, Yang E10) from mature cells (Day 30 neurons, E17) with the reported Wilcoxon P values.

## Related tools

- **c-CSN** (Consumes the normalized expression profiles to build per-cell CCSN/CSN networks and compute NFE; normalization conditions the edge weights used downstream) — https://github.com/LinLi-0909/c-CSN

## Examples

```
python scripts/build_ccsn.py --input wang_npc_counts.csv --stages Day_0,Day_1,Day_5,Day_7,Day_10,Day_30 --normalize
```

## Evaluation signals

- Per-cell NFE scores computed on normalized Wang Day 0/Day 1 progenitors must exceed those of Day 30 mature neurons (one-sided Wilcoxon rank-sum P = 2.062E-12 as reported)
- Per-cell NFE scores computed on normalized Yang E10 cells must exceed those of E17 cells (reported P = 3.756E-19)
- Row-normalized weighted network P must satisfy sum_j pij = 1 for every cell before NFE computation
- All 483 Wang cells and both Yang stages (E10, E17) must be retained after normalization with per-stage labels intact for violin-plot comparison

## Limitations

- Normalization itself does not eliminate dropout noise; the article notes CSN overestimation and c-CSN is proposed to address indirect associations in the downstream network step
- No parallel computation scheme is provided despite the G-fold computational cost increase from conditioning on multiple genes; the authors state it is desired

## Evidence

- [methods] Load the Wang NPC differentiation scRNA-seq dataset (483 cells; stages Day 0, Day 1, Day 5, Day 7, Day 10, Day 30) and the Yang mouse hepatoblast dataset (E10 vs E17), and normalize the gene expression profiles.: [evidence span withheld — anchor status: not_found]
- [methods] CCSN Czg(k) is obtained for cell k given conditional gene zg: "CCSN Czg(k) is obtained for cell k given conditional gene zg"
- [methods] we normalize the weighted network as a stochastic matrix, P = (pij): "we normalize the weighted network as a stochastic matrix, P = (pij)"
