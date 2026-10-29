---
name: hvg-selection-median-absolute-deviation
description: Use when a scRNA-seq matrix (e.g. melted .csv or .parquet count data)
  has already been normalized (CPM-like or quantile) and you need a robust, Spark-parallel
  HVG/gene-selection step downstream of cell/gene quality filtering.
when_to_use_negative:
- The input counts have not yet been normalized (MAD is computed on normalized count
  values, so run scSPARKL's CPM-like Global or Quantile Normalization first)
- You specifically need a fixed-size gene set (e.g. top 2000 genes) — scSPARKL's Coefficient
  of Variance Squared method returning the top n genes is the appropriate alternative
- Data are small enough for in-memory tools like Seurat or Scanpy, or the analysis
  requires GPU acceleration or optimized Spark cluster mode, which the paper states
  are not yet available
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Provides the MAD-based gene-selection routine with default threshold k > 3,
    executed over Spark DataFrames built from melted, normalized count data
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed execution engine on which the per-gene median and MAD statistics
    are computed in parallel across partitions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:5a47e9669a0c19baa785a46d95258b57d12593b35b92a5e12cde0f837c97aebe
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/asif7adil/scSparkl
  tools:
  - Apache Spark
  - scSPARKL
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# hvg-selection-median-absolute-deviation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects highly variable genes (HVGs) in the scSPARKL pipeline by scoring each gene with the Median Absolute Deviation (MAD) of its normalized count values and retaining genes passing the default robust threshold k > 3, computed in parallel on Apache Spark.

## When to use

Use when a scRNA-seq matrix (e.g. melted .csv or .parquet count data) has already been normalized (CPM-like or quantile) and you need a robust, Spark-parallel HVG/gene-selection step downstream of cell/gene quality filtering.

## When NOT to use

- The input counts have not yet been normalized (MAD is computed on normalized count values, so run scSPARKL's CPM-like Global or Quantile Normalization first)
- You specifically need a fixed-size gene set (e.g. top 2000 genes) — scSPARKL's Coefficient of Variance Squared method returning the top n genes is the appropriate alternative
- Data are small enough for in-memory tools like Seurat or Scanpy, or the analysis requires GPU acceleration or optimized Spark cluster mode, which the paper states are not yet available

## Inputs

- Stage-2 gene quality summaries as a Spark DataFrame (from the automatically generated analyses folder)
- Normalized count values in tall format (.csv or .parquet) from scSPARKL's Global/CPM-like or Quantile normalization
- MAD threshold parameter k (default k > 3)

## Outputs

- hvg_mad_selected_genes.csv containing retained highly variable gene identifiers and their MAD scores

## How to apply

Load the Stage-2 gene quality summaries (written automatically to the analyses folder after data melting and `data_filter()`) as a distributed Spark DataFrame with scSPARKL. Compute, in parallel across Spark partitions, each gene's median normalized expression and its Median Absolute Deviation (MAD), which is robust to outliers compared with variance-based dispersion. Score every gene with this MAD statistic and flag genes exceeding the default threshold k > 3; this selection follows scSPARKL's Stage-2 quality-control stage and consumes the cell/gene quality summaries produced earlier. Filter the gene table to keep only genes whose MAD score passes k > 3, then write the retained genes with their MAD scores to hvg_mad_selected_genes.csv. scSPARKL offers an alternative selection route (Coefficient of Variance Squared returning the top n genes) — choose MAD when you want a robust threshold-based rule rather than a fixed top-n ranking.

## Related tools

- **scSPARKL** (Provides the MAD-based gene-selection routine with default threshold k > 3, executed over Spark DataFrames built from melted, normalized count data) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed execution engine on which the per-gene median and MAD statistics are computed in parallel across partitions)

## Evaluation signals

- hvg_mad_selected_genes.csv exists and parses as valid CSV
- Output contains gene identifier and MAD-score fields for every retained gene
- Every retained gene satisfies the k > 3 MAD criterion — recompute per-gene MAD from the normalized counts and confirm no selected gene is below threshold
- Selected gene set is a strict subset of genes surviving the Stage-2 data_filter() quality filtering (percent_ercc > 10%, percent_mito > 5%, cells < 10 genes, genes < 3 cells, dropout > 70%)
- Compare the MAD-selected set against the alternative CV^2 top-n selection to sanity-check that known marker/variable genes agree across methods

## Limitations

- The MAD threshold k > 3 is scSPARKL's default; the README notes default parameters must be changed by directly manipulating the filter package
- scSPARKL currently accepts only .csv and .parquet inputs, and the repository is not actively maintained (a new version is under development)
- The paper reports no benchmarking against Seurat or Scanpy, so HVG concordance with those frameworks is unverified; GPU acceleration and optimized Spark cluster mode are also not yet supported
- Data melting for very large matrices is a RAM-limitation workaround that the authors note needs further optimization

## Evidence
