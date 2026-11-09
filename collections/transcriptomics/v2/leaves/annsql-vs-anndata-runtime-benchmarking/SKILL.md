---
name: annsql-vs-anndata-runtime-benchmarking
description: 'Use when reproducing or adapting the AnnSQL runtime benchmark: comparing
  in-memory AnnSQL vs in-memory AnnData query runtimes across simulated dataset sizes
  with a 45,000-cell crossover expectation.'
when_to_use_negative:
- The dataset exceeds available memory — the paper's in-memory comparisons apply only
  when both AnnData and the AnnSQL database fit in RAM; use AnnSQL on-disk (MakeDb)
  versus AnnData backed mode instead
- The analysis goal is accuracy or result equivalence rather than runtime — this skill
  measures query latency only and does not validate that AnnSQL and AnnData return
  identical results
- The target dataset has more than 30,000 genes — a known DuckDB memory leak affects
  databases with >30k genes
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: AnnSQL
  role: Provides in-memory AnnData-to-DuckDB database instantiation (AnnSQL(adata=adata))
    and the query() interface being benchmarked
  repo: github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: Baseline framework whose in-memory (not-backed) query runtimes are compared
    against AnnSQL at each simulated dataset size
- name: DuckDB
  role: In-process SQL engine underlying AnnSQL databases; converts each AnnData layer
    into an individual relational table
- name: Pandas
  role: Returns AnnSQL query results as DataFrames for runtime-timed comparisons
- name: Python
  role: Runtime environment (Python 3.12 conda env) hosting the benchmark scripts
    and timing loops
- name: Scanpy
  role: Supplies AnnData sample datasets (pbmc3k, pbmc68k_reduced) and h5ad loading
    used to exercise the database construction before benchmarking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:7dfe4081bc6f4df3f91f018bc6562d94a5096cc82b0932bfbfbc6b43b28a6153
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Pandas
  - Python
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# annsql-vs-anndata-runtime-benchmarking

## Summary

Benchmark methodology for comparing query runtimes of in-memory AnnSQL (DuckDB-backed) against in-memory AnnData on simulated dense scRNA-seq matrices from 1,000 to 100,000 cells with 10,000 genes, aggregating per-trial runtimes into a single comparison table.

## When to use

Use when reproducing or adapting the AnnSQL runtime benchmark: comparing in-memory AnnSQL vs in-memory AnnData query runtimes across simulated dataset sizes with a 45,000-cell crossover expectation.

## When NOT to use

- The dataset exceeds available memory — the paper's in-memory comparisons apply only when both AnnData and the AnnSQL database fit in RAM; use AnnSQL on-disk (MakeDb) versus AnnData backed mode instead
- The analysis goal is accuracy or result equivalence rather than runtime — this skill measures query latency only and does not validate that AnnSQL and AnnData return identical results
- The target dataset has more than 30,000 genes — a known DuckDB memory leak affects databases with >30k genes

## Inputs

- Simulated dense scRNA-seq count matrices at dataset sizes 1,000–100,000 cells with 10,000 genes per dataset
- In-memory AnnData objects constructed from the simulated matrices
- In-memory AnnSQL databases instantiated via AnnSQL(adata=adata)

## Outputs

- runtimes_table.csv with per-size runtimes of in-memory AnnData vs in-memory AnnSQL
- Aggregated runtime comparison confirming the <45,000-cell crossover and <0.01 s sub-75,000-cell pattern

## How to apply

Install annsql via pip in a Python 3.12 conda environment (conda create -n annsql python=3.12; pip install annsql) and obtain the benchmark scripts from the ArpiarSaundersLab/annsql GitHub repository. Generate simulated dense single-cell count matrices at the paper's sizes (1,000; 5,000; 10,000; 15,000; 20,000; 25,000; 30,000; 35,000; 40,000; 45,000; 50,000; 75,000; 100,000 cells) with 10,000 genes per dataset. Instantiate each matrix as an in-memory AnnData object and as an in-memory AnnSQL database via AnnSQL(adata=adata). Execute equivalent filter/query operations on both frameworks (gene expression threshold queries such as gene_1 > 0.5, compound two-gene filters, gene-ratio conditions, cell/gene count filters) and record per-trial runtimes. Aggregate per-size runtimes into one table of in-memory AnnData vs in-memory AnnSQL and write it to runtimes_table.csv. Validate against the reported pattern: in-memory AnnData wins only below 45,000 cells, and both methods stay under 0.01 s for libraries below 75,000 cells.

## Related tools

- **AnnSQL** (Provides in-memory AnnData-to-DuckDB database instantiation (AnnSQL(adata=adata)) and the query() interface being benchmarked) — github.com/ArpiarSaundersLab/annsql
- **AnnData** (Baseline framework whose in-memory (not-backed) query runtimes are compared against AnnSQL at each simulated dataset size)
- **DuckDB** (In-process SQL engine underlying AnnSQL databases; converts each AnnData layer into an individual relational table)
- **Pandas** (Returns AnnSQL query results as DataFrames for runtime-timed comparisons)
- **Python** (Runtime environment (Python 3.12 conda env) hosting the benchmark scripts and timing loops)
- **Scanpy** (Supplies AnnData sample datasets (pbmc3k, pbmc68k_reduced) and h5ad loading used to exercise the database construction before benchmarking)

## Examples

```
conda create -n annsql python=3.12 && pip install annsql; asql = AnnSQL(adata=adata); asql.query("SELECT gene_1 FROM X WHERE gene_1 > 0.5 LIMIT 5")
```

## Evaluation signals

- runtimes_table.csv contains one row per simulated dataset size (13 sizes from 1,000 to 100,000 cells) for both in-memory AnnData and in-memory AnnSQL
- In-memory AnnData runtimes are lower than in-memory AnnSQL runtimes only for libraries with fewer than 45,000 cells
- Both frameworks achieve runtimes under 0.01 seconds for libraries with fewer than 75,000 cells
- Each simulated dataset contains exactly 10,000 genes, matching the paper's columns = 10000 configuration
- Per-size comparisons are consistent with paired t-tests with Bonferroni-corrected alpha (0.05 / number of dataset sizes)

## Limitations

- No software version numbers were reported for any tool in the benchmark section
- DuckDB has a known unresolved memory leak on databases with more than 30k genes (TODO in source)
- The AnnSQL database structure mirrors the AnnData object and is not optimized for performance
- Numeric runtime results are computed at run time and written to CSV; no static result values appear in the source code
- Paper reports runtime advantage over AnnData backed mode and Seurat mainly for datasets larger than memory — the in-memory crossover at 45,000 cells does not generalize to on-disk comparisons

## Evidence

- [methods] In-memory crossover threshold: "dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000]"
- [methods] In-memory instantiation: "asql = AnnSQL(adata=adata)"
- [results] SQL query interface: "adata_sql_memory.query("SELECT gene_1 FROM X WHERE gene_1 > 0.5 LIMIT 5")"
- [results] Statistics approach: "# Adjusted alpha using Bonferroni correction
alpha_corrected = 0.05 / num_comparisons"
- [readme] Query output format: "#query the expression table. Returns Pandas Dataframe by Default
asql.query("SELECT * FROM adata LIMIT 10")"
