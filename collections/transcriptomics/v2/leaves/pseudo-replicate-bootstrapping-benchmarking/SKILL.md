---
name: pseudo-replicate-bootstrapping-benchmarking
description: Use when evaluating or running Scanpro on unreplicated single-cell proportion
  data (no samples_col), or when determining how many pseudo-replicates (5-8 vs 8+)
  to use in bootstrapped differential composition analysis.
when_to_use_negative:
- The dataset already has biological replicates for all conditions — run the replicated
  linear-regression analysis instead of bootstrapping
- Clusters of interest make up less than 2% of all cells — benchmarking shows such
  clusters produce unstable results and bootstrapping has limited benefits for rare
  cell types
- The goal is differential gene expression within cell types rather than differential
  cell-type proportion (PA) analysis
tools:
- name: Scanpro
  role: Provides the scanpro function and iteration-based bootstrapping for pseudo-replicate
    simulation on unreplicated data
  repo: https://github.com/loosolab/scanpro
- name: sim_scanpro
  role: Generates 100 null or proportion-change simulated datasets per replicate count
    (2-14) for benchmarking accuracy
  repo: https://github.com/loosolab/scanpro
- name: run_stats / anova / t_test
  role: Statistical backends for testing cell proportions in replicated and bootstrapped
    runs
  repo: https://github.com/loosolab/scanpro
- name: propeller
  role: R-based linear regression PA comparator run on the same replicated and pseudo-replicated
    matrices
- name: scCODA
  role: Compositional-analysis comparator for cross-tool validation of significance
    calls
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_007
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:c0d344e46f6c66007b6bd7ffbb96f000aedfcfc5db1ad5ee8546d3af0d1ffd7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - Python
  - Scanpro
  - Scanpy
  - propeller
  - run_stats
  - run_stats / anova / t_test
  - scCODA
  - sim_scanpro
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# pseudo-replicate-bootstrapping-benchmarking

## Summary

Benchmark Scanpro's iteration-based bootstrapping, which simulates pseudo-replicates for unreplicated single-cell datasets, against replicated runs on null-simulation and proportion-change simulated data to identify the accuracy sweet spot for pseudo-replicate counts.

## When to use

Use when evaluating or running Scanpro on unreplicated single-cell proportion data (no samples_col), or when determining how many pseudo-replicates (5-8 vs 8+) to use in bootstrapped differential composition analysis.

## When NOT to use

- The dataset already has biological replicates for all conditions — run the replicated linear-regression analysis instead of bootstrapping
- Clusters of interest make up less than 2% of all cells — benchmarking shows such clusters produce unstable results and bootstrapping has limited benefits for rare cell types
- The goal is differential gene expression within cell types rather than differential cell-type proportion (PA) analysis

## Inputs

- AnnData object (e.g. scanpro_tutorial.h5ad) with sample, cluster, and condition metadata columns
- sim_scanpro-generated simulated datasets: 100 per replicate count from 2 to 14 (null or proportion-change settings)
- Unreplicated (merged) single-cell datasets lacking biological replicates per condition

## Outputs

- Accuracy per replicate count and pseudo-replicate count (e.g. null_simulation_accuracy.csv, simulation_accuracy.csv)
- out.results with per-cluster p-values and significance calls
- Accuracy curves and per-cluster p-value boxplots via out.plot(kinde='boxplot')

## How to apply

Use sim_scanpro to generate 100 simulated datasets per replicate count (2-14): for the null setting, two conditions and five clusters with unchanged proportions drawn from a negative-binomial distribution (mu=20000, dispersion=20, ~20,000 cells per condition); for the proportion-change setting, seven clusters with three changed. Run scanpro (from scanpro.scanpro import scanpro) with samples_col='sample', clusters_col='cluster', conds_col='conds' on replicated data, and rerun with bootstrapping (samples_col=None) at increasing pseudo-replicate counts, scoring accuracy as the fraction of correctly called clusters. On unreplicated data, accuracy peaks at a sweet spot of 5-8 pseudo-replicates and decreases with more or fewer; from 8 pseudo-replicates upward, null-simulation accuracy matches the replicated result (0.98-0.99). Compare arcsin versus logit transformations (arcsin slightly outperforms logit at 5-14 pseudo-replicates) and inspect per-cluster p-value boxplots via out.plot(kinde='boxplot'), paying attention to rare clusters like Changed3 (<2% of cells).

## Related tools

- **Scanpro** (Provides the scanpro function and iteration-based bootstrapping for pseudo-replicate simulation on unreplicated data) — https://github.com/loosolab/scanpro
- **sim_scanpro** (Generates 100 null or proportion-change simulated datasets per replicate count (2-14) for benchmarking accuracy) — https://github.com/loosolab/scanpro
- **run_stats / anova / t_test** (Statistical backends for testing cell proportions in replicated and bootstrapped runs) — https://github.com/loosolab/scanpro
- **propeller** (R-based linear regression PA comparator run on the same replicated and pseudo-replicated matrices)
- **scCODA** (Compositional-analysis comparator for cross-tool validation of significance calls)

## Examples

```
from scanpro import scanpro; out = scanpro(adata, clusters_col='clusters', conds_col='condition', samples_col=None)
```

## Evaluation signals

- Replicated null-simulation accuracy falls in the reported 0.98-0.99 range, indicating good false-positive control with replicates
- Unreplicated null-simulation accuracy matches the replicated result from 8 pseudo-replicates upward
- On proportion-change data (3 of 7 clusters changed, 20,000 cells per condition), replicated accuracy spans 0.6-0.97 and the 5-8 pseudo-replicate sweet spot is visible
- arcsin slightly outperforms logit at 5-14 pseudo-replicates
- The Changed3 cluster (<2% of cells) shows the reported unstable assignment-rate behavior in per-cluster p-value boxplots

## Limitations

- Clusters supported by less than 2% of all cells in an experiment produce unstable results
- Bootstrapping has limited benefits when handling rare cell types
- Bootstrapped results should be seen as an indication of proportional shift, interpreted with caution, and require additional experimental validation
- Lack of replicates limits the ability to draw definitive conclusions about variability and is per se not advisable
- Bootstrapping accuracy on proportion-change data decreases outside the 5-8 pseudo-replicate sweet spot

## Evidence

- [intro] Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally: "Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally"
- [methods] out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds'): "out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
- [discussion] clusters supported by less than 2% of all cells in an experiment to produce unstable results: "From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
- [readme] If samples_col is not given or set to None, the dataset is assumed to be not replicated and scanpro will run the bootstrapping method.: [evidence span withheld — anchor status: external_doc]
