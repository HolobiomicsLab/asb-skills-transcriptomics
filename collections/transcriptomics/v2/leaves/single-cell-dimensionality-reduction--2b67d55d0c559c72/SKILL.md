---
name: single-cell-dimensionality-reduction
description: Use when a normalized scRNA-seq expression matrix (wide format, e.g.
  tens of thousands to millions of cells from 10x Genomics data) exceeds single-machine
  memory and you need Spark-native PCA cell embeddings compatible with downstream
  Parquet-based UMAP/t-SNE steps.
when_to_use_negative:
- Input is raw count data that has not passed scSPARKL QC/filtering (e.g. cells expressing
  < 10 genes, percent_mito > 5% not yet removed) and normalization — PCA here expects
  the normalized wide-format matrix
- Dataset fits comfortably in single-machine memory and you need a UMAP/t-SNE embedding
  instead — those steps in scSPARKL run as Python outside Spark, not via the Spark
  PCA routine
- You need a biologically benchmarked reduction validated against Seurat/Scanpy —
  the paper explicitly acknowledges 'No benchmarking against Seurat and Scanpy performed'
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3473
tools:
- name: scSPARKL
  role: Provides the PCA dimension-reduction routine executed over the vectorized
    normalized matrix; script file or Jupyter Notebook from the GitHub repo
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Runtime engine; the PCA is computed with Spark MLlib across nodes (standalone
    or distributed mode depending on data load)
- name: Apache Parquet
  role: Serialization format for the pca_reduced.parquet artifact consumed by downstream
    UMAP/t-SNE and clustering steps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:eea7cd8147bb9f3f06c8317b9083b884cd5b821f20ceeb69ef686fc9c6fad0f2
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-dimensionality-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Distributed PCA-based dimensionality reduction for scRNA-seq data using scSPARKL, implemented exclusively in Spark MLlib so it scales beyond single-machine RAM on normalized wide-format expression matrices.

## When to use

Use when a normalized scRNA-seq expression matrix (wide format, e.g. tens of thousands to millions of cells from 10x Genomics data) exceeds single-machine memory and you need Spark-native PCA cell embeddings compatible with downstream Parquet-based UMAP/t-SNE steps.

## When NOT to use

- Input is raw count data that has not passed scSPARKL QC/filtering (e.g. cells expressing < 10 genes, percent_mito > 5% not yet removed) and normalization — PCA here expects the normalized wide-format matrix
- Dataset fits comfortably in single-machine memory and you need a UMAP/t-SNE embedding instead — those steps in scSPARKL run as Python outside Spark, not via the Spark PCA routine
- You need a biologically benchmarked reduction validated against Seurat/Scanpy — the paper explicitly acknowledges 'No benchmarking against Seurat and Scanpy performed'

## Inputs

- Normalized wide-format scRNA-seq expression matrix (from scSPARKL Global/Simple or Quantile normalization, returned as wide + tall normalized data)
- Spark DataFrame or .csv/.parquet input file (README: 'currently the pipeline uses .csv and .parquet as input files for the analysis')

## Outputs

- pca_reduced.parquet — Parquet file with one reduced-dimension PCA embedding per cell
- Intermediate Parquet artifact readable by downstream Python-side UMAP/t-SNE and clustering steps

## How to apply

Load the normalized wide-format matrix (from the pipeline's Global/Simple CPM-like or Quantile normalization step) as a Spark DataFrame on an Apache Spark 3.0+ environment, standalone or distributed depending on data load. Convert the gene-expression features into the vectorized representation required by Spark's MLlib PCA transformer. Run the scSPARKL PCA routine, which the paper states is 'exclusively spark based' (no Python fallback for this step, unlike UMAP/t-SNE). Write the reduced cell embeddings out as a Parquet artifact (pca_reduced.parquet) so downstream UMAP/t-SNE and clustering steps — which run outside Spark and read directly from intermediate Parquet files — can consume them. Note the paper reports no quantitative acceptance threshold for this step; validation is completion and output format, so benchmark against Seurat/Scanpy manually if biological concordance matters.

## Related tools

- **scSPARKL** (Provides the PCA dimension-reduction routine executed over the vectorized normalized matrix; script file or Jupyter Notebook from the GitHub repo) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Runtime engine; the PCA is computed with Spark MLlib across nodes (standalone or distributed mode depending on data load))
- **Apache Parquet** (Serialization format for the pca_reduced.parquet artifact consumed by downstream UMAP/t-SNE and clustering steps)

## Evaluation signals

- The scSPARKL PCA script runs to completion on the Spark environment without error
- pca_reduced.parquet is emitted in Parquet format (readable with any Parquet reader)
- The output contains exactly one reduced-dimension embedding per cell (row count equals input cell count after filtering)
- Downstream UMAP/t-SNE steps can load their input directly from the written Parquet file
- Full pipeline (QC, filtering, normalization, dimension reduction, clustering, visualization) timing is consistent with the paper's ~8 min reference on PBMC (n = 10000) scale data

## Limitations

- The paper reports no quantitative acceptance threshold for the PCA step; validation is completion and output format only, and no benchmarking against Seurat/Scanpy was performed
- Repository is not actively maintained ('A NEW VERSION IS UNDER DEVELOPMENT'), tested only on Microsoft Windows 11, and cluster mode operations are not yet optimized (each node has separate memory allocation)
- RAM limits during data reshaping/melting are mitigated by data melting techniques that need further optimization; GPU acceleration is not utilized in the current version

## Evidence

- [full_text] PCA as the distributed dimensionality reduction method: [evidence span withheld — anchor status: not_found]
