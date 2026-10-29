---
name: large-scale-scrna-database-benchmarking
description: Use when reproducing or extending runtime comparisons between AnnSQL
  (in-process DuckDB) and AnnData backed-mode filtering on datasets larger than memory,
  such as the 4.4 million cell Macosko Mouse Atlas Single Nuclei h5ad.
when_to_use_negative:
- The dataset fits in memory, where AnnData not-backed mode is adequate and the backed-mode
  comparison is unnecessary
- The goal is accuracy or result correctness rather than runtime benchmarking of expression
  filtering
- Databases contain more than 30k genes, where a known DuckDB memory leak issue is
  unresolved
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AnnSQL
  role: Runs the SQL expression filter on the on-disk AnnData-inspired DuckDB database
    via asql.query() and provides MakeDb to build the .asql database
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process database engine backing AnnSQL on-disk queries that delivers the
    runtime advantage being benchmarked
- name: AnnData
  role: 'Baseline: same atlas dataset loaded in backed mode with scanpy for boolean-mask
    subsetting timing comparison'
- name: Pandas
  role: Holds aggregated timing results and writes the summary to gene_filter_runtime_comparison.csv
- name: Python
  role: Implementation language for the timing benchmark scripts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:b9564a1b908f65bef7737e465cfd5c90bf4958dc2734f7fb86c3f841bc7bd058
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# large-scale-scrna-database-benchmarking

## Summary

Benchmark SQL-based filtering of atlas-scale single-cell data: query an AnnSQL on-disk DuckDB database for an expression filter (e.g., ENSMUSG00000070880 > 0 on the 4.4 million cell Macosko Mouse Atlas) and compare runtime against AnnData backed-mode subsetting on laptop-class hardware, targeting the paper's ~746x speedup.

## When to use

Use when reproducing or extending runtime comparisons between AnnSQL (in-process DuckDB) and AnnData backed-mode filtering on datasets larger than memory, such as the 4.4 million cell Macosko Mouse Atlas Single Nuclei h5ad.

## When NOT to use

- The dataset fits in memory, where AnnData not-backed mode is adequate and the backed-mode comparison is unnecessary
- The goal is accuracy or result correctness rather than runtime benchmarking of expression filtering
- Databases contain more than 30k genes, where a known DuckDB memory leak issue is unresolved

## Inputs

- Atlas-scale .h5ad file (Macosko Mouse Atlas Single Nuclei, 4.4 million cells)
- Pre-built on-disk AnnSQL database (.asql) generated via MakeDb from the same AnnData object
- Laptop-class compute environment (and optionally an HPC environment for the in-memory reference)

## Outputs

- Timing summary table with AnnSQL and AnnData backed-mode runtimes
- Speedup ratio (target ~746x laptop; ~42x vs HPC in-memory AnnData)
- gene_filter_runtime_comparison.csv

## How to apply

Open the atlas-scale on-disk database with AnnSQL(db=...), which uses the in-process DuckDB engine, and time the SQL expression filter 'SELECT ... WHERE ENSMUSG00000070880 > 0' via asql.query(), repeating the query for a stable runtime measurement. Then load the same Macosko Mouse Atlas Single Nuclei h5ad (4.4 million cells) via scanpy in backed mode and time the equivalent boolean-mask subsetting adata_mem[adata_mem[:, 'ENSMUSG00000070880'].X > 0, 'ENSMUSG00000070880']. Optionally, where an HPC cluster is available, time in-memory AnnData for reference (the paper reports ~42x AnnSQL advantage there vs ~746x on a laptop). Compute the AnnSQL-vs-backed-mode speedup ratio, aggregate timings into a summary table, and write the timing table with speedup ratios to gene_filter_runtime_comparison.csv. Judged correct when the CSV contains runtime columns for both AnnSQL and AnnData backed mode and the reproduced laptop speedup is orders of magnitude (hundreds-fold) in favor of AnnSQL, consistent with the paper's reported figure.

## Related tools

- **AnnSQL** (Runs the SQL expression filter on the on-disk AnnData-inspired DuckDB database via asql.query() and provides MakeDb to build the .asql database) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process database engine backing AnnSQL on-disk queries that delivers the runtime advantage being benchmarked)
- **AnnData** (Baseline: same atlas dataset loaded in backed mode with scanpy for boolean-mask subsetting timing comparison)
- **Pandas** (Holds aggregated timing results and writes the summary to gene_filter_runtime_comparison.csv)
- **Python** (Implementation language for the timing benchmark scripts)

## Examples

```
asql = AnnSQL(db='db/atlas.asql'); df = asql.query('SELECT cell_id FROM X WHERE ENSMUSG00000070880 > 0'); df.to_csv('gene_filter_runtime_comparison.csv', index=False)
```

## Evaluation signals

- gene_filter_runtime_comparison.csv exists and contains runtime columns for both AnnSQL and AnnData backed mode
- Reproduced laptop speedup ratio is orders of magnitude (hundreds-fold, ~746x) in favor of AnnSQL, consistent with the paper's reported figure
- Runtime measurements are repeated queries aggregated into a stable summary table
- HPC in-memory AnnData reference ratio (~42x) is computed when a cluster is available; comparison flagged not-applicable otherwise
- Both methods filter the identical gene (ENSMUSG00000070880) on the identical 4.4 million cell atlas dataset

## Limitations

- No software version numbers reported for the tools in the profiling section
- Known unresolved DuckDB memory leak on databases with more than 30k genes
- The AnnSQL database structure mirrors the AnnData object rather than being optimized for performance
- Numeric runtime values are only computed at run time and written to CSV; they are not fixed constants in the article

## Evidence

- [abstract] abstract runtime claim: "AnnSQL-based operations were executed in minutes on a laptop for which equivalent operations in AnnData or Seurat largely failed (or were ∼700× slower)"
- [abstract] engine description: "constructs an AnnData-inspired database using the in-process DuckDb engine, enabling orders-of-magnitude performance enhancements"
- [results] backed-mode filter: "adata_mem[adata_mem[:, "ENSMUSG00000070880"].X > 0, "ENSMUSG00000070880"]"
- [methods] backed-mode improvement: "Running AnnSQL locally for datasets that are larger than memory, that would typically require AnnData in backed mode see substantial runtime improvements"
- [readme] README on-disk usage: "MakeDb(adata=adata, db_name="pbmc3k_reduced", db_path="db/")"
- [readme] README on-disk open: "asql = AnnSQL(db="db/pbmc3k_reduced.asql")"
