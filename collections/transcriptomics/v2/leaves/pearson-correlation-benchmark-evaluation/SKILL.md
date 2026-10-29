---
name: pearson-correlation-benchmark-evaluation
description: Use when validating a trajectory-inference method against ground-truth
  time points (e.g., annotated sampling days in a time-series CyTOF or scRNA-seq dataset)
  and you need a single quantitative score per method to compare against ablations
  and baselines.
when_to_use_negative:
- The dataset has no ground-truth temporal labels (e.g., a static atlas or perturbation
  screen) — Pearson correlation against days is undefined without annotated sampling
  times.
- You need topology/lineage-quality metrics (branch detection sensitivity) rather
  than pseudotime accuracy — the article evaluates topology and lineage prediction
  separately and defers those to supplementary stress tests.
- Comparing methods on purely simulated data without real annotated time points (e.g.,
  dyntoy toy trajectories), where synthetic milestone networks call for different
  scoring schemes.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: VIA (pyVIA)
  role: Primary method under evaluation; produces single_cell_pt_markov pseudotime
    via lazy-teleporting random walks with MCMC, ablated by setting memory=1
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering backend inside VIA that builds the cluster-graph (edgepruning_clustering_resolution=0.15)
    used for pseudotime computation
  repo: https://github.com/ShobiStassen/VIA
- name: SlingShot
  role: Baseline trajectory-inference method whose pseudotime ordering is correlated
    against the annotated days
- name: Palantir
  role: Baseline trajectory-inference method compared under the same KNN settings
- name: STREAM
  role: Baseline trajectory-inference method compared under the same KNN settings
- name: pandas
  role: Computes the Pearson correlations between inferred pseudotime and day labels
    and assembles the cytof_pseudotime_correlations.csv table
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:666df7942a81deede861cce0d2958b8085e55e63dfbd40962fd25d8d15d8789a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - Palantir
  - STREAM
  - SlingShot
  - VIA
  - VIA (pyVIA)
  - pandas
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# pearson-correlation-benchmark-evaluation

## Summary

Benchmark protocol that scores trajectory-inference pseudotime accuracy by computing, in pandas, the Pearson correlation between each method's inferred pseudotime and annotated time labels — here, VIA vs. its lazy-teleporting MCMC ablation (memory=1) and baselines (Slingshot, Palantir, STREAM, PAGA) on the mESC CyTOF dataset.

## When to use

Use when validating a trajectory-inference method against ground-truth time points (e.g., annotated sampling days in a time-series CyTOF or scRNA-seq dataset) and you need a single quantitative score per method to compare against ablations and baselines.

## When NOT to use

- The dataset has no ground-truth temporal labels (e.g., a static atlas or perturbation screen) — Pearson correlation against days is undefined without annotated sampling times.
- You need topology/lineage-quality metrics (branch detection sensitivity) rather than pseudotime accuracy — the article evaluates topology and lineage prediction separately and defers those to supplementary stress tests.
- Comparing methods on purely simulated data without real annotated time points (e.g., dyntoy toy trajectories), where synthetic milestone networks call for different scoring schemes.

## Inputs

- mESC CyTOF time-series dataset (~90K cells, 28 antibody markers) with annotated sampling-day labels (pyVIA Datasets: mESC_timeseries)
- Per-cell pseudotime vectors from VIA (full lazy-teleporting MCMC mode) and the ablated mode (memory=1)
- Per-cell pseudotime orderings from baseline TI methods: Slingshot, Palantir, STREAM, PAGA
- KNN sweep settings for the HNSWlib KNN graph (values around the default knn=30)

## Outputs

- Pearson correlation of each method's pseudotime vs. annotated days, per KNN setting
- Consolidated comparison table saved as cytof_pseudotime_correlations.csv

## How to apply

