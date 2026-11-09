---
name: aucroc-benchmark-evaluation
description: Use when evaluating a cell-proportion or differential-abundance method's
  performance across simulated dataset sizes — e.g. choosing how many pseudo-replicates
  Scanpro's bootstrapping should use for a given total cell count per condition.
when_to_use_negative:
- The dataset is real (not simulated) without known differential-proportion ground
  truth — auROC cannot be computed; use biological interpretation and validation instead
- The goal is to run differential proportion analysis on a user dataset rather than
  benchmark method parameters — call scanpro/run_scanpro directly with a fixed pseudo-replicate
  count
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_2229
tools:
- name: Scanpro
  role: Provides sim_scanpro for simulating replicate datasets, run_scanpro/anova/t_test
    for bootstrapped differential proportion calls, and out.plot for benchmark panels
  repo: https://github.com/loosolab/scanpro
- name: Python
  role: Execution environment for the simulation and benchmarking pipeline
- name: Scanpy
  role: Single-cell framework Scanpro integrates with; input data handled via Scanpy/AnnData
- name: AnnData
  role: Native input format for Scanpro (read via ad.read_h5ad)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:6df63d4b7daade553508ee841edcbce7f7c477171dd41fa18a5e10d86bc999ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - AnnData
  - Python
  - Scanpro
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# aucroc-benchmark-evaluation

## Summary

Benchmark a proportion-analysis method by simulating replicated single-cell datasets with known ground truth, sweeping parameters (pseudo-replicate counts, total cells per condition), and scoring sensitivity, false-positive rate, accuracy, and auROC against the simulated truth.

## When to use

Use when evaluating a cell-proportion or differential-abundance method's performance across simulated dataset sizes — e.g. choosing how many pseudo-replicates Scanpro's bootstrapping should use for a given total cell count per condition.

## When NOT to use

- The dataset is real (not simulated) without known differential-proportion ground truth — auROC cannot be computed; use biological interpretation and validation instead
- The goal is to run differential proportion analysis on a user dataset rather than benchmark method parameters — call scanpro/run_scanpro directly with a fixed pseudo-replicate count

## Inputs

- AnnData object (.h5ad) with clustered single-cell data (samples_col, clusters_col, conds_col annotations)
- Original replicate datasets (e.g. 4 replicates) used as the basis for simulation
- Sweep specification: pseudo-replicate counts (2, 3, 4, 5, 8, 10, 14) and total cell counts per condition (~1,000 to ~50,000)

## Outputs

- Per-parameter sensitivity and false-positive rates against simulated ground truth
- auROC curves as a function of mean total cell count per condition for each pseudo-replicate count and transformation (logit vs arcsin-sqrt)
- Overall accuracy values per pseudo-replicate count
- Benchmark figure panels (e.g. fig3_benchmark_panels.png) with boxplots via out.plot

## How to apply

Generate ~100 simulated replicate datasets per condition-size using Scanpro's sim_scanpro function, starting from original replicates (4 in the paper) and escalating total cells per condition (~1,000 to ~50,000). For each simulated dataset, run run_scanpro with the parameter sweep of interest (2, 3, 4, 5, 8, 10, and 14 pseudo-replicates) under both logit and arcsin-sqrt variance-stabilizing transformations, calling the underlying anova and t_test routines for differential proportion calls. Record sensitivity and false-positive rates against the simulated ground truth, then compute auROC as a function of mean total cell count per condition and overall accuracy per pseudo-replicate count. Render sensitivity/FPR, auROC-vs-mean-cell-count, and accuracy panels via Scanpro's plotting interface (out.plot with kind='boxplot' where appropriate) and assemble the benchmark figure.

## Related tools

- **Scanpro** (Provides sim_scanpro for simulating replicate datasets, run_scanpro/anova/t_test for bootstrapped differential proportion calls, and out.plot for benchmark panels) — https://github.com/loosolab/scanpro
- **Python** (Execution environment for the simulation and benchmarking pipeline)
- **Scanpy** (Single-cell framework Scanpro integrates with; input data handled via Scanpy/AnnData)
- **AnnData** (Native input format for Scanpro (read via ad.read_h5ad))

## Examples

```
from scanpro import scanpro, sim_scanpro; out = run_scanpro(sim_data, samples_col='sample', clusters_col='cluster', conds_col='conds'); out.plot(kind='boxplot')
```

## Evaluation signals

- Reproduced auROC curves show good auROC for 2 and 4 pseudo-replicates up to ~5,000 mean cells per condition
- 8, 10, and 14 pseudo-replicates outperform lower counts for datasets above ~25,000 cells per condition
- Sensitivity and false-positive rates are computed against the simulated ground truth and agree with reported Fig. 3 behavior
- Both variance-stabilizing transformations (logit and arcsin-sqrt) are evaluated and compared per pseudo-replicate count
- auROC and accuracy are reported per pseudo-replicate count over ~100 simulated datasets at each of 16 escalating total cell counts

## Limitations

- Bootstrapping has limited benefits when handling rare cell types
- Clusters supported by less than 2% of all cells produce unstable results in benchmarking
- Recommendation to prefer larger, less specific clusters with higher cell numbers over many small clusters when choosing clustering parameters for PA
- Simulated benchmarks reflect the paper's simulation setup (sim_scanpro, 4 original replicates); transferability to other designs is untested here

## Evidence

- [other] 2 and 4 pseudo-replicates achieved the best auROC for mean cell counts up to ~5,000 cells, whereas 8, 10 and 14 pseudo-replicates performed better with datasets having more than 25,000 cells.: [evidence span withheld — anchor status: not_found]
- [intro] Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally: "Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally"
- [discussion] From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results: "From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
- [discussion] as indicated by the benchmarking test and the real datasets, bootstrapping has limited benefits when handling rare cell types: "as indicated by the benchmarking test and the real datasets, bootstrapping has limited benefits when handling rare cell types"
- [results] adata = ad.read_h5ad('scanpro_tutorial.h5ad'): "adata = ad.read_h5ad('scanpro_tutorial.h5ad')"
