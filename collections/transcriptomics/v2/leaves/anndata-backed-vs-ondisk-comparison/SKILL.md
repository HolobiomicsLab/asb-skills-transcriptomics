---
name: anndata-backed-vs-ondisk-comparison
description: Use when benchmarking AnnSQL against AnnData on simulated single-cell
  data, or when testing whether runtime conclusions transfer from randomly generated
  matrices to sparser Splatter-simulated UMI-count datasets of the same shapes.
when_to_use_negative:
- The goal is measuring absolute runtime on real experimental data (e.g. pbmc3k, pbmc68k_reduced,
  or the 4.4M-cell Macosko atlas) rather than the Splatter-simulated in silico comparison
- Datasets exceed the known DuckDB memory-leak threshold of more than 30k genes
- You need software-version-pinned benchmark results — the article reports no version
  numbers, so exact reproduction of numeric runtimes is not guaranteed
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AnnSQL
  role: Provides in-memory (AnnSQL(adata=adata)) and on-disk (MakeDb + AnnSQL(db=...))
    database modes timed in the benchmark; installed via pip
  repo: github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: Baseline storage/query layer benchmarked in not-backed (in-memory) and backed
    (on-disk h5ad) modes
- name: Splatter
  role: Generates the sparser in silico UMI-count datasets matching the shapes of
    the randomly generated benchmark data
- name: DuckDB
  role: In-process engine underlying AnnSQL's relational database representation of
    AnnData layers
- name: Python
  role: Environment (Python 3.12 conda) used to run the simulation, timing, statistics,
    and plotting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:21fbd156c5e9c842886c63309ecabb857ce01efa29e83ed00ce4f04c95e1e3bc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Python
  - Splatter
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# anndata-backed-vs-ondisk-comparison

## Summary

Reproduce the AnnSQL-versus-AnnData runtime benchmark on Splatter-simulated in silico scRNA-seq datasets matching the shapes of the article's randomly generated data (up to 250,000 cells × 10,000 genes), demonstrating that sparser UMI-count data yields lower runtimes across AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, and AnnData backed modes.

## When to use

Use when benchmarking AnnSQL against AnnData on simulated single-cell data, or when testing whether runtime conclusions transfer from randomly generated matrices to sparser Splatter-simulated UMI-count datasets of the same shapes.

## When NOT to use

- The goal is measuring absolute runtime on real experimental data (e.g. pbmc3k, pbmc68k_reduced, or the 4.4M-cell Macosko atlas) rather than the Splatter-simulated in silico comparison
- Datasets exceed the known DuckDB memory-leak threshold of more than 30k genes
- You need software-version-pinned benchmark results — the article reports no version numbers, so exact reproduction of numeric runtimes is not guaranteed

## Inputs

- Splatter-simulated in silico AnnData objects (sparse UMI counts) sized 1,000–250,000 cells × 10,000 genes
- Randomly generated benchmark AnnData objects of identical shapes (for runtime delta comparison)
- Python 3.12 environment with annsql (pip install) and the Splatter simulation framework

## Outputs

- Aggregated runtime comparison plot across AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, and AnnData backed
- Runtime table written via df.to_csv(index=False)
- Paired t-test results with Bonferroni-corrected alpha (alpha_corrected = 0.05 / num_comparisons)

## How to apply

Set up a Python 3.12 conda environment and pip install annsql, then generate Splatter-simulated datasets with dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000] cells and columns = 10000 genes, matching the random-data shapes. Build on-disk AnnSQL databases from each simulated AnnData object with MakeDb using db_name='random_data_'+str(i), create_all_indexes=False, convenience_view=False, and instantiate corresponding in-memory AnnSQL objects. Time the filter/query benchmark operations against AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, and AnnData backed for every size. Aggregate runtimes, apply paired t-tests with Bonferroni correction (alpha_corrected = 0.05 / num_comparisons), plot the aggregated runtime comparison, and write the runtime table with df.to_csv(index=False). Validate by confirming that all measured runtimes decreased relative to the random-data benchmark, reflecting the sparser UMI counts of Splatter output.

## Related tools

- **AnnSQL** (Provides in-memory (AnnSQL(adata=adata)) and on-disk (MakeDb + AnnSQL(db=...)) database modes timed in the benchmark; installed via pip) — github.com/ArpiarSaundersLab/annsql
- **AnnData** (Baseline storage/query layer benchmarked in not-backed (in-memory) and backed (on-disk h5ad) modes)
- **Splatter** (Generates the sparser in silico UMI-count datasets matching the shapes of the randomly generated benchmark data)
- **DuckDB** (In-process engine underlying AnnSQL's relational database representation of AnnData layers)
- **Python** (Environment (Python 3.12 conda) used to run the simulation, timing, statistics, and plotting)

## Examples

```
from AnnSQL import AnnSQL
from AnnSQL.MakeDb import MakeDb
MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)
```

## Evaluation signals

- Measured runtimes of every AnnSQL-versus-AnnData comparison decrease on the Splatter-simulated data relative to the random-data benchmark, confirming the sparser-data finding
- All 14 dataset sizes (1,000 through 250,000 cells) complete with columns = 10000 genes per dataset
- Paired t-tests across method comparisons use the Bonferroni-corrected threshold alpha_corrected = 0.05 / num_comparisons and the corrected results are reported
- Runtime table CSV (df.to_csv(index=False)) contains one runtime entry per method mode (AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, AnnData backed) per dataset size
- On-disk databases are built with the benchmark settings create_all_indexes=False and convenience_view=False, matching the article's MakeDb configuration

## Limitations

- DuckDB has a known unresolved memory leak on databases with more than 30k genes (TODO debug note in the code)
- No software version numbers are reported for any tool in the benchmark code, so exact numeric reproduction is not guaranteed
- No numeric memory/runtime values appear in the article's code; results are only computed at runtime and written to CSV files, so expected magnitudes must be re-derived empirically
- The AnnSQL database structure mirrors the AnnData object structure rather than being optimized for performance

## Evidence

- [results] Splatter simulation matches random-data shapes: "dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000]

columns = 10000"
- [results] Benchmark MakeDb settings: "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [results] Bonferroni-corrected paired t-tests: "# Adjusted alpha using Bonferroni correction
alpha_corrected = 0.05 / num_comparisons"
- [methods] pip install command: "conda create -n annsql python=3.12"
- [results] DuckDB memory leak caveat: "#TODO debug duckdb memory leak on daabases with >30k genes."
- [readme] AnnSQL converts AnnData layers to DuckDB: "Behind the scenes, AnnSQL converts the layers of an AnnData object into a relational [DuckDB](https://duckdb.org/) database."
