---
name: single-cell-count-normalization
description: Use when an AnnData h5ad (e.g. the Macosko Mouse Atlas, 4.4M cells) is
  too large for in-memory Scanpy QC, and you need cell filtering, library-size normalization,
  and log transformation via AnnSQL SQL operations.
when_to_use_negative:
- The dataset fits comfortably in memory and Scanpy/AnnData in-memory QC is fast enough
  — the AnnSQL database build adds overhead
- You need a normalization scheme other than per-cell count scaling and log transform
  (e.g. SCTransform-style regularized negative binomial), which AnnSQL's expression_normalize/expression_log
  do not implement
- The database has more than 30k genes — the paper notes an unresolved DuckDB memory
  leak on such databases
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: AnnSQL
  role: Provides the SQL/hybrid preprocessing methods calculate_total_counts, filter_by_cell_counts,
    filter_by_gene_counts, expression_normalize, and expression_log run on an AnnData-inspired
    DuckDB database
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process engine backing the AnnSQL on-disk database that executes the normalization
    SQL on tables larger than memory
- name: AnnData
  role: Source single-cell object format (.h5ad) converted into the AnnSQL database;
    also the baseline method whose equivalent normalization failed or was ~700x slower
    on HPC
- name: Seurat
  role: R baseline whose equivalent object building/normalization failed with memory
    errors in the benchmark
- name: Python
  role: Host language for AnnSQL, Scanpy loading, and memory_profiler-based runtime
    profiling
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:acc0903628fe776b5cff3b5ffeeb3c4ed4f4b86ee8275121d74f05b8669a2ac7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArpiarSaundersLab/annsql
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Python
  - Seurat
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-count-normalization

## Summary

Performs standard single-cell QC normalization — total-count calculation, cell/gene count filtering, count-per-cell normalization, and log transformation — entirely in SQL on an AnnSQL/DuckDB database so it scales to a 4.4-million-cell atlas on a laptop.

## When to use

Use when an AnnData h5ad (e.g. the Macosko Mouse Atlas, 4.4M cells) is too large for in-memory Scanpy QC, and you need cell filtering, library-size normalization, and log transformation via AnnSQL SQL operations.

## When NOT to use

- The dataset fits comfortably in memory and Scanpy/AnnData in-memory QC is fast enough — the AnnSQL database build adds overhead
- You need a normalization scheme other than per-cell count scaling and log transform (e.g. SCTransform-style regularized negative binomial), which AnnSQL's expression_normalize/expression_log do not implement
- The database has more than 30k genes — the paper notes an unresolved DuckDB memory leak on such databases

## Inputs

- AnnData object or .h5ad file (e.g. Macosko Mouse Atlas Single Nuclei, 4.4 million cells; or Scanpy sample datasets pbmc3k / pbmc68k_reduced)
- On-disk AnnSQL (.asql) DuckDB database built from the AnnData object

## Outputs

- Updated AnnSQL database with per-cell total_counts added
- Filtered and normalized (counts-per-10k, log-transformed) expression table
- Runtime/memory profiling records as a Pandas DataFrame exported to CSV (e.g. atlas_profile_hpc.csv)

## How to apply

Load the AnnData object (e.g. from .h5ad, possibly in backed mode) and build an on-disk AnnSQL database with MakeDb, saving the raw matrix layer via asql.save_raw(). First compute per-cell totals with asql.calculate_total_counts(chunk_size=750, print_progress=True), which is the chunked SQL/Python-hybrid approach recommended over a memory-intensive full SUM(COLUMNS(*)) query. Optionally apply count thresholds such as asql.filter_by_cell_counts(min_cell_count=2500, max_cell_count=40000) and asql.filter_by_gene_counts(min_gene_counts=100, max_gene_counts=10000), then normalize with asql.expression_normalize(total_counts_per_cell=10000) and compress values with asql.expression_log(log_type='LN'). Profile runtime and memory (e.g. memory_profiler's memory_usage); the article reports the full routine on the 4.4M-cell atlas takes roughly 4.28–67 minutes on a laptop, whereas equivalent AnnData/Seurat operations failed or were ~700x slower on an HPC. Export timing results to CSV (e.g. atlas_profile_hpc.csv) and accept the run if AnnSQL runtimes fall within that reported range.

## Related tools

- **AnnSQL** (Provides the SQL/hybrid preprocessing methods calculate_total_counts, filter_by_cell_counts, filter_by_gene_counts, expression_normalize, and expression_log run on an AnnData-inspired DuckDB database) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process engine backing the AnnSQL on-disk database that executes the normalization SQL on tables larger than memory)
- **AnnData** (Source single-cell object format (.h5ad) converted into the AnnSQL database; also the baseline method whose equivalent normalization failed or was ~700x slower on HPC)
- **Seurat** (R baseline whose equivalent object building/normalization failed with memory errors in the benchmark)
- **Python** (Host language for AnnSQL, Scanpy loading, and memory_profiler-based runtime profiling)

## Examples

```
from AnnSQL import AnnSQL; asql = AnnSQL(db='db/atlas.asql'); asql.calculate_total_counts(chunk_size=750, print_progress=True); asql.expression_normalize(total_counts_per_cell=10000); asql.expression_log(log_type='LN')
```

## Evaluation signals

- Recorded AnnSQL runtimes on the 4.4M-cell atlas fall within the reported ~4.28–67 minute laptop range
- Per-cell total counts are populated before filtering/normalization (calculate_total_counts must run first)
- Normalized values follow the specified scale (total_counts_per_cell=10000) followed by natural-log (log_type='LN') transformation
- AnnData (backed mode) and Seurat equivalents are recorded as failed or substantially slower (~700x on HPC) in the comparison DataFrame
- Profiling output is written to a CSV (e.g. atlas_profile_hpc.csv) with runtime and memory_usage measurements per operation

## Limitations

- Known unresolved DuckDB memory leak on databases with more than 30k genes (TODO in the authors' code)
- calculate_total_counts via raw SQL (SUM(COLUMNS(*))) is memory intensive; the chunked approach (chunk_size=750) is required for large datasets
- Equivalent AnnData analysis failed on both laptop and HPC and Seurat object building failed with memory errors, so cross-tool validation of the normalized result is not straightforward
- No software version numbers are reported for AnnSQL, Scanpy, or DuckDB in the article's code, complicating exact reproduction

## Evidence

- [readme] basic QC on the dataset via calculate_total_counts, filter_by_cell_counts, filter_by_gene_counts then normalize & log umi counts: "#basic QC on the dataset
asql.calculate_total_counts()
asql.filter_by_cell_counts(min_cell_count=1000, max_cell_count=50000)
asql.filter_by_gene_counts(min_gene_counts=100,"
- [readme] Extended methods are SQL based or Python/SQL hybrid implementations: "# Extended AnnSQL methods (See: https://docs.annsql.com/preprocessing)
# These methods are either SQL based or Python/SQL hybrid implementations."
- [results] Atlas workflow: save raw layer then chunked total-count calculation with chunk_size=750: "#save a raw matrix layer
asql.save_raw()"
- [results] Atlas profiling runtimes and CSV export of measurements: "df.to_csv("atlas_profile_hpc.csv", index=False)"
- [abstract] AnnSQL minutes on laptop vs AnnData/Seurat ~700x slower on HPC: "AnnSQL-based operations were executed in minutes on a laptop for which equivalent operations in AnnData or Seurat largely failed (or were ∼700× slower)"
