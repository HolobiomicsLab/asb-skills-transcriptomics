---
name: simulated-single-cell-expression-data-generation
description: Use when you need controlled-size dense scRNA-seq test data (e.g., 1,000
  to 250,000 cells, 10,000 genes) to benchmark query runtimes of AnnSQL, AnnData (backed
  or in-memory), or Seurat without depending on real atlas files.
when_to_use_negative:
- Databases with more than 30,000 genes — the article notes an unresolved DuckDB memory
  leak on databases with >30k genes
- Analyses requiring realistic biological structure or annotations — simulated matrices
  are random-expression values, not real scRNA-seq
- Single small datasets (e.g., pbmc3k/pbmc68k_reduced) where framework runtime differences
  are not the question being tested
tools:
- name: AnnSQL
  role: Package under benchmark; its MakeDb utility builds on-disk .asql databases
    from each simulated AnnData object
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: Container format for the simulated dense count matrices; also the comparison
    framework in backed and in-memory modes
- name: Scanpy
  role: Used in the environment for AnnData I/O and sample datasets (e.g., sc.datasets.pbmc68k_reduced());
    imported in the benchmark scripts
- name: Pandas
  role: Writes per-size runtime results to CSV (df.to_csv(..., index=False)) and aggregates
    the runtime comparison table
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:d98fe36cdb6526f2ce7b7c4478d34ba45a3bc6045a502bcb1c7766b136cc8b27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArpiarSaundersLab/annsql
  tools:
  - AnnData
  - AnnSQL
  - Pandas
  - Python
  - Scanpy
  - matplotlib
  - memory_profiler
  - scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# simulated-single-cell-expression-data-generation

## Summary

Generate synthetic dense single-cell count matrices as AnnData objects at controlled sizes (1,000–250,000 cells × 10,000 genes with random expression values) to benchmark query runtime and memory of AnnSQL versus AnnData.

## When to use

Use when you need controlled-size dense scRNA-seq test data (e.g., 1,000 to 250,000 cells, 10,000 genes) to benchmark query runtimes of AnnSQL, AnnData (backed or in-memory), or Seurat without depending on real atlas files.

## When NOT to use

- Databases with more than 30,000 genes — the article notes an unresolved DuckDB memory leak on databases with >30k genes
- Analyses requiring realistic biological structure or annotations — simulated matrices are random-expression values, not real scRNA-seq
- Single small datasets (e.g., pbmc3k/pbmc68k_reduced) where framework runtime differences are not the question being tested

## Inputs

- Dataset size list (dataset_sizes = [1000, ..., 250000] cells) and gene count (columns = 10000)
- Python 3.12 environment with annsql, scanpy, pandas, matplotlib, memory_profiler

## Outputs

- Simulated dense AnnData objects (one per dataset size, 10,000 random-expression genes)
- On-disk AnnSQL .asql databases (e.g., db/random_data_<i>.asql) built from each simulated dataset
- Per-size runtime CSV tables and aggregated runtime comparison plots (e.g., runtime_comparison.png)

## How to apply

Create dense AnnData objects with random expression values for each size in dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000], holding genes fixed at columns = 10000. Instantiate each as an in-memory AnnData object and optionally as an on-disk AnnSQL database via MakeDb (e.g., db_name='random_data_'+str(i), create_all_indexes=False, convenience_view=False) so the same simulated data feed both frameworks. Rationale: synthetic dense matrices isolate runtime scaling from dataset-specific sparsity or annotation effects, allowing per-size timing of filter/query operations (e.g., filter_by_cell_counts(2500, 40000), SELECT gene_1 FROM X WHERE gene_1 > 0.5) whose results are aggregated into runtime tables and plots. Validate by checking runtimes are written to CSV per size and that cross-over patterns reproduce (AnnData faster below 45,000 cells in-memory; AnnSQL ~400x faster at 250,000 cells on-disk).

## Related tools

- **AnnSQL** (Package under benchmark; its MakeDb utility builds on-disk .asql databases from each simulated AnnData object) — https://github.com/ArpiarSaundersLab/annsql
- **AnnData** (Container format for the simulated dense count matrices; also the comparison framework in backed and in-memory modes)
- **Scanpy** (Used in the environment for AnnData I/O and sample datasets (e.g., sc.datasets.pbmc68k_reduced()); imported in the benchmark scripts)
- **Pandas** (Writes per-size runtime results to CSV (df.to_csv(..., index=False)) and aggregates the runtime comparison table)

## Examples

```
import anndata as ad, numpy as np; adata = ad.AnnData(np.random.rand(50000, 10000)); MakeDb(adata, db_path='../db/random/', db_name='random_data_45000', create_all_indexes=False, convenience_view=False)
```

## Evaluation signals

- Each simulated AnnData object has shape (n_cells, 10000) with dense random expression values, for every n_cells in dataset_sizes
- Per-size runtimes are recorded and written to CSV files (df.to_csv(..., index=False)) with no missing sizes
- Reproduced comparison matches the reported pattern: in-memory AnnData faster only below 45,000 cells; both <0.01 s below 75,000 cells in-memory
- On-disk benchmark reproduces the crossover below 5,000 cells and AnnSQL ~0.14 s vs AnnData ~57 s at 250,000 cells
- Statistical checks use paired t-tests with Bonferroni correction (alpha_corrected = 0.05 / num_comparisons)

## Limitations

- Known unresolved issue: DuckDB memory leak on databases with more than 30k genes (TODO in the benchmark code)
- No software version numbers are reported for the tools in the results section, complicating exact reproduction
- Simulated matrices use random expression values, so results reflect dense synthetic data rather than real scRNA-seq sparsity

## Evidence

- [results] Benchmark dataset sizes ranged from 1,000 to 250,000 cells with 10,000 genes per dataset: "dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000]

columns = 10000"
- [results] On-disk AnnSQL databases are built from each simulated AnnData with specific MakeDb flags: "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [results] Known unresolved DuckDB memory leak above 30k genes: "#TODO debug duckdb memory leak on daabases with >30k genes."
- [results] Statistical comparisons across dataset sizes use paired t-tests with Bonferroni correction: "# Adjusted alpha using Bonferroni correction
alpha_corrected = 0.05 / num_comparisons"
- [results] Runtime results are written to CSV at runtime rather than reported inline: "df.to_csv("atlas_profile_hpc.csv", index=False)"
