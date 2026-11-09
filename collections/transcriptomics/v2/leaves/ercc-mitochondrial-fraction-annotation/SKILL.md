---
name: ercc-mitochondrial-fraction-annotation
description: Use when tall-format melted scRNA-seq expression data (CSV or Parquet)
  has been loaded in Spark and per-cell/per-gene QC metrics — percent ERCC, percent
  mitochondrial, counts, and dropouts — are needed before filtering.
when_to_use_negative:
- Input is raw uncounted sequencing output (e.g. FASTQ or Illumina HiSeq raw data)
  rather than a melted count matrix — the pipeline expects .csv or .parquet input
- Data is already in wide matrix form and has not been melted to tall format; data
  melting must precede quality summary generation
- The analysis is not scRNA-seq downstream processing (e.g. multi-omics CITE-seq/scATAC-seq)
  — such Spark-based pipelines are not yet developed
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scSPARKL
  role: Provides the Stage-2 quality-control routine that groups the melted Spark
    DataFrame by cell and gene to build the Θ1/Θ2 quality summary matrices and writes
    them to the analyses folder
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed execution engine on which the group-by aggregations over the tall-format
    expression data run
- name: Apache Parquet
  role: Columnar format used to write the two per-cell and per-gene quality summary
    files for downstream filtering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:a37582d252a89039b33159bb74d99045803c47628634452e1505c02c4a8d3842
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
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# ercc-mitochondrial-fraction-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In scSPARKL's Stage-2 quality control, grouped aggregations over the melted tall-format expression DataFrame derive per-cell (Θ1) and per-gene (Θ2) quality summary matrices, including percent ERCC and percent mitochondrial reads per cell, saved as Parquet files for downstream `data_filter()` thresholding.

## When to use

Use when tall-format melted scRNA-seq expression data (CSV or Parquet) has been loaded in Spark and per-cell/per-gene QC metrics — percent ERCC, percent mitochondrial, counts, and dropouts — are needed before filtering.

## When NOT to use

- Input is raw uncounted sequencing output (e.g. FASTQ or Illumina HiSeq raw data) rather than a melted count matrix — the pipeline expects .csv or .parquet input
- Data is already in wide matrix form and has not been melted to tall format; data melting must precede quality summary generation
- The analysis is not scRNA-seq downstream processing (e.g. multi-omics CITE-seq/scATAC-seq) — such Spark-based pipelines are not yet developed

## Inputs

- Cleaned, melted tall-format scRNA-seq expression data (CSV or Parquet) loaded as an Apache Spark distributed DataFrame
- Cell and gene identifiers distinguishable in the tall format (e.g. mouse vs human cells in a Jurkat–293T mixture)
- Optional: ERCC spike-in and mitochondrial gene annotations used to compute percent_ercc and percent_mito columns

## Outputs

- Θ1 per-cell quality summary table (total expression, genes expressed, average expression, percent ERCC, percent mitochondrial)
- Θ2 per-gene quality summary table (cells expressing gene, total expression, dropout count, dropout percentage, mean expression)
- Two Parquet QC files written into an automatically generated analyses folder for downstream `data_filter()` consumption

## How to apply

Load the cleaned, melted tall-format expression data as a Spark distributed DataFrame, then group by cell to aggregate Θ1 metrics: total expression, number of genes expressed, average expression, and percentage of ERCC and mitochondrial reads, following scater/SCANPY/Seurat conventions. Separately group by gene to aggregate Θ2 metrics: number of cells expressing the gene, total expression, dropout count, dropout percentage, and mean expression. Apply log1p transformation to the relevant expression-count columns of both summary tables, then write each quality dataframe as a separate Parquet file into the automatically generated analyses folder. These summaries are the arguments later passed to `data_filter()` with thresholds such as percent_ercc > 10%, percent_mito > 5%, cells expressing < 10 genes, genes expressed in < 3 cells, and dropout rate > 70%; default parameters can be changed by directly manipulating the filter package.

## Related tools

- **scSPARKL** (Provides the Stage-2 quality-control routine that groups the melted Spark DataFrame by cell and gene to build the Θ1/Θ2 quality summary matrices and writes them to the analyses folder) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed execution engine on which the group-by aggregations over the tall-format expression data run)
- **Apache Parquet** (Columnar format used to write the two per-cell and per-gene quality summary files for downstream filtering)

## Evaluation signals

- Both Parquet QC files exist in the automatically generated analyses folder after the run
- Θ1 file contains the required per-cell columns including percent_ercc and percent_mito
- Θ2 file contains the required per-gene columns including cells-expressing count, dropout count, and dropout percentage
- Relevant expression-count columns in both summary tables carry the log1p transformation
- Downstream `data_filter()` thresholds (percent_ercc > 10%, percent_mito > 5%) evaluate against these summary columns without schema errors

## Limitations

- The pipeline currently only accepts .csv and .parquet input files; other formats are not yet supported
- scSPARKL cluster-mode operations are not yet optimized (each node has separate memory allocation), so Stage-2 aggregations were demonstrated mainly in standalone configurations
- The repository is not being actively maintained; a new version is under development
- Data melting mitigates RAM limitations but needs further optimization, which affects the tall-format input feeding the quality summaries

## Evidence
