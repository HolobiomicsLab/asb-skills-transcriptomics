---
name: sparse-umi-count-dataset-generation
description: Use when benchmarking single-cell analysis tools and you need controlled,
  in silico UMI-count datasets whose shapes (cells x genes) match real or randomly
  generated data, especially to isolate the effect of sparsity on runtime or memory.
when_to_use_negative:
- You need realistic biological structure or specific cell types rather than shape-controlled
  synthetic data — use a real dataset such as pbmc3k or the 4.4M-cell Macosko Mouse
  Atlas instead
- Databases will have more than 30k genes, where a known DuckDB memory leak issue
  is unresolved
- You require exact reproducibility of the original benchmark's density, since Splatter
  UMI counts are sparser and runtimes will not match random-data results
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Splatter
  role: Generates the in silico sc/nRNA-seq sparse UMI-count datasets matching the
    shapes of the random simulated data
- name: AnnSQL
  role: Consumes the simulated AnnData objects to build in-memory and on-disk DuckDB-backed
    databases for the runtime benchmark
  repo: https://github.com/ArpiarSaundersLab/annsql
- name: AnnData
  role: In-memory container format for the simulated matrices and the not-backed/backed
    comparison baseline in the runtime experiment
- name: DuckDB
  role: In-process engine underlying AnnSQL on-disk databases built from the simulated
    data
  repo: https://github.com/duckdb/duckdb-go
- name: Python
  role: Environment (3.12) and scripting language for simulation conversion, timing,
    statistics, and plotting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbaf105
    title: 'AnnSQL: a Python SQL-based package for fast large-scale single-cell genomics
      analysis using minimal computational resources'
  source_content_hash: sha256:dccddcd43a7a83ab604e0823f66f7062e686fccfacfe639da7777af12812c80a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ArpiarSaundersLab/annsql
  tools:
  - AnnData
  - AnnSQL
  - DuckDB
  - Python
  - Splatter
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Sparse UMI-count dataset generation

## Summary

Generate in silico single-cell datasets with Splatter that match the shapes of existing benchmark data (e.g., dataset_sizes = [1000 ... 250000] cells with columns = 10000 genes), yielding sparser UMI-count matrices used to test how data sparsity affects tool runtimes such as AnnSQL versus AnnData.

## When to use

Use when benchmarking single-cell analysis tools and you need controlled, in silico UMI-count datasets whose shapes (cells x genes) match real or randomly generated data, especially to isolate the effect of sparsity on runtime or memory.

## When NOT to use

- You need realistic biological structure or specific cell types rather than shape-controlled synthetic data — use a real dataset such as pbmc3k or the 4.4M-cell Macosko Mouse Atlas instead
- Databases will have more than 30k genes, where a known DuckDB memory leak issue is unresolved
- You require exact reproducibility of the original benchmark's density, since Splatter UMI counts are sparser and runtimes will not match random-data results

## Inputs

- Reference dataset shapes (list of cell counts, e.g. [1000..250000], with 10000 gene columns)
- Splatter simulation framework (R) for generating sparse UMI-count in silico datasets
- Python 3.12 environment with annsql installed via pip

## Outputs

- Splatter-simulated AnnData objects at each dataset size (sparser UMI counts)
- On-disk AnnSQL databases (.asql) built with MakeDb from each simulated AnnData object
- Aggregated runtime comparison plot and CSV runtime table (df.to_csv(index=False))
- Paired t-test p-values with Bonferroni-corrected alpha (0.05 / num_comparisons)

## How to apply

Set up a Python 3.12 environment (conda create -n annsql python=3.12; pip install annsql) with the Splatter simulation framework. Simulate datasets with Splatter matching the shapes of the reference data: dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000] with columns = 10000. Convert each simulated AnnData object into an on-disk AnnSQL database with MakeDb (db_name='random_data_'+str(i), create_all_indexes=False, convenience_view=False) and instantiate corresponding in-memory AnnSQL objects. Time the benchmark filter/query operations (e.g., filter_by_cell_counts, SQL queries on X) across AnnSQL in-memory, AnnSQL on-disk, AnnData not-backed, and AnnData backed; compare against the original random-data benchmark using paired t-tests with Bonferroni correction (alpha_corrected = 0.05 / num_comparisons). The rationale is that Splatter data contain sparser UMI counts, so measured runtime decreases reveal how density drives compute cost.

## Related tools

- **Splatter** (Generates the in silico sc/nRNA-seq sparse UMI-count datasets matching the shapes of the random simulated data)
- **AnnSQL** (Consumes the simulated AnnData objects to build in-memory and on-disk DuckDB-backed databases for the runtime benchmark) — https://github.com/ArpiarSaundersLab/annsql
- **AnnData** (In-memory container format for the simulated matrices and the not-backed/backed comparison baseline in the runtime experiment)
- **DuckDB** (In-process engine underlying AnnSQL on-disk databases built from the simulated data) — https://github.com/duckdb/duckdb-go
- **Python** (Environment (3.12) and scripting language for simulation conversion, timing, statistics, and plotting)

## Examples

```
conda create -n annsql python=3.12; pip install annsql; MakeDb(adata=splatter_adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)
```

## Evaluation signals

- All simulated datasets have exactly columns = 10000 genes and match the target dataset_sizes list (1000 through 250000 cells)
- Measured runtimes for every AnnSQL-versus-AnnData comparison decrease relative to the random-data benchmark, confirming the sparser Splatter UMI counts
- On-disk databases are built with the benchmark settings (create_all_indexes=False, convenience_view=False) and open correctly via AnnSQL(db=...)
- Paired t-tests use Bonferroni correction with alpha_corrected = 0.05 / num_comparisons
- Aggregated runtime table is written to CSV with index=False and plotted across all dataset sizes

## Limitations

- Splatter-simulated data are sparser than the random data, so absolute runtimes are not comparable to the original benchmark; only the relative direction (decrease) is interpretable
- No software version numbers are reported for the tools in the benchmark section
- No numeric memory/runtime values are stored in code; results exist only at runtime in CSV outputs, and a known DuckDB memory leak affects databases with >30k genes

## Evidence

- [results] Splatter datasets match shapes of random data: "dataset_sizes = [1000, 5000, 10000, 15000, 20000, 25000, 30000, 35000, 40000, 45000, 50000, 75000, 100000, 250000]"
- [results] MakeDb benchmark settings: "MakeDb(adata, db_path="../db/random/", db_name="random_data_"+str(i), create_all_indexes=False, convenience_view=False)"
- [results] Bonferroni correction of alpha: "# Adjusted alpha using Bonferroni correction
alpha_corrected = 0.05 / num_comparisons"
- [results] Sparse data lower runtimes: "Running AnnSQL locally for datasets that are larger than memory, that would typically require AnnData in backed mode see substantial runtime improvements"
- [readme] Install annsql via pip: "The Python based AnnSQL package enables SQL-based queries on [AnnData](https://anndata.readthedocs.io/en/latest/) objects, returning results as either a [Pandas](https://pandas.pydata.org/) DataFrame"
