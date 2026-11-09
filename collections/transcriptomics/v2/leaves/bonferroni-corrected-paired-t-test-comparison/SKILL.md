---
name: bonferroni-corrected-paired-t-test-comparison
description: Use when comparing method runtimes (or similar paired measurements) across
  multiple dataset sizes — e.g., AnnSQL vs AnnData filter/query timings at 14 simulated
  cell counts — and per-size significance must be controlled for multiple testing.
when_to_use_negative:
- When only a single runtime measurement per method per dataset size exists — the
  paired t-test requires repeated paired trials at each size
- When the goal is a general performance claim rather than a per-dataset-size statistical
  comparison — a single aggregate runtime plot suffices without correction
- When the compared runtimes come from different query operations per method (not
  paired equivalents), since pairing is the basis of the test
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: AnnSQL
  role: Subject of one arm of the paired runtime comparison; its filter/query runtimes
    are the paired samples tested
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: Comparator framework whose backed and in-memory runtimes form the other arm
    of each paired t-test
- name: Pandas
  role: Writes the per-trial runtimes to CSV (df.to_csv(..., index=False)) so the
    paired tests and aggregation are reproducible
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:14fad6850867a7ed7603329afde81f15fe7b358a26e55e8a285884cf2440a9ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArpiarSaundersLab/annsql
  tools:
  - AnnData
  - AnnSQL
  - Pandas
  - Python
  - Splatter
  - matplotlib
  - memory_profiler
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# bonferroni-corrected-paired-t-test-comparison

## Summary

Statistical procedure from the AnnSQL benchmark in which paired t-tests compare runtimes of AnnSQL (in-memory and on-disk) versus AnnData (backed and in-memory) at each simulated dataset size (1,000–250,000 cells × 10,000 genes), with the significance alpha Bonferroni-corrected as 0.05 divided by the number of dataset-size comparisons.

## When to use

Use when comparing method runtimes (or similar paired measurements) across multiple dataset sizes — e.g., AnnSQL vs AnnData filter/query timings at 14 simulated cell counts — and per-size significance must be controlled for multiple testing.

## When NOT to use

- When only a single runtime measurement per method per dataset size exists — the paired t-test requires repeated paired trials at each size
- When the goal is a general performance claim rather than a per-dataset-size statistical comparison — a single aggregate runtime plot suffices without correction
- When the compared runtimes come from different query operations per method (not paired equivalents), since pairing is the basis of the test

## Inputs

- Per-trial runtime measurements for each method (AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, AnnData backed) at each dataset size
- Dataset-size comparison count (num_comparisons = number of dataset sizes, e.g., 14 sizes from 1,000 to 250,000 cells)
- Aggregated runtime comparison table (runtimes_table.csv)

## Outputs

- Per-dataset-size paired t-test p-values with Bonferroni-corrected significance calls (alpha_corrected = 0.05 / num_comparisons)
- Aggregated runtime comparison table (CSV, df.to_csv(..., index=False))
- Runtime comparison plot (runtime_comparison.png, matplotlib) annotated with significant/insignificant differences

## How to apply

Run the same benchmark filter/query (e.g., filter_by_cell_counts(min_cell_count=2500, max_cell_count=40000) or SQL like SELECT gene_1 FROM X WHERE gene_1 > 0.5) repeatedly on both frameworks at every dataset size in [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000], recording per-trial runtimes to CSV via df.to_csv(index=False). Because trials are repeated within each dataset size, treat the per-size measurements as paired samples and apply a paired t-test per size. Adjust the alpha with Bonferroni correction: alpha_corrected = 0.05 / num_comparisons, where num_comparisons equals the number of dataset sizes (14 here), so only differences with p < alpha_corrected are declared significant. Aggregate the runtime table, plot the comparison across all filters and sizes with matplotlib, and interpret significance jointly with the observed crossover pattern (e.g., AnnData faster only below ~45,000 cells in-memory, or below 5,000 cells on-disk).

## Related tools

- **AnnSQL** (Subject of one arm of the paired runtime comparison; its filter/query runtimes are the paired samples tested) — https://github.com/ArpiarSaundersLab/annsql
- **AnnData** (Comparator framework whose backed and in-memory runtimes form the other arm of each paired t-test)
- **Pandas** (Writes the per-trial runtimes to CSV (df.to_csv(..., index=False)) so the paired tests and aggregation are reproducible)

## Examples

```
from scipy import stats; alpha_corrected = 0.05 / num_comparisons; significant = stats.ttest_rel(annsql_runtimes, anndata_runtimes).pvalue < alpha_corrected
```

## Evaluation signals

- alpha_corrected equals exactly 0.05 / num_comparisons (i.e., ~0.00357 for 14 dataset sizes), not a raw alpha of 0.05
- A per-size difference is only declared significant when its paired t-test p-value is below alpha_corrected
- Significance calls align with the reported runtime pattern: AnnData outperforms AnnSQL only for libraries <45,000 cells in-memory (<5,000 cells on-disk), and both are <0.01 s below 75,000 cells
- At 250,000 cells the on-disk comparison shows AnnSQL ~0.14 s vs AnnData ~57 s (≈400× speedup), and the test at that size should reflect the large difference
- Runtime tables are written with index=False so per-trial pairs can be re-verified from the CSV

## Limitations

- No software version numbers are reported for the benchmarking tools, so exact reproducibility of runtimes and p-values is not guaranteed
- No numeric runtime/memory values appear in the code — results are only computed at runtime and written to CSV, so expected values must be validated against regenerated runs
- The article's runtime comparisons were performed on simulated data (random dense matrices and Splatter-generated sparse UMI counts); runtimes on sparser data decreased across all comparisons, so significance conclusions do not necessarily transfer to dense real datasets

## Evidence

- [results] Bonferroni-corrected alpha in the benchmark code: "# Adjusted alpha using Bonferroni correction
alpha_corrected = 0.05 / num_comparisons"
- [results] Paired t-tests applied to runtime comparisons across dataset sizes: [evidence span withheld — anchor status: not_found]
- [abstract] Runtime claims the corrected tests are meant to adjudicate: "equivalent operations in AnnData or Seurat largely failed (or were ∼700× slower) on a high-performance computing cluster"
- [results] Runtimes are persisted to CSV for later statistical comparison: "df.to_csv("atlas_profile_hpc.csv", index=False)"
- [methods] Rationale for the runtime comparisons being tested: "Running AnnSQL locally for datasets that are larger than memory, that would typically require AnnData in backed mode see substantial runtime improvements"
