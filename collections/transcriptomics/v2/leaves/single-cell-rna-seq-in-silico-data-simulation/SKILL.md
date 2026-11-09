---
name: single-cell-rna-seq-in-silico-data-simulation
description: Use when a runtime or performance benchmark must be repeated on controlled
  synthetic scRNA-seq data with the same dimensions as an existing benchmark (e.g.,
  dataset_sizes = [1000..250000] cells × 10000 genes), especially to isolate the effect
  of UMI-count sparsity.
when_to_use_negative:
- The benchmark requires real biological data (e.g., pbmc3k, pbmc68k_reduced, or the
  4.4M-cell Macosko Mouse Atlas) rather than synthetic sparsity-controlled data
- The study targets accuracy or biological validity of downstream analyses rather
  than runtime/memory profiling, since the AnnSQL database structure mirrors AnnData
  rather than being optimized for performance
- Databases exceed 30k genes, given the known unresolved DuckDB memory leak on such
  databases
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Splatter
  role: Generates the in silico sparse UMI-count scRNA-seq datasets with shapes matching
    the random benchmark data
- name: AnnSQL
  role: Builds on-disk (MakeDb) and in-memory AnnSQL databases from the simulated
    AnnData objects and executes the timed filter/query operations
  repo: github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: Baseline format for the simulated objects; used in not-backed and backed modes
    as comparison targets for the runtime measurements
- name: DuckDB
  role: In-process relational engine underlying AnnSQL databases built from the simulated
    data
- name: Python
  role: Environment (Python 3.12) hosting AnnSQL, Splatter integration, benchmark
    timing, t-tests, and plotting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:c6dfca15c25cdfbc564b21d48094ce97989dc9f6cb8dbef4180a0247d94f2c89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - AnnSQL
  - ClusterEnsembles
  - DuckDB
  - Python
  - R
  - SCINA
  - ScType
  - Splatter
  - scSorter
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- single-cell-rna-seq-simulation
- single-cell-rna-seq-simulation-splatter
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-rna-seq-in-silico-data-simulation

## Summary

Generate in silico single-cell UMI-count datasets with the Splatter simulation framework, matching the shapes of real or randomly generated AnnData objects, to enable controlled runtime benchmarking (e.g., AnnSQL vs AnnData) on sparser synthetic data.

## When to use

Use when a runtime or performance benchmark must be repeated on controlled synthetic scRNA-seq data with the same dimensions as an existing benchmark (e.g., dataset_sizes = [1000..250000] cells × 10000 genes), especially to isolate the effect of UMI-count sparsity.

## When NOT to use

- The benchmark requires real biological data (e.g., pbmc3k, pbmc68k_reduced, or the 4.4M-cell Macosko Mouse Atlas) rather than synthetic sparsity-controlled data
- The study targets accuracy or biological validity of downstream analyses rather than runtime/memory profiling, since the AnnSQL database structure mirrors AnnData rather than being optimized for performance
- Databases exceed 30k genes, given the known unresolved DuckDB memory leak on such databases

## Inputs

- Benchmark shapes to replicate: dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000] cells with 10000 genes/columns
- Splatter simulation parameters for sparse in silico scRNA-seq UMI-count data
- Reference randomly generated AnnData benchmark datasets from the original experiment

## Outputs

- Splatter-simulated AnnData objects matching the original benchmark shapes
- On-disk AnnSQL databases (.asql files, e.g., db_name="random_data_"+str(i)) built with MakeDb (create_all_indexes=False, convenience_view=False)
- Aggregated runtime tables (CSV via df.to_csv(index=False)) and runtime comparison plots for AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, and AnnData backed
- Paired t-test results with Bonferroni-corrected alpha (0.05 / num_comparisons)

## How to apply

Set up a Python 3.12 environment with AnnSQL (pip install annsql) and Splatter, then simulate a second set of sparse UMI-count datasets whose shapes exactly match the original randomly generated benchmark data (dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000] with columns = 10000). Build on-disk AnnSQL databases from each simulated AnnData object using MakeDb with the benchmark settings (db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False) and instantiate the corresponding in-memory AnnSQL objects. Time the benchmark filter/query operations against AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, and AnnData backed for every dataset size so results are directly comparable to the random-data benchmark. Aggregate runtimes, apply paired t-tests with Bonferroni correction (alpha_corrected = 0.05 / num_comparisons), plot the aggregated runtime comparison, and write the runtime table to CSV via df.to_csv(index=False). The rationale is that Splatter-generated data contain sparser UMI counts, so any runtime deltas relative to the random-data benchmark reveal how data density affects each method.

## Related tools

- **Splatter** (Generates the in silico sparse UMI-count scRNA-seq datasets with shapes matching the random benchmark data)
- **AnnSQL** (Builds on-disk (MakeDb) and in-memory AnnSQL databases from the simulated AnnData objects and executes the timed filter/query operations) — github.com/ArpiarSaundersLab/annsql
- **AnnData** (Baseline format for the simulated objects; used in not-backed and backed modes as comparison targets for the runtime measurements)
- **DuckDB** (In-process relational engine underlying AnnSQL databases built from the simulated data)
- **Python** (Environment (Python 3.12) hosting AnnSQL, Splatter integration, benchmark timing, t-tests, and plotting)

## Examples

```
conda create -n annsql python=3.12; pip install annsql; MakeDb(adata=sim_adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)
```

## Evaluation signals

- Simulated AnnData shapes exactly match dataset_sizes = [1000 ... 250000] with columns = 10000, identical to the random-data benchmark
- Every measured runtime in the AnnSQL-vs-AnnData comparisons decreases on the Splatter-simulated sparser UMI-count data relative to the random-data benchmark (the reported finding)
- All four comparison arms (AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, AnnData backed) are timed for every dataset size
- Statistical comparisons use paired t-tests with Bonferroni correction (alpha_corrected = 0.05 / num_comparisons)
- Aggregated runtime table is written to CSV with df.to_csv(index=False) and the runtime comparison plot renders for all sizes

## Limitations

- Runtime reductions on Splatter data are attributed to sparser UMI counts, so conclusions are specific to data density and may not transfer to dense datasets
- DuckDB has a known unresolved memory leak on databases with more than 30k genes (TODO in the source code)
- No software version numbers are reported for the tools in this workflow; results are computed at runtime and only persisted to CSV, so exact runtime values are not reproducible from the article

## Evidence

- [results] Splatter simulation matches benchmark shapes: "dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000]

columns = 10000"
- [results] MakeDb benchmark settings: "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [results] Bonferroni correction for method comparisons: "# Adjusted alpha using Bonferroni correction
alpha_corrected = 0.05 / num_comparisons"
- [intro] AnnSQL converts AnnData layers into a relational DuckDB database: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database"
- [readme] AnnSQL pip installation: "## Quick Setup
``npip install annsql
"
