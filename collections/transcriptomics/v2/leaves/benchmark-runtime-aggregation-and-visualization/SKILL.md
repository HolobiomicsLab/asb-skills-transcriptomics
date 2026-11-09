---
name: benchmark-runtime-aggregation-and-visualization
description: Use when comparing execution runtimes of multiple analysis tools (e.g.,
  AnnSQL vs AnnData) across a series of dataset sizes and you need statistical comparison
  plus a visualized, tabulated runtime summary.
when_to_use_negative:
- Input consists of already-aggregated summary statistics rather than raw per-operation
  runtime measurements — re-timing is required for paired t-tests
- The benchmark compares a single method only, with no pairwise runtime comparisons
  to test or plot
- Datasets are so small that runtime differences are below measurement noise (e.g.,
  the pbmc3k_reduced sample dataset rather than the 1k–250k cell benchmark series)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: AnnSQL
  role: Tool under benchmark; its in-memory and on-disk modes provide the filter/query
    operations that are timed
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: Comparison tool; its not-backed and backed modes are timed against AnnSQL
- name: Splatter
  role: Generates the sparse in-silico UMI-count datasets used in the second runtime-comparison
    series
- name: Python
  role: Implements timing loops, aggregation, Bonferroni-corrected paired t-tests,
    plotting, and CSV export
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:c29185fb70ec9930ad0590eadd1883078ab52c2adeb60d6baa028225645c0f80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArpiarSaundersLab/annsql
  tools:
  - AnnData
  - AnnSQL
  - Python
  - Splatter
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# benchmark-runtime-aggregation-and-visualization

## Summary

Aggregates benchmark runtimes across methods (AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, AnnData backed) and dataset sizes, applies paired t-tests with Bonferroni correction, and produces aggregated runtime comparison plots and CSV tables.

## When to use

Use when comparing execution runtimes of multiple analysis tools (e.g., AnnSQL vs AnnData) across a series of dataset sizes and you need statistical comparison plus a visualized, tabulated runtime summary.

## When NOT to use

- Input consists of already-aggregated summary statistics rather than raw per-operation runtime measurements — re-timing is required for paired t-tests
- The benchmark compares a single method only, with no pairwise runtime comparisons to test or plot
- Datasets are so small that runtime differences are below measurement noise (e.g., the pbmc3k_reduced sample dataset rather than the 1k–250k cell benchmark series)

## Inputs

- Runtime measurements for AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, and AnnData backed across filter/query types and dataset sizes (1,000 to 250,000 cells, 10,000 genes)
- Optional second benchmark series (e.g., Splatter-simulated sparse UMI-count datasets matching the random-data shapes)

## Outputs

- Aggregated runtime comparison plot (all filters, one color per method)
- CSV runtime table written with df.to_csv(index=False)
- Paired t-test results with Bonferroni-corrected alpha (alpha_corrected = 0.05 / num_comparisons)

## How to apply

Time each benchmark filter/query operation against every method (AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, AnnData backed) for each dataset size in a size series such as dataset_sizes = [1000, 5000, ..., 250000] with columns = 10000. Aggregate runtimes per method and size, then test pairwise method comparisons with paired t-tests, correcting alpha with a Bonferroni correction (alpha_corrected = 0.05 / num_comparisons, where num_comparisons equals the number of dataset sizes). Plot the aggregated runtime comparison for all filter/query types, assigning distinct colors per method. Write the aggregated runtime table to CSV with df.to_csv(..., index=False) so numeric results are persisted even though they are only computed at runtime. Validate by checking the expected ordering/invariant (e.g., sparser Splatter-simulated UMI data yields lower runtimes across all comparisons than the random-data benchmark).

## Related tools

- **AnnSQL** (Tool under benchmark; its in-memory and on-disk modes provide the filter/query operations that are timed) — https://github.com/ArpiarSaundersLab/annsql
- **AnnData** (Comparison tool; its not-backed and backed modes are timed against AnnSQL)
- **Splatter** (Generates the sparse in-silico UMI-count datasets used in the second runtime-comparison series)
- **Python** (Implements timing loops, aggregation, Bonferroni-corrected paired t-tests, plotting, and CSV export)

## Examples

```
import pandas as pd; from scipy.stats import ttest_rel; alpha_corrected = 0.05 / num_comparisons; ttest_rel(annsql_runtimes, anndata_runtimes); df.to_csv('runtime_comparison.csv', index=False)
```

## Evaluation signals

- Runtimes of all AnnSQL-vs-AnnData comparisons decrease on sparser Splatter-simulated UMI data relative to the random-data benchmark
- Statistical tests use alpha_corrected = 0.05 / num_comparisons, with num_comparisons equal to the number of dataset sizes
- Runtime table CSV contains rows for every dataset size (14 sizes from 1000 to 250000, columns = 10000) and every filter/query type
- Aggregated plot uses one distinct color per method (e.g., colors = ['#07b88e', '#a4a6a4', '#07b88e', '#a4a6a4', '#FFA067'])
- On-disk AnnSQL databases were built with benchmark settings create_all_indexes=False and convenience_view=False so timings reflect the documented configuration

## Limitations

- Numeric runtime and memory values are computed at runtime and only persisted via CSV export; no result values appear in the source code
- Known unresolved issue: DuckDB memory leak on databases with more than 30k genes
- No software version numbers are reported, so benchmark results may not be directly reproducible across tool versions

## Evidence

- [results] Aggregated runtimes and Bonferroni-corrected paired t-tests: "Adjusted alpha using Bonferroni correction
alpha_corrected = 0.05 / num_comparisons"
- [results] Benchmark dataset size series and gene count: "dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000]

columns = 10000"
- [results] Aggregated plot colors per method: "colors = ["#07b88e", "#a4a6a4", "#07b88e", "#a4a6a4","#FFA067"]

#plot aggregation of all filters runtime"
- [results] On-disk database build settings used in the benchmark: "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [results] Results are only persisted at runtime via CSV: "df.to_csv("atlas_profile_hpc.csv", index=False)"
