---
name: log1p-expression-summary-transformation
description: Use when scRNA-seq count data (CSV or Parquet, melted tall format) has
  been reshaped in Spark and per-cell/per-gene QC summaries (total expression, genes
  expressed, percent ERCC, percent mitochondrial, dropout) need variance stabilization
  before filtering.
when_to_use_negative:
- Input is already a log-transformed or normalized expression matrix — applying log1p
  again would double-transform the values
- Data has not been melted to tall format and QC summaries have not yet been aggregated
  per cell and per gene
- The goal is gene selection or PCA on the full expression matrix rather than QC summary
  transformation prior to data_filter()
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Provides the Stage-2 QC module whose per-cell (Θ1) and per-gene (Θ2) quality
    summary tables receive the log1p transformation before Parquet export
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed engine on which the tall-format DataFrame is grouped/aggregated
    and the log1p transformation is applied in parallel
- name: Apache Parquet
  role: Storage format for the two log1p-transformed quality summary dataframes written
    to the analyses folder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:9ed8a717d88b06af8fc927824778dc219339cbf2eb15c7227219b8b28e03ac61
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# log1p-expression-summary-transformation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies a log1p transformation to the expression-count columns of the per-cell (Θ1) and per-gene (Θ2) quality summary tables computed by scSPARKL's Stage-2 QC module on Apache Spark, stabilizing skewed count distributions before downstream filtering.

## When to use

Use when scRNA-seq count data (CSV or Parquet, melted tall format) has been reshaped in Spark and per-cell/per-gene QC summaries (total expression, genes expressed, percent ERCC, percent mitochondrial, dropout) need variance stabilization before filtering.

## When NOT to use

- Input is already a log-transformed or normalized expression matrix — applying log1p again would double-transform the values
- Data has not been melted to tall format and QC summaries have not yet been aggregated per cell and per gene
- The goal is gene selection or PCA on the full expression matrix rather than QC summary transformation prior to data_filter()

## Inputs

- Cleaned, melted tall-format scRNA-seq expression data as an Apache Spark DataFrame (.csv or .parquet input)
- Θ1 per-cell quality summary columns: total expression, number of genes expressed, average expression, percent ERCC, percent mitochondrial
- Θ2 per-gene quality summary columns: cells expressing the gene, total expression, dropout count, dropout percentage, mean expression

## Outputs

- Θ1 per-cell QC summary DataFrame (Parquet) with log1p-transformed expression-count columns
- Θ2 per-gene QC summary DataFrame (Parquet) with log1p-transformed expression-count columns, written to the analyses folder

## How to apply

After loading the melted tall-format expression data as a Spark distributed DataFrame, aggregate the Θ1 per-cell metrics (total expression, number of genes expressed, average expression, percentage of ERCC and mitochondrial reads) and the Θ2 per-gene metrics (cells expressing the gene, total expression, dropout count, dropout percentage, mean expression). Apply log1p to the relevant raw expression-count columns of both summary tables — the log1p form keeps zero-expression entries valid (log1p(0)=0) and matches scater/SCANPY/Seurat QC conventions. Write the two transformed quality dataframes as separate Parquet files into the automatically generated analyses folder. Validation requires that both Parquet QC files exist in the analyses folder and contain the summary columns (including percent ERCC and percent mitochondrial) consumed by downstream thresholds such as percent_ercc > 10% and percent_mito > 5%; the quality summaries are then passed to data_filter().

## Related tools

- **scSPARKL** (Provides the Stage-2 QC module whose per-cell (Θ1) and per-gene (Θ2) quality summary tables receive the log1p transformation before Parquet export) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed engine on which the tall-format DataFrame is grouped/aggregated and the log1p transformation is applied in parallel)
- **Apache Parquet** (Storage format for the two log1p-transformed quality summary dataframes written to the analyses folder)

## Examples

```
qc = qc.withColumn('log1p_total_expression', F.log1p('total_expression'))  # apply per-cell and per-gene before writing Parquet QC files
```

## Evaluation signals

- Both Parquet QC files (Θ1 cell and Θ2 gene summaries) exist in the automatically generated analyses folder
- QC files contain the required summary columns including percent_ercc and percent_mito used by downstream thresholds (percent_ercc > 10%, percent_mito > 5%)
- log1p-transformed count columns contain no negative values and preserve zeros (log1p(0)=0), so zero-expression entries are valid
- Transformed summaries feed cleanly into data_filter() — e.g. filtering cells expressing < 10 genes and genes expressed in < 3 cells with dropout rate > 70% executes without missing-column errors
- Per-cell metrics (genes expressed, total expression, percent ERCC/mito) and per-gene metrics (cells expressing, dropout percentage) match raw-count recomputation after log1p inversion

## Limitations

- The repository is not actively maintained and a new version is under development, so QC behavior may change
- The pipeline currently accepts only .csv and .parquet inputs; other formats are not supported
- Data melting is a workaround for RAM limitations and 'further optimization is needed' per the article discussion
- Filter defaults (e.g. percent_ercc > 10%, percent_mito > 5%) can only be changed by directly manipulating the filter package

## Evidence