Load the time-series dataset with annotated sampling-day labels (e.g., the ~90K-cell mESC CyTOF data with 28 antibody markers from the pyVIA Datasets resources). Run the method under evaluation — VIA via via.via_wrapper with a KNN sweep around knn=30, PARC clustering with edgepruning_clustering_resolution=0.15 and pseudotime_threshold_TS=30 — to obtain single_cell_pt_markov pseudotime; then re-run in the ablated non-memory mode (memory=1, Via 1.0 behavior) and run baselines (Slingshot, Palantir, STREAM, PAGA) under the same graph settings. For each method and each KNN setting, compute the Pearson correlation of inferred pseudotime versus the annotated day labels with pandas and assemble all correlations into one table (cytof_pseudotime_correlations.csv). The rationale is that a single rank-correlation-like score over many cells captures whether the pseudotime ordering recovers real developmental time, and sweeping KNN tests robustness to the graph parameter rather than a single favorable setting.

## Related tools

- **VIA (pyVIA)** (Primary method under evaluation; produces single_cell_pt_markov pseudotime via lazy-teleporting random walks with MCMC, ablated by setting memory=1) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering backend inside VIA that builds the cluster-graph (edgepruning_clustering_resolution=0.15) used for pseudotime computation) — https://github.com/ShobiStassen/VIA
- **SlingShot** (Baseline trajectory-inference method whose pseudotime ordering is correlated against the annotated days)
- **Palantir** (Baseline trajectory-inference method compared under the same KNN settings)
- **STREAM** (Baseline trajectory-inference method compared under the same KNN settings)
- **pandas** (Computes the Pearson correlations between inferred pseudotime and day labels and assembles the cytof_pseudotime_correlations.csv table)

## Examples

```
v = via.via_wrapper(adata, true_label, embedding, knn=30, jac_std_global=0.15, root=[42], pseudotime_threshold_TS=30, memory=1); corr = pd.Series(v.single_cell_pt_markov).corr(pd.Series(day_labels))
```

## Evaluation signals

- VIA's full-mode pseudotime attains a Pearson correlation of approximately 0.88 with the annotated days on the mESC CyTOF data.
- Ablating the lazy-teleporting MCMC stage (memory=1) drops the correlation to near PAGA's level, confirming the component's contribution.
- VIA's correlation exceeds that of the no-LT ablation and of the Slingshot/Palantir/STREAM/PAGA baselines across the KNN sweep, matching the paper's CyTOF benchmark.
- The correlation ranking is stable across multiple KNN settings rather than at a single favorable knn value.
- All correlations are consolidated into cytof_pseudotime_correlations.csv with one row per method per KNN setting.

## Limitations

- Only Pearson correlation against known day labels is scored; the paper does not report quantitative accuracy metrics in the discussion for topology and lineage prediction, deferring those to supplementary figures.
- No software version numbers are stated for VIA or the compared methods, so exact reproducibility of correlation values may vary across versions.
- Specific parameter ranges used in the stress tests are not reported in the discussion, so benchmark robustness beyond the swept KNN values is not documented.

## Evidence

- [discussion] CyTOF ablation benchmark (~88% correlation, drop to PAGA level): "We first applied VIA to the mESC CyTOF dataset and showed that the lazy-teleporting MCMCs strategy in VIA enables it to outperform other methods"
- [intro] VIA algorithm definition: "we present VIA, a graph-based TI algorithm that uses a new strategy to compute pseudotime, and reconstruct cell lineages based on lazy-teleporting random walks"
- [intro] PARC clustering role: "Single-cell level graph is clustered such that each node represents a cluster of single cells (computed by our clustering algorithm PARC14)"
- [methods] via_wrapper invocation with knn parameter: "via.via_wrapper(adata, true_label, embedding, knn=10, ncomps=20, jac_std_global=0.15, root=[42], dataset='',"
- [methods] pseudotime_threshold_TS default: "(optional, default = 30) Percentile threshold for potential node to qualify as Terminal State"
- [readme] Datasets location for mESC and other time-series data: "Dataset are available in the [Datasets folder](https://github.com/ShobiStassen/VIA/tree/master/Datasets) (smaller files)"
- [readme] Memory / higher-order walks distinction in Via 2.0: "StaVia extends the lazy-teleporting walks to higher order random walks with **memory** to allow better lineage detection, pathway recovery"
