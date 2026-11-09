---
name: stability-curve-benchmark-comparison
description: 'Decision support for choosing how many reference genes to aggregate
  in qPCR normalization: when you need to compare SD-vs-k stability curves of a weighted
  (geom_sd_hybrid) versus unweighted (geom) virtual RG to find the crossover point.'
when_to_use_negative:
- When the experiment uses only 2-3 preselected internal controls (e.g. RNU48, hsa-miR-16-5p)
  — use calcWeight on raw CT values instead of iterating over combination sizes
- When the goal is to select the best combination within a fixed k rather than to
  determine how many reference genes to aggregate — the benchmarking pipeline per-k
  sweep is unnecessary
- When analyzing non-qPCR normalized expression data where raw CT-based weighting
  assumptions do not hold
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: InterOpt
  role: R package providing run_experiment to evaluate geom and geom_sd_hybrid weighting
    of reference gene combinations and compute SD of the resulting virtual RGs
  repo: https://github.com/asalimih/InterOpt
- name: devtools
  role: Used to install the InterOpt package from GitHub via devtools::install_github
  repo: https://github.com/asalimih/InterOpt
- name: R
  role: Runtime environment in which the benchmarking pipeline is executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2023.107945
    title: 'InterOpt: Improved gene expression quantification in qPCR experiments
      using weighted aggregation of reference genes'
  source_content_hash: sha256:3e070c33b8612ff403c2842cca9ac0cd401389664f47ce19e069434ca41b7b09
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/asalimih/InterOpt
  tools:
  - InterOpt
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# stability-curve-benchmark-comparison

## Summary

Benchmarks the geom(sd) weighted virtual reference gene against the usual (unweighted) geometric mean across increasing numbers of aggregated reference genes (k=2 to 20), quantifying where the weighting advantage disappears. This reproduces the InterOpt finding that geom(sd) is applicable to up to five or six reference genes.

## When to use

Decision support for choosing how many reference genes to aggregate in qPCR normalization: when you need to compare SD-vs-k stability curves of a weighted (geom_sd_hybrid) versus unweighted (geom) virtual RG to find the crossover point.

## When NOT to use

- When the experiment uses only 2-3 preselected internal controls (e.g. RNU48, hsa-miR-16-5p) — use calcWeight on raw CT values instead of iterating over combination sizes
- When the goal is to select the best combination within a fixed k rather than to determine how many reference genes to aggregate — the benchmarking pipeline per-k sweep is unnecessary
- When analyzing non-qPCR normalized expression data where raw CT-based weighting assumptions do not hold

## Inputs

- Raw CT qPCR expression matrix (genes x samples), e.g. data_GSE78870 or data_GSE50013 bundled with InterOpt
- Corresponding sample group annotations (groups_GSE78870, groups_GSE50013)
- Reference gene combination lists per k (top 400 RGk combinations by lowest SD) generated iteratively

## Outputs

- stability_vs_nrgs.csv containing per-d (number of aggregated reference genes) SD values for geom(sd) and geom methods on both datasets
- Identified diminishing-benefit threshold (crossover d, expected ~5-6 reference genes)

## How to apply

Load the InterOpt package in R with the bundled datasets data_GSE78870/groups_GSE78870 and data_GSE50013/groups_GSE50013. Starting at k=2, iteratively build larger reference gene sets by crossing the top 400 combinations of RGk with the lowest SD with the remaining genes to form RGk+1, up to k=20, calling run_experiment with weight_methods=c('geom','geom_sd_hybrid'). For each k, record the SD of the resulting virtual RG under geom(sd) weighting and under the regular geometric mean on both datasets, using the same normalization (e.g. norm_method='high_exp', norm_method_exp_thr=35) as the benchmarking pipeline. Compare the two stability curves as the number of aggregated reference genes d increases and identify the d beyond which geom(sd) no longer improves on the geometric mean — expect the crossover at roughly 5-6 reference genes. The rationale is that weights optimized on raw CT values show over-fitting beyond this point, so the weighted aggregation's benefit diminishes with larger k.

## Related tools

- **InterOpt** (R package providing run_experiment to evaluate geom and geom_sd_hybrid weighting of reference gene combinations and compute SD of the resulting virtual RGs) — https://github.com/asalimih/InterOpt
- **devtools** (Used to install the InterOpt package from GitHub via devtools::install_github) — https://github.com/asalimih/InterOpt
- **R** (Runtime environment in which the benchmarking pipeline is executed)

## Examples

```
library(InterOpt); result = run_experiment(data_source=data_GSE50013, gr_source=groups_GSE50013, ctVal_source=T, sub_names=mirs_for_combinations, norm_method='high_exp', norm_method_exp_thr=35, k=2, weight_methods=c('geom','geom_sd_hybrid'))
```

## Evaluation signals

- The SD-vs-d curves for geom_sd_hybrid and geom overlap or cross beyond d ≈ 5-6; geom(sd) shows no improvement past that threshold
- SD values for both methods are computed on normalized data (norm_method='high_exp', exp_thr=35), not raw CT values, for the benchmark comparison
- The iterative combination build respects the k+1 rule: top 400 lowest-SD RGk combinations crossed with remaining genes
- Output CSV contains per-k SD rows for both datasets (GSE78870 and GSE50013) and both weight methods
- geom(sd) shows better aggregated stability at small k (e.g. k=2), consistent with the paper's reported superiority of geom(sd) with raw-CT weights

## Limitations

- Aggregating more than 5 RGs by geom(sd) does not provide the expected benefits compared to the regular geometric mean (over-fitting pattern with raw CT-derived weights)
- RG stability measure depends on gene expression distribution assumptions; other distributions (like beta distribution) and better measures are suggested for further improvement
- Calculating Genorm/NormFinder stability measures additionally requires the InterOptCuda executable in PATH

## Evidence
