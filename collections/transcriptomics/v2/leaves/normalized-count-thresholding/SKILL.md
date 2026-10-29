---
name: normalized-count-thresholding
description: Use when selecting highly variable genes from normalized scRNA-seq count
  data at scale — e.g. after scSPARKL Global/CPM or Quantile normalization — where
  a robust MAD threshold (default k > 3) should decide gene retention.
when_to_use_negative:
- The goal is to return a fixed number of top-n genes by variance — scSPARKL's alternative
  Coefficient of Variance Squared method (returning Top n genes) is the intended choice.
- Input counts have not been normalized yet (scSPARKL normalization precedes gene
  selection) — applying a MAD threshold on raw counts is out of scope.
- A small dataset fits in memory and standard in-memory tools such as Seurat or Scanpy
  are adequate — scSPARKL targets datasets of hundreds of thousands to millions of
  cells.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scSPARKL
  role: Framework implementing the MAD-based HVG/gene selection step with default
    threshold k > 3 over tall-format normalized scRNA-seq data
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed computation of per-gene median and MAD statistics in parallel
    across Spark partitions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:9da0357ad341056fc683510893fdc699ff33178c3bed3df09892e01ae846297c
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# normalized-count-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A MAD-based highly variable gene selection step from the scSPARKL pipeline: per-gene Median Absolute Deviation of normalized count values is computed in parallel on Apache Spark, and genes with MAD score k > 3 are retained as HVGs.

## When to use

Use when selecting highly variable genes from normalized scRNA-seq count data at scale — e.g. after scSPARKL Global/CPM or Quantile normalization — where a robust MAD threshold (default k > 3) should decide gene retention.

## When NOT to use

- The goal is to return a fixed number of top-n genes by variance — scSPARKL's alternative Coefficient of Variance Squared method (returning Top n genes) is the intended choice.
- Input counts have not been normalized yet (scSPARKL normalization precedes gene selection) — applying a MAD threshold on raw counts is out of scope.
- A small dataset fits in memory and standard in-memory tools such as Seurat or Scanpy are adequate — scSPARKL targets datasets of hundreds of thousands to millions of cells.

## Inputs

- Gene quality summaries (CSV) from scSPARKL's analyses folder, loaded as an Apache Spark DataFrame
- Normalized count values in tall format (produced by scSPARKL Global/CPM or Quantile normalization)
- MAD threshold parameter k (default k > 3)

## Outputs

- hvg_mad_selected_genes.csv — CSV of retained highly variable genes with gene identifier and MAD-score fields
- Filtered gene table (low-MAD genes dropped)

## How to apply

Load the Stage-2 gene quality summaries that scSPARKL wrote to the automatically generated analyses folder as a Spark DataFrame. Compute per-gene median expression and the Median Absolute Deviation (MAD) of the normalized count values in parallel across Spark partitions, then score each gene with this robust statistic — MAD is used because it is outlier-resistant compared to plain variance. Apply the default MAD threshold k > 3 to flag highly variable genes (the README lists MAD with 'Default Threshold k > 3' as one of two selection methods, alongside Coefficient of Variance Squared returning top n genes). Retain only genes whose MAD score passes the threshold and write the retained gene list with their MAD scores to hvg_mad_selected_genes.csv. This selection consumes the quality summaries passed through the pipeline's `data_filter()` mechanism, whose default parameters can be changed by directly manipulating the filter package.

## Related tools

- **scSPARKL** (Framework implementing the MAD-based HVG/gene selection step with default threshold k > 3 over tall-format normalized scRNA-seq data) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed computation of per-gene median and MAD statistics in parallel across Spark partitions)

## Evaluation signals

- hvg_mad_selected_genes.csv exists and parses as valid CSV
- Output file contains both the gene identifier field and a MAD-score field
- Every retained gene satisfies the MAD criterion k > 3
- The gene quality summaries in the analyses folder were produced and consumed by data_filter() before selection ran
- Retained gene set is a strict subset of the input gene table (genes failing the threshold are dropped, not kept)

## Limitations

- The scSPARKL repository is not being actively maintained; a new version is under development, so default parameters may change.
- The current implementation has been tested on Microsoft Windows 11 and accepts only .csv and .parquet inputs.
- Default parameters can only be changed by directly manipulating the filter package; no exposed CLI configuration is documented.
- Spark cluster mode operations are not yet optimized (deferred to future development), and no benchmarking against Seurat and Scanpy has been performed.

## Evidence
