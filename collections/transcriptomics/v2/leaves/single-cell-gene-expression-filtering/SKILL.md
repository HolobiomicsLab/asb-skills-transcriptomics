---
name: single-cell-gene-expression-filtering
description: Use when benchmarking or reproducing gene-expression filtering runtime
  on atlas-scale (>memory) single-cell datasets, comparing AnnSQL on-disk SQL queries
  versus AnnData backed-mode subsetting on a laptop-class machine.
when_to_use_negative:
- The dataset fits comfortably in memory and no runtime comparison is needed — use
  in-memory AnnData/scanpy filtering directly.
- The input is not an AnnData/h5ad expression matrix (e.g. already a filtered feature
  table or bulk counts), so the SQL X-table WHERE filter does not apply.
- Only qualitative performance claims are needed without timing measurements — this
  skill is a reproduction/benchmark, not a generic filter utility.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: AnnSQL
  role: Runs the SQL expression filter via asql.query() on the on-disk database using
    the in-process DuckDB engine
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process database engine backing AnnSQL on-disk query execution
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: Loaded in backed mode (>memory datasets) with scanpy to time the equivalent
    boolean-mask subsetting baseline
- name: Pandas
  role: Holds the aggregated timing table written to gene_filter_runtime_comparison.csv
- name: Python
  role: Implementation language for the benchmark scripts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:4368674455e4754a641733121e95654954139bf861b6832efb416f0486fcf664
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# single-cell-gene-expression-filtering

## Summary

Reproduces AnnSQL's large-scale single-cell gene expression filtering benchmark: a SQL WHERE-clause filter (e.g. WHERE ENSMUSG00000070880 > 0) on a 4.4 million cell atlas via the in-process DuckDB engine, timed against the equivalent backed-mode AnnData boolean-mask subsetting to quantify the ~746x laptop speedup.

## When to use

Use when benchmarking or reproducing gene-expression filtering runtime on atlas-scale (>memory) single-cell datasets, comparing AnnSQL on-disk SQL queries versus AnnData backed-mode subsetting on a laptop-class machine.

## When NOT to use

- The dataset fits comfortably in memory and no runtime comparison is needed — use in-memory AnnData/scanpy filtering directly.
- The input is not an AnnData/h5ad expression matrix (e.g. already a filtered feature table or bulk counts), so the SQL X-table WHERE filter does not apply.
- Only qualitative performance claims are needed without timing measurements — this skill is a reproduction/benchmark, not a generic filter utility.

## Inputs

- On-disk AnnSQL database (.asql) built from a 4.4 million cell atlas AnnData object
- Atlas-scale h5ad file (Macosko Mouse Atlas Single Nuclei, 4.4 million cells) for backed-mode AnnData loading
- Gene identifier to filter on, e.g. ENSMUSG00000070880, with expression > 0 threshold

## Outputs

- gene_filter_runtime_comparison.csv with runtime columns for AnnSQL and AnnData backed mode
- AnnSQL-vs-backed-mode speedup ratio (~746x expected on laptop; ~42x over HPC in-memory AnnData if measured)

## How to apply

Open the atlas-scale on-disk AnnSQL database with AnnSQL(db=...) on a laptop-class machine, which runs the in-process DuckDB engine, and time the SQL expression filter 'SELECT ... WHERE ENSMUSG00000070880 > 0' via asql.query(), repeating the query for a stable measurement. Then load the same 4.4 million cell Macosko Mouse Atlas Single Nuclei h5ad in AnnData backed mode with scanpy and time the equivalent boolean-mask subset adata_mem[adata_mem[:, 'ENSMUSG00000070880'].X > 0, 'ENSMUSG00000070880']. Optionally, if an HPC cluster is available, also time in-memory AnnData for reference (paper reports ~42x AnnSQL advantage there). Aggregate timings into a summary table, compute the AnnSQL-vs-backed-mode speedup ratio (target ~746x), and write the table to gene_filter_runtime_comparison.csv. Validate by checking the CSV has runtime columns for both methods and that the reproduced laptop ratio is hundreds-fold in favor of AnnSQL; flag the HPC comparison as not-applicable if no cluster is available.

## Related tools

- **AnnSQL** (Runs the SQL expression filter via asql.query() on the on-disk database using the in-process DuckDB engine) — https://github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process database engine backing AnnSQL on-disk query execution) — https://github.com/ArpiarSaundersLab/annsql
- **AnnData** (Loaded in backed mode (>memory datasets) with scanpy to time the equivalent boolean-mask subsetting baseline)
- **Pandas** (Holds the aggregated timing table written to gene_filter_runtime_comparison.csv)
- **Python** (Implementation language for the benchmark scripts)

## Examples

```
from AnnSQL import AnnSQL; asql = AnnSQL(db="db/atlas.asql"); df = asql.query("SELECT cell_id FROM X WHERE ENSMUSG00000070880 > 0")
```

## Evaluation signals

- gene_filter_runtime_comparison.csv exists with runtime columns for AnnSQL and AnnData backed mode
- Reproduced laptop speedup ratio is orders of magnitude (~hundreds-fold, target ~746x) in favor of AnnSQL, consistent with the paper's runtime comparison figure
- If HPC is measured, the AnnSQL-over-in-memory-AnnData ratio is ~42x; otherwise the HPC comparison is explicitly flagged not-applicable
- Timings are stable (repeated query runs) rather than single cold-start measurements
- Both methods return the same set of cells passing ENSMUSG00000070880 > 0 for the filter to be an equivalent comparison

## Limitations

- No software version numbers are reported for the tools, so exact version pinning must be determined independently.
- Known unresolved DuckDB memory leak on databases with more than 30k genes may affect atlas-scale builds.
- The AnnSQL database structure mirrors AnnData objects and is not optimized for performance beyond what the in-process DuckDB engine provides.
- No numeric memory/runtime values appear in the paper's code — results are computed at runtime and written to CSV files, so expected values must be reproduced, not read off.

## Evidence

- [results] On the 4.4 million cell atlas dataset, backed-mode AnnData filtering could be performed on a laptop, but the equivalent AnnSQL filtering procedure was approximately 746 times faster.: [evidence span withheld — anchor status: not_found]
- [results] Backed-mode subsetting code used in the benchmark.: "adata_mem[adata_mem[:, "ENSMUSG00000070880"].X > 0, "ENSMUSG00000070880"]"
- [methods] AnnSQL opens on-disk databases with the in-process DuckDB engine.: "asql = AnnSQL(db="db/pbmc3k_reduced.asql")"
- [intro] AnnSQL converts AnnData layers into a relational DuckDB database.: "AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database. Each layer is stored as an individual table"
- [readme] README documents on-disk usage for larger datasets.: [evidence span withheld — anchor status: external_doc]
