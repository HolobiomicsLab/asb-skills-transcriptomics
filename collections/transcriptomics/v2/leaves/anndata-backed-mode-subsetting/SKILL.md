---
name: anndata-backed-mode-subsetting
description: Use when a .h5ad dataset is larger than memory and you need boolean-mask
  cell/gene subsetting in AnnData backed mode, or when benchmarking that baseline
  against AnnSQL on-disk SQL filters.
when_to_use_negative:
- The AnnData object fits fully in memory — backed mode adds I/O overhead versus normal
  (not-backed) in-memory subsetting
- The task requires complex aggregative SQL queries (joins, group-by averages, gene
  ratios) rather than a simple expression boolean mask — AnnSQL is the paper's recommended
  path
- A quick exploratory subset is needed and runtime is irrelevant; a one-off backed-mode
  load suffices without benchmarking infrastructure
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: AnnData
  role: Provides backed-mode on-disk matrix storage; subsetting is performed with
    boolean masks on adata_mem slices
- name: Scanpy
  role: Loads the .h5ad atlas file via sc.read_h5ad in backed mode for datasets larger
    than memory
- name: AnnSQL
  role: Executes the equivalent SQL expression filter (SELECT ... WHERE ENSMUSG00000070880
    > 0) via asql.query() on an on-disk DuckDB database for the speedup comparison
  repo: github.com/ArpiarSaundersLab/annsql
- name: DuckDB
  role: In-process engine backing the AnnSQL on-disk database used as the fast comparison
    arm
- name: Pandas
  role: Assembles and writes the runtime summary table to gene_filter_runtime_comparison.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:189c78270fee835a8e9a0485d0773b82cce5959443bd0620baa1317f10ee3fbf
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# anndata-backed-mode-subsetting

## Summary

Reproduces the paper's baseline AnnData backed-mode gene-expression subsetting on on-disk .h5ad single-cell datasets (e.g., the 4.4M-cell Macosko Mouse Atlas Single Nuclei file) and times it against the equivalent AnnSQL/DuckDB filter to quantify the reported ~746x laptop speedup.

## When to use

Use when a .h5ad dataset is larger than memory and you need boolean-mask cell/gene subsetting in AnnData backed mode, or when benchmarking that baseline against AnnSQL on-disk SQL filters.

## When NOT to use

- The AnnData object fits fully in memory — backed mode adds I/O overhead versus normal (not-backed) in-memory subsetting
- The task requires complex aggregative SQL queries (joins, group-by averages, gene ratios) rather than a simple expression boolean mask — AnnSQL is the paper's recommended path
- A quick exploratory subset is needed and runtime is irrelevant; a one-off backed-mode load suffices without benchmarking infrastructure

## Inputs

- On-disk .h5ad single-cell AnnData file larger than memory (e.g., Macosko Mouse Atlas Single Nuclei, 4.4 million cells)
- Target gene identifier for the expression filter (e.g., ENSMUSG00000070880)
- AnnSQL on-disk .asql database for the same dataset (speedup comparison baseline)

## Outputs

- gene_filter_runtime_comparison.csv with runtime columns for AnnSQL and AnnData backed mode and computed speedup ratios
- Timing measurements (stable, repeated) for the backed-mode boolean-mask subsetting
- Speedup ratio (target ~746x in favor of AnnSQL on laptop; ~42x over HPC in-memory AnnData where available)

## How to apply

Load the on-disk .h5ad (e.g., Macosko_Mouse_Atlas_Single_Nuclei, 4.4 million cells) with scanpy in backed mode so the matrix is not fully materialized in memory. Time the boolean-mask subsetting pattern adata_mem[adata_mem[:, "ENSMUSG00000070880"].X > 0, "ENSMUSG00000070880"], which forces a full scan of the backed expression matrix for the gene column. Repeat the timing to obtain a stable measurement, then compute the speedup ratio versus the equivalent AnnSQL on-disk query 'SELECT ... WHERE ENSMUSG00000070880 > 0' via asql.query(). Write runtimes for both methods to gene_filter_runtime_comparison.csv; optionally add an HPC in-memory AnnData reference (~42x slower than AnnSQL per the paper), flagging it not-applicable if no cluster is available.

## Related tools

- **AnnData** (Provides backed-mode on-disk matrix storage; subsetting is performed with boolean masks on adata_mem slices)
- **Scanpy** (Loads the .h5ad atlas file via sc.read_h5ad in backed mode for datasets larger than memory)
- **AnnSQL** (Executes the equivalent SQL expression filter (SELECT ... WHERE ENSMUSG00000070880 > 0) via asql.query() on an on-disk DuckDB database for the speedup comparison) — github.com/ArpiarSaundersLab/annsql
- **DuckDB** (In-process engine backing the AnnSQL on-disk database used as the fast comparison arm)
- **Pandas** (Assembles and writes the runtime summary table to gene_filter_runtime_comparison.csv)

## Examples

```
import scanpy as sc; adata_mem = sc.read_h5ad("Macosko_Mouse_Atlas_Single_Nuclei.h5ad", backed="r"); sub = adata_mem[adata_mem[:, "ENSMUSG00000070880"].X > 0, "ENSMUSG00000070880"]
```

## Evaluation signals

- gene_filter_runtime_comparison.csv exists and contains runtime columns for both AnnSQL and AnnData backed mode
- The reproduced laptop speedup ratio is orders of magnitude (~hundreds-fold, target ~746x) in favor of AnnSQL
- Timings are aggregated over repeated query runs to yield a stable measurement
- The HPC in-memory AnnData comparison (~42x) is either reproduced or explicitly flagged not-applicable when no cluster is available
- Both arms filter the same dataset (4.4M-cell Macosko Mouse Atlas) with the same predicate (ENSMUSG00000070880 > 0), so the comparison is apples-to-apples

## Limitations

- Backed-mode AnnData filtering on the 4.4M-cell atlas is extremely slow (~746x slower than AnnSQL on a laptop) and equivalent AnnData/Seurat operations 'largely failed' on the paper's HPC reference
- No software version numbers are reported in the results section, so exact reproducibility of timings is not guaranteed
- No numeric runtime values are published in the article (results are computed at runtime and written to CSV), so only the ratio, not absolute times, can be validated
- DuckDB comparison arm has a known unresolved memory leak on databases with more than 30k genes (TODO in the paper's code)

## Evidence

- [results] Backed-mode AnnData filtering could be performed on a laptop, but the equivalent AnnSQL filtering procedure was approximately 746 times faster.: [evidence span withheld — anchor status: not_found]
- [results] Backed-mode subsetting code snippet: "adata_mem[adata_mem[:, "ENSMUSG00000070880"].X > 0, "ENSMUSG00000070880"]"
- [results] Load atlas dataset in backed mode: "#load the atlas dataset (4.4 million cells) in backed mode"
- [methods] AnnSQL runtime improvements over backed mode: "Running AnnSQL locally for datasets that are larger than memory, that would typically require AnnData in backed mode see substantial runtime improvements"
- [readme] README description of AnnData-to-DuckDB conversion: "Behind the scenes, AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database."
