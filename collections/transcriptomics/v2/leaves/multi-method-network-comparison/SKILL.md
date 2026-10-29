---
name: multi-method-network-comparison
description: Use when a new differential network inference method must be validated
  against existing baselines on simulated gene regulatory networks with known ground
  truth, across multiple network complexity settings.
when_to_use_negative:
- No ground-truth network is available, so precision/recall/F1 on true edges cannot
  be computed (e.g. purely exploratory real-data analysis)
- The goal is a single-method application (e.g. inferring one disease-vs-control network)
  rather than a comparative benchmark across methods and complexity settings
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: BoostDiff
  role: Method under evaluation; run per simulated disease/control pair with n_estimators=50
  repo: https://github.com/scibiome/boostdiff_inference
- name: dcanr
  role: SimulatorGRN one-gene-knockdown scheme used to generate ground-truth GRNs
    and paired expression data
- name: minet
  role: Provides the ARACNE comparator implementation
- name: conda
  role: Creates the bdenv environment from environment.yml for installing BoostDiff
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbae034
    title: Inference of differential gene regulatory networks using boosted differential
      trees
  source_content_hash: sha256:04c513fb535f5eadba8faf0e987cc13ab74ab79bf83c6fc09d6eacb820c6cf19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scibiome/boostdiff_inference
  tools:
  - BoostDiff
  - Python
  - R
  - conda
  - dcanr
  - minet
  - pip
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# multi-method-network-comparison

## Summary

Benchmark BoostDiff against nine comparator differential network inference methods (GENIE3, ARACNE via minet, RF Difftrees, Differential GENIE3, z-score, EBcoexpress, Diffcoex, GGM, chNet) on simulated ground-truth GRNs, ranking predicted edges and scoring precision/recall/F1 on the top-100 edges per network-size setting.

## When to use

Use when a new differential network inference method must be validated against existing baselines on simulated gene regulatory networks with known ground truth, across multiple network complexity settings.

## When NOT to use

- No ground-truth network is available, so precision/recall/F1 on true edges cannot be computed (e.g. purely exploratory real-data analysis)
- The goal is a single-method application (e.g. inferring one disease-vs-control network) rather than a comparative benchmark across methods and complexity settings

## Inputs

- Simulated ground-truth GRNs (50/150/300/500 nodes) with disease/control expression matrices from dcanr SimulatorGRN one-gene-knockdown scheme (100 samples per condition)
- Disease and control expression matrices in BoostDiff tab-separated format (rows = genes, columns = samples, first column named "Gene")
- Predicted edge lists (ranked) from each of the nine comparator differential network methods

## Outputs

- benchmark_performance.csv with precision, recall, and F1 per method per network-size setting
- Per-method rankings of the top-100 predicted edges per simulation

## How to apply

Simulate ground-truth GRNs and paired disease/control expression data with the dcanr SimulatorGRN one-gene-knockdown scheme (100 samples per condition) for 50-, 150-, and 300-node networks with 500 simulations each and a 500-node network with 250 simulations. Run the candidate method (e.g. BoostDiff with the recommended low ensemble size n_estimators=50 to avoid overfitting) and all nine comparator methods on each simulated disease/control pair. Rank predicted edges per method and evaluate inference performance on the top-100 edges, computing precision, recall, and F1 per method per network-size setting, writing results to benchmark_performance.csv. The key decision point is method ranking per complexity setting: the reported pattern is RF Difftrees leading in the 50-node setting while BoostDiff leads in the 150-, 300-, and 500-node settings, so conclusions should be drawn per setting rather than averaged across all sizes.

## Related tools

- **BoostDiff** (Method under evaluation; run per simulated disease/control pair with n_estimators=50) — https://github.com/scibiome/boostdiff_inference
- **dcanr** (SimulatorGRN one-gene-knockdown scheme used to generate ground-truth GRNs and paired expression data)
- **minet** (Provides the ARACNE comparator implementation)
- **conda** (Creates the bdenv environment from environment.yml for installing BoostDiff)

## Examples

```
conda create --name bdenv --file environment.yml && conda activate bdenv && pip install . ; python -c "from boostdiff.main_boostdiff import BoostDiff; model.run(file_etha, file_ctrl, output_folder, n_estimators=50, n_features, n_subsamples, n_processes=n_processes)"
```

## Evaluation signals

- benchmark_performance.csv reproduces the reported pattern: RF Difftrees is the leading method in the 50-node setting while BoostDiff leads in the 150-, 300-, and 500-node settings
- Precision, recall, and F1 are computed for all ten methods (BoostDiff + nine comparators) in all four complexity settings
- Each simulation uses 100 samples per condition, with 500 simulations for the 50/150/300-node networks and 250 simulations for the 500-node network
- Edge evaluation is restricted to the top-100 ranked edges per method per simulation
- BoostDiff runs use the recommended low ensemble size n_estimators=50 to avoid overfitting

## Limitations

- Method ranking is complexity-dependent: BoostDiff does not win in the simplest 50-node setting, so single-setting benchmarks can be misleading
- Running BoostDiff on large real datasets (e.g. the Crohn's disease dataset GSE126124) can require a high-performance computing cluster
- BoostDiff was tested on Linux only

## Evidence

- [results] RF Difftrees (random forest of differential trees) is the top performer in the 50-gene setting, while BoostDiff is the best performing method in the more complex 150-, 300-, and 500-gene settings.: [evidence span withheld — anchor status: not_found]
- [abstract] BoostDiff outperforms existing differential network methods on simulated data evaluated in four different complexity settings: "BoostDiff outperforms existing differential network methods on simulated data evaluated in four different complexity settings."
- [readme] Recommendation to use a low ensemble size on real datasets: "we recommend to set a relatively low number of base learners, e.g. set `n_estimators=50`"
- [readme] BoostDiff tested on linux only: "Note: BoostDiff was tested on linux."
