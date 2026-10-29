---
name: large-scale-single-cell-runtime-benchmarking
description: Use when profiling runtime and memory of AnnSQL preprocessing procedures
  on a large single-cell atlas (e.g. 4.4 million cell h5ad) and comparing against
  AnnData/Seurat equivalents that may fail on the same hardware.
when_to_use_negative:
- Dataset fits comfortably in memory and is small (e.g. Scanpy's pbmc3k or pbmc68k_reduced);
  in-memory AnnSQL or plain AnnData is simpler and the laptop-vs-HPC benchmark framing
  does not apply.
- The goal is statistical accuracy or biological result comparison rather than runtime/memory
  profiling; this skill records timing outcomes, not expression values.
- The database has more than ~30k genes, where the known unresolved DuckDB memory
  leak affects AnnSQL runs.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AnnSQL
  role: Builds the AnnData-inspired on-disk DuckDB database and executes the preprocessing
    procedures (calculate_total_counts, normalization, log transformation) being timed
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process engine underlying the AnnSQL database that enables the fast chunked
    SQL procedures being benchmarked
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: Backed-mode baseline whose equivalent operations are recorded as failed or
    substantially slower for comparison
- name: Seurat
  role: R baseline whose object building fails with memory errors on the atlas dataset,
    serving as the failure baseline
- name: Pandas
  role: Collects runtime measurements into a DataFrame exported as atlas_profile_hpc.csv
- name: memory_profiler
  role: Provides memory_usage profiling used to time each AnnSQL operation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:a1e1134fc846366d74e1a93d295b37f13675478638cd97530f19bf7d0c31eae0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArpiarSaundersLab/annsql
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Pandas
  - Python
  - Seurat
  - memory_profiler
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# large-scale-single-cell-runtime-benchmarking

## Summary

A benchmarking skill for measuring how long routine single-cell preprocessing procedures (cell filtering, total-count calculation, normalization, log transformation) take on million-cell-scale AnnData atlases using AnnSQL/DuckDB, compared against AnnData backed mode and Seurat baselines.

## When to use

Use when profiling runtime and memory of AnnSQL preprocessing procedures on a large single-cell atlas (e.g. 4.4 million cell h5ad) and comparing against AnnData/Seurat equivalents that may fail on the same hardware.

## When NOT to use

- Dataset fits comfortably in memory and is small (e.g. Scanpy's pbmc3k or pbmc68k_reduced); in-memory AnnSQL or plain AnnData is simpler and the laptop-vs-HPC benchmark framing does not apply.
- The goal is statistical accuracy or biological result comparison rather than runtime/memory profiling; this skill records timing outcomes, not expression values.
- The database has more than ~30k genes, where the known unresolved DuckDB memory leak affects AnnSQL runs.

## Inputs

- Macosko Mouse Atlas Single Nuclei AnnData .h5ad file (4.4 million cells), loaded in backed mode
- On-disk AnnSQL (.asql) DuckDB database built from the AnnData object with the raw layer saved
- Benchmark configuration (chunk_size=750 for chunked count calculations)

## Outputs

- Per-operation runtime and memory measurements (e.g. via memory_profiler memory_usage)
- atlas_profile_hpc.csv: Pandas DataFrame of runtime measurements and AnnData/Seurat comparison outcomes
- Comparison verdicts recording AnnData/Seurat failures or ~700x slowdowns on HPC

## How to apply

Load the Macosko Mouse Atlas single-nuclei AnnData object (4.4 million cells) from its .h5ad file in backed mode with scanpy's read_h5ad. Build an on-disk AnnSQL database via MakeDb and save the raw matrix layer with asql.save_raw(), then execute the Table 1 extended-functionality procedures: asql.calculate_total_counts(chunk_size=750, print_progress=True), cell-count normalization, and cell-count log transformation. Profile each operation with memory_profiler's memory_usage to capture runtime and memory, and record outcomes for the equivalent AnnData (backed mode) and Seurat attempts as failed or slower baselines. Collect the measurements into a Pandas DataFrame and export to atlas_profile_hpc.csv. Accept the benchmark when AnnSQL runtimes fall within the reported ~4.28-67 minute laptop range and the AnnData/Seurat equivalents are recorded as failed or ~700x slower on the HPC.

## Related tools

- **AnnSQL** (Builds the AnnData-inspired on-disk DuckDB database and executes the preprocessing procedures (calculate_total_counts, normalization, log transformation) being timed) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process engine underlying the AnnSQL database that enables the fast chunked SQL procedures being benchmarked) — https://github.com/ArpiarSaundersLab/annsql
- **AnnData** (Backed-mode baseline whose equivalent operations are recorded as failed or substantially slower for comparison)
- **Seurat** (R baseline whose object building fails with memory errors on the atlas dataset, serving as the failure baseline)
- **Pandas** (Collects runtime measurements into a DataFrame exported as atlas_profile_hpc.csv)
- **memory_profiler** (Provides memory_usage profiling used to time each AnnSQL operation)

## Examples

```
asql.calculate_total_counts(chunk_size=750, print_progress=True); df.to_csv("atlas_profile_hpc.csv", index=False)
```

## Evaluation signals

- Recorded AnnSQL runtimes fall within the reported ~4.28-67 minute laptop range for the 4.4 million cell atlas
- AnnData and Seurat equivalents are recorded as failed or substantially slower (~700x on the HPC) matching the paper's baseline
- atlas_profile_hpc.csv is produced with per-operation runtime rows and no missing timing entries
- Chunked procedures run with chunk_size=750 and print_progress=True complete without DuckDB memory errors
- The on-disk .asql database contains the raw matrix layer saved via asql.save_raw() before preprocessing begins

## Limitations

- AnnData analysis failed on both machines and Seurat object building failed due to memory errors, so the comparison baselines are failure records rather than completed runtimes
- Known unresolved issue: DuckDB memory leak on databases with more than 30k genes (TODO in the profiling code)
- No software version numbers are reported for the tools in the profiling section, hindering reproducibility
- No numeric memory/runtime values appear in the article code; results are only computed at runtime and written to CSV
- Laptop runtimes were up to 3x faster than the HPC in the reported range, so absolute times vary by hardware

## Evidence

- [abstract] AnnSQL operations ran in minutes on a laptop while baselines failed: "AnnSQL-based operations were executed in minutes on a laptop for which equivalent operations in AnnData or Seurat largely failed (or were ∼700× slower)"
- [results] Profiling code imports memory_profiler and writes results to CSV: "import scanpy as sc
from memory_profiler import memory_usage"
- [results] Results written to atlas_profile_hpc.csv: "df.to_csv("atlas_profile_hpc.csv", index=False)"
- [results] Atlas dataset is 4.4 million cells loaded in backed mode: "#load the atlas dataset (4.4 million cells) in backed mode"
- [readme] README documents chunked count calculation method: "asql.calculate_total_counts()
asql.filter_by_cell_counts(min_cell_count=1000, max_cell_count=50000)"
- [methods] Substantial runtime improvements for datasets larger than memory: "Running AnnSQL locally for datasets that are larger than memory, that would typically require AnnData in backed mode see substantial runtime improvements"
