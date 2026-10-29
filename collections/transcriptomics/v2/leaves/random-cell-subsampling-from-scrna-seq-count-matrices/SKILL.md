---
name: random-cell-subsampling-from-scrna-seq-count-matrices
description: Use when benchmarking or comparing runtime scaling of scRNA-seq pipelines
  across dataset sizes, and you need controlled random cell subsets drawn from a large
  public count matrix.
when_to_use_negative:
- The goal is a definitive biological result rather than a performance benchmark —
  subsampled cells may not preserve population structure or rare cell types
- The pipeline under test does not accept .csv or .parquet inputs and format conversion
  would confound the timing measurement
- The dataset is already small enough that runtime scaling differences cannot be distinguished
  from noise
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Pipeline whose runtime is benchmarked on each randomly sampled cell subsample
    (data melting, quality summaries, data_filter(), normalization, PCA)
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Execution environment (single-node or two-node cluster) under which per-subsample
    timings are recorded
- name: 10x Genomics
  role: Source of the public 68K PBMC dataset from which random cell subsets are drawn
- name: Apache Parquet
  role: Columnar file format used to write each cell subsample for pipeline input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:00ebcff707ce2eb6430d4489303869b19aee29af916ba6b72e932d3abf3c8496
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/asif7adil/scSparkl
  tools:
  - 10x Genomics
  - Apache Parquet
  - Apache Spark
  - Python
  - scSPARKL
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Random cell subsampling from scRNA-seq count matrices

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Draws random cell subsets of increasing size (e.g., 2,000–10,000 cells from the 68K PBMC dataset, Zheng et al. 2017, 10x Genomics) to generate benchmark inputs for measuring how an scRNA-seq pipeline's runtime scales with cell count under different compute configurations.

## When to use

Use when benchmarking or comparing runtime scaling of scRNA-seq pipelines across dataset sizes, and you need controlled random cell subsets drawn from a large public count matrix.

## When NOT to use

- The goal is a definitive biological result rather than a performance benchmark — subsampled cells may not preserve population structure or rare cell types
- The pipeline under test does not accept .csv or .parquet inputs and format conversion would confound the timing measurement
- The dataset is already small enough that runtime scaling differences cannot be distinguished from noise

## Inputs

- Public scRNA-seq count matrix (e.g., 68K PBMC dataset from Zheng et al. 2017, 10x Genomics)
- List of target subsample sizes (e.g., 2,000–10,000 cells)
- Random seed or sampling specification for reproducibility

## Outputs

- Per-subsample count matrix files in .csv or .parquet format
- execution_times.csv containing one timing entry per subsample size per configuration

## How to apply

Obtain a large public scRNA-seq dataset, such as the 68K PBMC dataset (Zheng et al. 2017, 10x Genomics), and draw random cell samples at a series of increasing sizes (e.g., 2,000 to 10,000 cells in steps). Write each subsample as a .csv or .parquet input file, the formats the scSPARKL pipeline currently accepts. Feed each subsample through the identical pipeline — data melting to tall format, cell/gene quality summaries, data_filter(), normalization, and PCA — under each Spark configuration being compared, recording total execution time per subsample size (e.g., in execution_times.csv). Use random sampling to avoid selection bias and keep all other pipeline parameters fixed so runtime differences reflect scale, not configuration drift.

## Related tools

- **scSPARKL** (Pipeline whose runtime is benchmarked on each randomly sampled cell subsample (data melting, quality summaries, data_filter(), normalization, PCA)) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Execution environment (single-node or two-node cluster) under which per-subsample timings are recorded)
- **10x Genomics** (Source of the public 68K PBMC dataset from which random cell subsets are drawn)
- **Apache Parquet** (Columnar file format used to write each cell subsample for pipeline input)

## Examples

```
python run_scsparkl.py --input pbmc_10000.parquet --config spark_two_node; echo $SECONDS >> execution_times.csv
```

## Evaluation signals

- execution_times.csv contains a timing entry for every subsample size in the planned range (2K–10K)
- Each run completed the full pipeline (melting, quality summaries, data_filter(), normalization, PCA) under the chosen Spark configuration
- Reported runtimes fall in the paper's reference order of magnitude (e.g., total computation as low as ~8 min on a PBMC dataset; ~10 min on a two-node Spark cluster for the 10,000-cell subsample vs >50 min sequential)
- Subsample sizes increase monotonically and each .csv/.parquet file contains the intended number of cells

## Limitations

- scSPARKL currently accepts only .csv and .parquet inputs, so other formats require conversion before subsampling can feed the pipeline
- Runtime comparisons reflect only the tested Spark configurations; the paper notes no benchmarking against Seurat and Scanpy and no GPU-accelerated configuration benchmarks
- Random subsampling captures scaling behavior, not biological completeness — conclusions apply to performance, not to cell-type discovery

## Evidence
