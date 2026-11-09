---
name: simulation-parameter-sweep
description: Use when benchmarking a method whose performance may depend on simulation
  settings (e.g. differential-expression level, rarity of a cell population) and multiple
  synthetic datasets with ground-truth labels are needed across a parameter grid.
when_to_use_negative:
- The input is real experimental data without ground-truth labels — a parameter sweep
  over simulated profiles provides no benchmarking signal there; use biological validation
  (e.g. known marker genes) instead.
- The method under test is insensitive to simulation parameters and a single simulated
  dataset suffices — the sweep's cost is not justified.
- The simulation tool cannot encode the factor being swept (e.g. no differential-expression
  parameter analogous to Splatter's de.prob), so the grid would not vary the relevant
  difficulty axis.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3308
tools:
- name: Splatter
  role: Generates simulated scRNA-seq expression profiles across the de.prob and rare-population-size
    grid with ground-truth labels
- name: GapClust
  role: Method under test; called on every simulated dataset using the inter-cluster
    gap statistic
  repo: github.com/fabotao/GapClust
- name: RaceID
  role: Competitor rare-cell detector run with default settings on the same simulated
    count matrices
- name: GiniClust
  role: Competitor rare-cell detector run with default settings on the same simulated
    count matrices
- name: CellSIUS
  role: Competitor rare-cell detector run with default settings on the same simulated
    count matrices
- name: FiRE
  role: Competitor rare-cell detector run with default settings on the same simulated
    count matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24489-8
    title: GapClust is a light-weight approach distinguishing rare cells from voluminous
      single cell expression profiles
  source_content_hash: sha256:8a972c023ff2f5fde0b2e2cfcf1373497c27d1537171ba9bee37284ec30fd9a8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellSIUS
  - FiRE
  - GapClust
  - GiniClust
  - RaceID
  - Splatter
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# simulation-parameter-sweep

## Summary

Systematically varies simulation parameters (e.g. Splatter de.prob from 0.1 to 0.8 and rare population size from 2 to 100 cells) to generate a grid of synthetic scRNA-seq datasets with known ground truth, enabling robust benchmarking of rare-cell detectors such as GapClust, RaceID, GiniClust, CellSIUS and FiRE.

## When to use

Use when benchmarking a method whose performance may depend on simulation settings (e.g. differential-expression level, rarity of a cell population) and multiple synthetic datasets with ground-truth labels are needed across a parameter grid.

## When NOT to use

- The input is real experimental data without ground-truth labels — a parameter sweep over simulated profiles provides no benchmarking signal there; use biological validation (e.g. known marker genes) instead.
- The method under test is insensitive to simulation parameters and a single simulated dataset suffices — the sweep's cost is not justified.
- The simulation tool cannot encode the factor being swept (e.g. no differential-expression parameter analogous to Splatter's de.prob), so the grid would not vary the relevant difficulty axis.

## Inputs

- Splatter simulation parameter grid (de.prob values 0.1-0.8; rare population size 2-100 cells)
- Simulated scRNA-seq count matrices with ground-truth rare-cell labels
- Default parameter configurations for each compared method (GapClust, RaceID, GiniClust, CellSIUS, FiRE)

## Outputs

- Per-dataset rare-cell calls from each method across the parameter grid
- Per-dataset, per-method F1 scores against ground-truth labels
- Aggregated F1 score tables/panels corresponding to Supplementary Fig. 2a-e
- Ranking of methods per simulation setting

## How to apply

Generate simulated expression profiles with Splatter, sweeping the differential expression probability (de.prob from 0.1 to 0.8, i.e. five scenarios) and the rare population size (2 to 100 cells), producing 99 simulated profiles per DE-probability scenario while retaining Splatter's ground-truth rare-cell labels. Run the method under test (GapClust, using its second-order-derivative-type distance score that exploits the inter-cluster gap) and all competitors (RaceID, GiniClust, CellSIUS, FiRE) with default settings on the same simulated count matrices. Score each method's called rare cells against the planted labels per dataset, compute the F1 score, and aggregate the scores into tables matching the panels of Supplementary Fig. 2a-e. The sweep's rationale is to show that conclusions (e.g. GapClust ranking best) hold across every simulated setting rather than at a single operating point.

## Related tools

- **Splatter** (Generates simulated scRNA-seq expression profiles across the de.prob and rare-population-size grid with ground-truth labels)
- **GapClust** (Method under test; called on every simulated dataset using the inter-cluster gap statistic) — github.com/fabotao/GapClust
- **RaceID** (Competitor rare-cell detector run with default settings on the same simulated count matrices)
- **GiniClust** (Competitor rare-cell detector run with default settings on the same simulated count matrices)
- **CellSIUS** (Competitor rare-cell detector run with default settings on the same simulated count matrices)
- **FiRE** (Competitor rare-cell detector run with default settings on the same simulated count matrices)

## Examples

```
devtools::install_github("fabotao/GapClust"); sim <- splatSimulate(params, group.prob = c(0.98, 0.02), de.prob = 0.1); GapClust::GapClust(data = counts(sim))
```

## Evaluation signals

- Each of the five DE-probability scenarios contains 99 simulated datasets, with rare population size spanning 2-100 cells — check the grid is fully populated before aggregation.
- Ground-truth rare-cell labels must be carried through from Splatter generation into the F1 computation for every dataset.
- All competitors must be run on the identical simulated count matrices with default settings so F1 differences reflect the method, not the data.
- Expected result invariant: GapClust achieves the best F1 score among the five methods in every simulation setting, reproducing the pattern of Supplementary Fig. 2a-e.
- F1 scores should fall in the [0,1] range and be reported per dataset per method before any aggregation.

## Limitations

- Conclusions are tied to Splatter's simulation model (DE-probability and rare-population-size axes); other simulator settings or data characteristics are not covered by this sweep.
- Competitor methods were run with default settings, so results may not reflect tuned best-case performance of RaceID, GiniClust, CellSIUS or FiRE.
- Only the rare-cell detection F1 axis is swept; computational efficiency comparisons (time and memory) come from separate experiments, not this sweep.

## Evidence

- [results] Across the five Splatter simulation scenarios, GapClust provides the best rare cell detection performance among all compared methods in every simulation setting, as evaluated by F1 score.: [evidence span withheld — anchor status: not_found]
- [supplementary] Sweep of de.prob and rare population size: [evidence span withheld — anchor status: not_found]
- [intro] GapClust mechanism grounding: "A second-order derivative type distance score capturing the variation of distance changes is proposed to fully utilise this gap information."
- [readme] README performance claim: "In terms of rare cell detection, GapClust offers best performance compared with other methods, quantified by F<sub>1</sub> score."
- [supplementary] Default-settings comparison and aggregation: [evidence span withheld — anchor status: not_found]
