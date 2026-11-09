---
name: parquet-serialization-of-quality-tables
description: Use when you have cleaned, melted tall-format scRNA-seq expression data
  (CSV or Parquet) loaded as a Spark DataFrame and need per-cell and per-gene quality
  summaries (ERCC %, mito %, dropouts) persisted as Parquet tables for threshold-based
  filtering.
when_to_use_negative:
- Input is raw count matrices in formats other than .csv or .parquet — the pipeline
  currently only accepts these input formats
- The goal is normalization or dimension reduction (Global/CPM-style, Quantile Normalization,
  MAD-based gene selection, Spark PCA) rather than QC summary generation
- Data is already aggregated into cell/gene quality tables — re-deriving Θ1/Θ2 would
  duplicate the summaries
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scSPARKL
  role: 'Framework that executes the Stage-2 QC: Spark-based group-by aggregations
    for Θ1/Θ2 quality summaries and writes them to the analyses folder'
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed engine on which the per-cell and per-gene QC aggregations run
    as DataFrame operations
- name: Apache Parquet
  role: Columnar storage format used to serialize the two quality summary tables for
    downstream consumption by data_filter()
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:148a9af96edb944ee64bdb369abbb2295ef18ddbb572590eda9778a5d5cff0a5
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# parquet-serialization-of-quality-tables

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scSPARKL Stage-2 quality control derives per-cell (Θ1) and per-gene (Θ2) quality summary matrices via Spark group-by aggregations on melted tall-format scRNA-seq data, applies log1p to expression columns, and serializes both summaries as separate Parquet files in an auto-generated analyses folder for downstream data_filter() usage.

## When to use

Use when you have cleaned, melted tall-format scRNA-seq expression data (CSV or Parquet) loaded as a Spark DataFrame and need per-cell and per-gene quality summaries (ERCC %, mito %, dropouts) persisted as Parquet tables for threshold-based filtering.

## When NOT to use

- Input is raw count matrices in formats other than .csv or .parquet — the pipeline currently only accepts these input formats
- The goal is normalization or dimension reduction (Global/CPM-style, Quantile Normalization, MAD-based gene selection, Spark PCA) rather than QC summary generation
- Data is already aggregated into cell/gene quality tables — re-deriving Θ1/Θ2 would duplicate the summaries

## Inputs

- Cleaned, melted tall-format scRNA-seq expression data as CSV or Apache Parquet, loaded as an Apache Spark distributed DataFrame

## Outputs

- Θ1 per-cell quality summary Parquet file (total expression, genes expressed, average expression, percent ERCC, percent mitochondrial, log1p-transformed counts)
- Θ2 per-gene quality summary Parquet file (cells expressing gene, total expression, dropout count, dropout percentage, mean expression, log1p-transformed counts)
- Both files written into an auto-generated analyses folder

## How to apply

Load the cleaned, melted tall-format expression data (CSV or Parquet) as a Spark distributed DataFrame. Group by cell and aggregate the Θ1 per-cell QC metrics — total expression, number of genes expressed, average expression, and percentage of ERCC and mitochondrial reads, following scater/SCANPY/Seurat conventions. Group by gene and aggregate the Θ2 per-gene metrics — number of cells expressing the gene, total expression, dropout count, dropout percentage, and mean expression. Apply a log1p transformation to the relevant expression-count columns of both summary tables, then write the two quality DataFrames as separate Parquet files into the automatically generated analyses folder. These summaries are consumed by data_filter() with thresholds such as percent_ercc > 10%, percent_mito > 5%, cells expressing < 10 genes, genes expressed in < 3 cells, and dropout rate > 70%.

## Related tools

- **scSPARKL** (Framework that executes the Stage-2 QC: Spark-based group-by aggregations for Θ1/Θ2 quality summaries and writes them to the analyses folder) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed engine on which the per-cell and per-gene QC aggregations run as DataFrame operations)
- **Apache Parquet** (Columnar storage format used to serialize the two quality summary tables for downstream consumption by data_filter())

## Examples

```
qc_cells.write.parquet("analyses/quality_summary_cells.parquet"); qc_genes.write.parquet("analyses/quality_summary_genes.parquet")
```

## Evaluation signals

- Both Parquet QC files exist in the auto-generated analyses folder after the step completes
- Θ1 file contains the required columns including percent_ercc and percent_mito, usable by downstream thresholds such as percent_ercc > 10% and percent_mito > 5%
- Θ2 file contains the required columns including dropout count and dropout percentage, usable by the dropout rate > 70% filter
- log1p transformation has been applied to the relevant expression-count columns of both summary tables
- Row counts of the Θ1 and Θ2 tables match the number of distinct cells and genes in the melted input, respectively

## Limitations

- The repository is not actively maintained; a new version is under development
- The current implementation has been tested on Microsoft Windows 11 only, and Spark cluster mode operations are not yet optimized
- Only .csv and .parquet input formats are supported; additional formats are still being worked on
- Default filter parameters must be changed by directly manipulating the filter package; adding new columns for other filtering operations requires manual extension

## Evidence
