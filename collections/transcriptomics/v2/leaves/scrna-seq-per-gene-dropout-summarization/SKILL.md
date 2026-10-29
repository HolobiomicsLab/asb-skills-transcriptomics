---
name: scrna-seq-per-gene-dropout-summarization
description: Use when you need per-gene quality metrics (cells expressing > 0, dropout
  count/percentage, mean/total expression) from tall-format scRNA-seq data to drive
  gene-level filtering in a Spark-based pipeline.
when_to_use_negative:
- Input is already a wide gene-by-cell matrix that has not been melted to tall format;
  the skill expects the tall/melted representation produced by the scSPARKL data-melting
  step
- You need per-cell quality metrics (ERCC percentage, mitochondrial percentage, total
  genes expressed) — use the Θ1 per-cell QC summary instead
- You are running an in-memory pipeline such as Seurat or Scanpy on a small dataset;
  the Spark-distributed approach is intended for large-scale scRNA-seq data
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: scSPARKL
  role: Framework whose Stage-2 quality control module performs the per-gene (Θ2)
    quality summarization via group-by aggregation on the melted data
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed computation engine used to group by gene and aggregate the Θ2
    QC metrics as a distributed DataFrame
- name: Apache Parquet
  role: Columnar storage format used to write the per-gene quality summary table for
    downstream consumption by the filtering step
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:8155060ef467d6dc1e6759425ad006ab30269934d5615bf00f952089796b93ce
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# scrna-seq-per-gene-dropout-summarization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute the Θ2 per-gene quality summary in scSPARKL by grouping the melted tall-format Spark DataFrame by gene and aggregating the number of cells expressing the gene, total expression, dropout count, dropout percentage, and mean expression, writing the result as Parquet for downstream gene filtering.

## When to use

Use when you need per-gene quality metrics (cells expressing > 0, dropout count/percentage, mean/total expression) from tall-format scRNA-seq data to drive gene-level filtering in a Spark-based pipeline.

## When NOT to use

- Input is already a wide gene-by-cell matrix that has not been melted to tall format; the skill expects the tall/melted representation produced by the scSPARKL data-melting step
- You need per-cell quality metrics (ERCC percentage, mitochondrial percentage, total genes expressed) — use the Θ1 per-cell QC summary instead
- You are running an in-memory pipeline such as Seurat or Scanpy on a small dataset; the Spark-distributed approach is intended for large-scale scRNA-seq data

## Inputs

- Cleaned, melted tall-format scRNA-seq expression table (CSV or Parquet) as a Spark distributed DataFrame

## Outputs

- Θ2 per-gene quality summary DataFrame (cells expressing gene > 0, total expression, dropout count, dropout percentage, mean expression, log1p-transformed count columns)
- Parquet file of gene quality summaries written to the automatically generated analyses folder

## How to apply

Load the cleaned, melted tall-format expression data (CSV or Parquet) as a Spark distributed DataFrame in scSPARKL, then group by gene and aggregate the Θ2 metrics: number of cells expressing the gene greater than zero, total expression, dropout count, dropout percentage, and mean expression, following scater/SCANPY/Seurat conventions. Apply a log1p transformation to the relevant expression-count columns of the gene summary table. Write the gene quality DataFrame as a Parquet file into the automatically generated analyses folder alongside the Θ1 per-cell summary. The summaries are then passed as arguments to `data_filter()`, where defaults such as genes expressed in < 3 cells and dropout rate > 70% can be changed by manipulating the filter package. Validate that the Parquet QC file exists in the analyses folder and contains the required summary columns.

## Related tools

- **scSPARKL** (Framework whose Stage-2 quality control module performs the per-gene (Θ2) quality summarization via group-by aggregation on the melted data) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed computation engine used to group by gene and aggregate the Θ2 QC metrics as a distributed DataFrame)
- **Apache Parquet** (Columnar storage format used to write the per-gene quality summary table for downstream consumption by the filtering step)

## Examples

```
spark-submit scsparkl.py --task quality_summaries --input melted_tall.parquet --qc-level gene --out analyses/
```

## Evaluation signals

- The gene QC Parquet file exists in the automatically generated analyses folder
- The gene summary table contains the required columns: number of cells expressing the gene, total expression, dropout count, dropout percentage, and mean expression
- Relevant expression-count columns have been log1p-transformed in the summary table
- Per-gene metrics are consistent with per-cell Θ1 metrics (e.g. summing 'cells expressing gene' columns reconciles with per-cell 'number of genes expressed')
- The summary feeds valid thresholds into data_filter(), e.g. genes expressed in < 3 cells and dropout rate > 70%

## Limitations

- The repository README warns the repo isn't being actively maintained and a new version is under development
- Pipeline currently only accepts .csv and .parquet input formats
- Data melting workaround for RAM limitation needs further optimization per the discussion section
- Cluster mode operations are not yet optimized; deferred to future development
- Current implementation has been tested on Microsoft Windows 11

## Evidence

- [other] Group by gene and aggregate the Θ2 per-gene QC metrics: [evidence span withheld — anchor status: not_found]
