---
name: rubins-rule-multiple-imputation-pooling
description: Use when a single-cell dataset lacks biological replicates and Scanpro's
  bootstrapping produces ~100 iterations of per-cluster regression statistics that
  must be pooled into a single robust result per cell type.
when_to_use_negative:
- The dataset already has biological replicates for all conditions — run the standard
  replicated Scanpro analysis instead, since bootstrapping is designed for non-replicated
  data (omit samples_col or set it to None to trigger bootstrapping only when unreplicated)
- Rare cell types below ~2% of all cells dominate the analysis — bootstrapping has
  limited benefits for rare cell types and small clusters produce unstable results,
  so filter them out with n_min trimming rather than pooling their statistics
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3170
tools:
- name: Scanpro
  role: Implements the bootstrapping pseudo-replication workflow, the propeller-style
    linear regression (anova/t_test helpers), and the Rubin's-rule pooling of coefficients
    with median adjusted p-values
  repo: https://github.com/loosolab/scanpro
- name: Scanpy
  role: Python framework Scanpro integrates into; used to load and prepare the AnnData
    object before proportion analysis
- name: AnnData
  role: Native input format Scanpro handles (h5ad objects with sample/cluster/condition
    columns)
- name: propeller
  role: Source of the linear regression approach on proportions that Scanpro implements
    and extends with bootstrapping
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:16a4a81a911f75e72f6caedcead79eb015448b4d5c46adba712cb38486006dce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - AnnData
  - Python
  - Scanpro
  - Scanpy
  - propeller
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# rubins-rule-multiple-imputation-pooling

## Summary

Pool per-iteration beta coefficients and p-values from Scanpro's bootstrapped pseudo-replication analysis of unreplicated single-cell datasets using Rubin's multiple-imputation rule for coefficients and the median of adjusted p-values per cluster.

## When to use

Use when a single-cell dataset lacks biological replicates and Scanpro's bootstrapping produces ~100 iterations of per-cluster regression statistics that must be pooled into a single robust result per cell type.

## When NOT to use

- The dataset already has biological replicates for all conditions — run the standard replicated Scanpro analysis instead, since bootstrapping is designed for non-replicated data (omit samples_col or set it to None to trigger bootstrapping only when unreplicated)
- Rare cell types below ~2% of all cells dominate the analysis — bootstrapping has limited benefits for rare cell types and small clusters produce unstable results, so filter them out with n_min trimming rather than pooling their statistics

## Inputs

- Single-cell dataset in AnnData (.h5ad) format with sample, cluster, and condition metadata columns (e.g. scanpro_tutorial.h5ad with samples_col='sample', clusters_col='cluster', conds_col='conds')
- Per-iteration per-cluster beta coefficients and adjusted p-values from bootstrapped pseudo-replicate regressions (e.g. 100 iterations)
- Cell-to-pseudo-replicate assignment table (pseudo_replicate_cells.csv)

## Outputs

- Pooled statistics table (pooled_statistics.csv) with one row per retained cluster, a finite pooled beta coefficient, and a median adjusted p-value column
- Pseudo-replicate cell table (pseudo_replicate_cells.csv) mapping every input cell to a pseudo-replicate

## How to apply

First run Scanpro's bootstrapping mode: for each of 100 iterations, split every sample group into n_reps pseudo-replicates by sampling cells without replacement, recording cell-to-pseudo-replicate assignments in pseudo_replicate_cells.csv. In each iteration, compute per-cluster cell proportions and fit the propeller-style linear regression of proportions on condition (reusing the exported anova/t_test helpers from scanpro.scanpro) to obtain per-cluster beta coefficients and p-values. Pool the beta coefficients across the 100 iterations with Rubin's multiple-imputation rule and take the median of the adjusted p-values per cluster, writing pooled_statistics.csv. Rubin's pooling is appropriate here because the bootstrapped pseudo-replicates are analogous to multiple-imputed datasets; the median p-value is used because p-values cannot be averaged reliably. Before pooling, apply n_min trimming to drop clusters supported by fewer than n_min cells, since the paper's benchmarking found clusters under ~2% of all cells produce unstable results.

## Related tools

- **Scanpro** (Implements the bootstrapping pseudo-replication workflow, the propeller-style linear regression (anova/t_test helpers), and the Rubin's-rule pooling of coefficients with median adjusted p-values) — https://github.com/loosolab/scanpro
- **Scanpy** (Python framework Scanpro integrates into; used to load and prepare the AnnData object before proportion analysis)
- **AnnData** (Native input format Scanpro handles (h5ad objects with sample/cluster/condition columns))
- **propeller** (Source of the linear regression approach on proportions that Scanpro implements and extends with bootstrapping)

## Examples

```
from scanpro.scanpro import scanpro; out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')
```

## Evaluation signals

- pooled_statistics.csv contains exactly one row per retained cluster (after n_min trimming), each with a finite pooled beta coefficient
- The pooled p-value column equals the median of the per-iteration adjusted p-values for each cluster, not their mean
- The beta coefficients were pooled with Rubin's multiple-imputation rule across all 100 iterations, not averaged per iteration naively
- pseudo_replicate_cells.csv accounts for all input cells per sample (every cell is assigned to exactly one pseudo-replicate per iteration)
- Bootstrapped conclusions match the conclusions from replicated benchmarks when comparable data is available

## Limitations

- Bootstrapping has limited benefits when handling rare cell types, as indicated by the benchmarking tests and real datasets
- Results from bootstrapping on unreplicated data should be seen as an indication of cell types with a proportional shift and should be interpreted with caution plus additional experimental validation
- Lack of replicates limits the ability to draw definitive conclusions about variability and is per se not advisable
- Clusters supported by less than 2% of all cells produce unstable results and should be filtered out before analysis

## Evidence

- [intro] Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally: "Scanpro implements the linear regression approach proposed by propeller, and extends this by a bootstrapping functionality to internally"
- [methods] from scanpro.scanpro import scanpro: "from scanpro.scanpro import scanpro

    out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
- [methods] Exported symbols anova, t_test: "### Exported symbols
- `__version__`
- `scanpro`
- `run_scanpro`
- `run_stats`
- `anova`
- `t_test`
- `sim_scanpro`"
- [readme] If samples_col is not given, bootstrapping is used: [evidence span withheld — anchor status: external_doc]
- [discussion] Bootstrapping often results in the same conclusion as replicated datasets: "Although this approach allows for proportion analysis on non replicated datasets and often results in the same conclusion as replicated datasets (see benchmarks)"
- [discussion] Clusters supported by less than 2% produce unstable results: "From the benchmarking, we found clusters supported by less than 2% of all cells in an experiment to produce unstable results"
