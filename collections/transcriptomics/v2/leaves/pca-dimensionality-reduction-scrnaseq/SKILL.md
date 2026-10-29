---
name: pca-dimensionality-reduction-scrnaseq
description: Use when a normalized scRNA-seq expression matrix (tall or wide format,
  e.g. CSV/Parquet) must be projected to a low-dimensional space to enable scalable
  clustering on large cell counts.
when_to_use_negative:
- Input is raw (unnormalized) scRNA-seq count data — scSPARKL requires the melting,
  filtering, and normalization stages to run first
- Input file formats other than .csv or .parquet — the README states only these are
  currently supported
- Single-cell multi-omics data (CITE-seq, scATAC-seq, scBS-seq) — no Spark-based pipelines
  exist yet for these modalities
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scSPARKL
  role: Provides the exclusively Spark-based PCA implementation for dimensionality
    reduction of scRNA-seq data
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed in-memory engine on which the PCA routine executes, enabling scalable
    dimension reduction on commodity hardware
- name: Apache Parquet
  role: Columnar storage format used alongside Spark for efficient data handling in
    the pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:729a08db680a4695e26ce2fb7b070b6589d0893b4e8ff5156b516e8c5d4fece2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/asif7adil/scSparkl
  tools:
  - Apache Parquet
  - Apache Spark
  - scSPARKL
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# pca-dimensionality-reduction-scrnaseq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Perform PCA-based dimensionality reduction on normalized scRNA-seq expression data using scSPARKL's Spark-native PCA implementation, producing a reduced representation that feeds downstream K-means clustering of cell populations.

## When to use

Use when a normalized scRNA-seq expression matrix (tall or wide format, e.g. CSV/Parquet) must be projected to a low-dimensional space to enable scalable clustering on large cell counts.

## When NOT to use

- Input is raw (unnormalized) scRNA-seq count data — scSPARKL requires the melting, filtering, and normalization stages to run first
- Input file formats other than .csv or .parquet — the README states only these are currently supported
- Single-cell multi-omics data (CITE-seq, scATAC-seq, scBS-seq) — no Spark-based pipelines exist yet for these modalities

## Inputs

- Normalized single-cell expression matrix in wide and/or tall format (.csv or .parquet)
- Selected highly variable gene set (MAD k > 3 or top-n by coefficient of variance squared)

## Outputs

- PCA-reduced cell representation stored for downstream clustering
- Reduced-dimension data written via Apache Parquet for scalable storage

## How to apply

After scSPARKL data melting, quality filtering via data_filter() (e.g. percent_ercc > 10%, percent_mito > 5%, cells expressing < 10 genes, genes expressed in < 3 cells, dropout rate > 70%), and normalization (Global/CPM-like or Quantile), run PCA on the selected highly variable genes. Gene selection is done via Median Absolute Deviation (default threshold k > 3) or Coefficient of Variance Squared returning top n genes. The PCA implementation in scSPARKL is exclusively Spark-based, so it operates on distributed DataFrames rather than in-memory matrices, avoiding the RAM constraints of Seurat/Scanpy-style pipelines. The PCA-reduced representation is then passed to K-means clustering, with the number of clusters k selected via elbow plot and silhouette score analysis.

## Related tools

- **scSPARKL** (Provides the exclusively Spark-based PCA implementation for dimensionality reduction of scRNA-seq data) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed in-memory engine on which the PCA routine executes, enabling scalable dimension reduction on commodity hardware)
- **Apache Parquet** (Columnar storage format used alongside Spark for efficient data handling in the pipeline)

## Examples

```
Download the source code and run either the Jupyter Notebook or the scSPARKL script file.  ### Please note: currently the pipeline uses `.csv` and `.parquet` as input files for the analysis.
```

## Evaluation signals

- PCA output feeds K-means clustering that achieves ARI ~0.90 against expected human vs mouse separation on the Jurkat–293T cell mixture
- The reduced representation supports cluster separation judged by silhouette score, with k chosen at the elbow of the within-cluster sum of squares plot
- Full upstream pipeline (QC, filtering, normalization, dimension reduction, clustering, visualization) completes in less than 10 minutes
- Input validation: normalized tall/wide DataFrames only, from .csv or .parquet sources

## Limitations

- Repository is not actively maintained; a new version is under development
- Spark cluster mode operations are not yet optimized, as each node has separate memory allocation
- Data melting workaround for RAM limitations needs further optimization
- Only tested on Windows 11; requires Spark 3.0+, Java, and Python 3.9+ with careful executor/driver memory tuning

## Evidence
