---
name: single-cell-cell-count-simulation
description: Use when you need to determine how many pseudo-replicates Scanpro's bootstrapping
  should use for an unreplicated single-cell dataset, given an estimate of total cells
  per condition.
when_to_use_negative:
- The dataset already has biological replicates for all conditions — run standard
  Scanpro with samples_col instead of bootstrapping simulation
- The cell type of interest is rare (<2% of all cells), where benchmarking showed
  bootstrapping has limited benefits
- The goal is differential expression, not differential cell-type proportion analysis
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3308
tools:
- name: Scanpro
  role: Provides sim_scanpro for generating simulated replicate datasets, run_scanpro
    for bootstrapping with pseudo-replicates, anova/t_test for differential proportion
    calls, and out.plot for rendering benchmark panels
  repo: https://github.com/loosolab/scanpro
- name: Python
  role: Environment in which Scanpro is installed (pip install scanpro) and the simulation/benchmarking
    sweep is orchestrated
- name: Scanpy
  role: Python framework into which Scanpro seamlessly integrates for loading and
    handling single-cell data
- name: AnnData
  role: Native input format (h5ad) that Scanpro handles for proportion analysis on
    simulated datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:57430f9f029bebf659c75a204271c52ea77ce5eec6630f9521cb2a89a2fe06f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - AnnData
  - Python
  - Scanpro
  - Scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# single-cell-cell-count-simulation

## Summary

Benchmark a proportion-analysis bootstrapping method by simulating single-cell replicate datasets across a range of total cell counts per condition (~1,000 to ~50,000 cells) and testing how the optimal number of pseudo-replicates (2–14) shifts with dataset size, scored by auROC and accuracy.

## When to use

Use when you need to determine how many pseudo-replicates Scanpro's bootstrapping should use for an unreplicated single-cell dataset, given an estimate of total cells per condition.

## When NOT to use

- The dataset already has biological replicates for all conditions — run standard Scanpro with samples_col instead of bootstrapping simulation
- The cell type of interest is rare (<2% of all cells), where benchmarking showed bootstrapping has limited benefits
- The goal is differential expression, not differential cell-type proportion analysis

## Inputs

- AnnData object (.h5ad) with per-cell clustering and condition annotations (e.g. scanpro_tutorial.h5ad)
- Real single-cell dataset with ~4 replicates to serve as simulation seed
- List of pseudo-replicate counts to test (2, 3, 4, 5, 8, 10, 14)
- Variance-stabilizing transformation choices (logit, arcsin-sqrt)

## Outputs

- 100 simulated replicate datasets per total cell count (16 counts, ~1,000–~50,000 cells)
- Sensitivity and false-positive rates against simulated ground truth
- auROC curves as a function of mean total cell count per condition
- Overall accuracy per pseudo-replicate count and transformation
- Benchmark figure panels (e.g. fig3_benchmark_panels.png) with boxplots

## How to apply

Starting from a real dataset with 4 original replicates, use Scanpro's sim_scanpro function to generate 100 simulated replicate datasets at each of 16 escalating total cell counts per condition (~1,000 to ~50,000 cells). For each simulated dataset, run Scanpro bootstrapping via run_scanpro with 2, 3, 4, 5, 8, 10, and 14 pseudo-replicates under both logit and arcsin-sqrt variance-stabilizing transformations, calling the underlying anova and t_test routines for differential proportion calls. Record sensitivity and false-positive rates against the simulated ground truth, then compute auROC as a function of mean total cell count per condition and overall accuracy per pseudo-replicate count and transformation. Render the sensitivity/FPR, auROC-vs-mean-cell-count, and accuracy panels with Scanpro's plotting interface (out.plot, kind='boxplot' where appropriate) and assemble them into a benchmark figure; decision guidance: prefer 2 and 4 pseudo-replicates below ~5,000 mean cells and 8, 10, or 14 above ~25,000 mean cells.

## Related tools

- **Scanpro** (Provides sim_scanpro for generating simulated replicate datasets, run_scanpro for bootstrapping with pseudo-replicates, anova/t_test for differential proportion calls, and out.plot for rendering benchmark panels) — https://github.com/loosolab/scanpro
- **Python** (Environment in which Scanpro is installed (pip install scanpro) and the simulation/benchmarking sweep is orchestrated)
- **Scanpy** (Python framework into which Scanpro seamlessly integrates for loading and handling single-cell data)
- **AnnData** (Native input format (h5ad) that Scanpro handles for proportion analysis on simulated datasets)

## Examples

```
from scanpro import scanpro; out = scanpro(adata, clusters_col='clusters', conds_col='condition', samples_col=None)
```

## Evaluation signals

- Reproduced auROC curves show best auROC for 2 and 4 pseudo-replicates at mean cell counts up to ~5,000, and for 8, 10, 14 pseudo-replicates above ~25,000 cells
- Sensitivity and false-positive rates match the simulated ground truth used for scoring, not a held-out biological label
- Both logit and arcsin-sqrt transformations are evaluated and their accuracy panels agree with the reported Fig. 3 behavior
- Clusters supported by less than 2% of all cells are recognized as producing unstable results in the simulation benchmark
- Assembled figure (e.g. fig3_benchmark_panels.png) contains sensitivity/FPR, auROC-vs-mean-cell-count, and accuracy panels

## Limitations

- Bootstrapping has limited benefits when handling rare cell types (<2% of all cells produce unstable results)
- Bootstrapped results on unreplicated data should be seen as an indication of proportional shifts and interpreted with caution plus additional experimental validation
- Lack of replicates limits drawing definitive conclusions about variability and is per se not advisable; bootstrapping often, but not always, yields the same conclusion as replicated analysis

## Evidence

- [abstract] Scanpro is fast, accurate, supports datasets without replicates, and is intended for bioinformatics experts and beginners alike: [evidence span withheld — anchor status: not_found]
- [intro] Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality: "Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally"
- [methods] - `run_scanpro` - `run_stats` - `anova` - `t_test` - `sim_scanpro`: "- `run_scanpro`
- `run_stats`
- `anova`
- `t_test`
- `sim_scanpro`"
- [discussion] we address this issue by our iteration-based bootstrapping method: "we address this issue by our iteration-based bootstrapping method"
- [readme] If samples_col is not given or set to None, the dataset is assumed to be not replicated and scanpro will run the bootstrapping method.: [evidence span withheld — anchor status: external_doc]
