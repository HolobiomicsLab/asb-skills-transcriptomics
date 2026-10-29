---
name: single-cell-rna-seq-quality-control-filtering
description: Use when raw single-cell RNA-seq count matrices (.csv or .parquet) need
  QC summarization and low-quality cell/gene removal as a preprocessing step before
  normalization, HVG selection, and clustering.
when_to_use_negative:
- Input is already normalized or filtered data where re-filtering would drop valid
  cells/genes (data_filter() expects raw count-derived quality summaries)
- Data are not single-cell transcriptomic counts — the cutoffs (genes per cell, dropout
  rate, percent_ercc/percent_mito) are meaningless for bulk RNA-seq or other data
  types
- Input files are formats other than .csv or .parquet; the current scSPARKL pipeline
  only accepts those two formats
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Provides the Spark-parallel quality-summary routines and the `data_filter()`
    function used to remove unwanted cells and genes
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed engine on which the parallel QC and filtering routines execute,
    avoiding in-memory RAM constraints
- name: Apache Parquet
  role: Storage format accepted for input count matrices feeding the QC/filtering
    stage
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:a6a889aea47574cb005f2350369c3488ef017c86118b96b4a1c62328ded8ae9a
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
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-rna-seq-quality-control-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quality control and cell/gene filtering stage of the scSPARKL pipeline: Spark-parallel routines generate per-cell and per-gene quality summaries from melted tall-format scRNA-seq count data, then apply the `data_filter()` function to remove unwanted cells and genes using configurable cutoffs before normalization.

## When to use

Use when raw single-cell RNA-seq count matrices (.csv or .parquet) need QC summarization and low-quality cell/gene removal as a preprocessing step before normalization, HVG selection, and clustering.

## When NOT to use

- Input is already normalized or filtered data where re-filtering would drop valid cells/genes (data_filter() expects raw count-derived quality summaries)
- Data are not single-cell transcriptomic counts — the cutoffs (genes per cell, dropout rate, percent_ercc/percent_mito) are meaningless for bulk RNA-seq or other data types
- Input files are formats other than .csv or .parquet; the current scSPARKL pipeline only accepts those two formats

## Inputs

- Raw scRNA-seq count matrix in .csv or .parquet format (wide, genes x cells)
- Melted tall-format count table produced by the data melting stage
- Cell and gene quality summaries (percent_ercc, percent_mito, gene counts per cell, cells per gene, dropout rate) written to the auto-generated analyses folder

## Outputs

- Filtered tall-format count table with unwanted cells and genes removed
- Filtered cell and gene quality summaries retained for downstream `data_filter()` reuse and auditing

## How to apply

First clean and melt the input count matrix to tall format, since scSPARKL's QC routines operate on the tall representation. Run the Spark-parallel quality summary routines for genes and cells, which write outputs to an analyses folder that is automatically generated. Pass those quality summaries as arguments to `data_filter()`, applying default cutoffs such as cells expressing < 10 genes, genes expressed in < 3 cells, dropout rate > 70%, percent_ercc > 10%, and percent_mito > 5%; defaults can be changed by directly manipulating the filter package, and new columns can be added for additional filtering operations. The rationale is to drop low-quality cells and low-information genes before downstream stages (normalization, PCA, clustering), as these cutoffs were used in the paper's reanalysis of real-world datasets such as the Jurkat-293T mixture.

## Related tools

- **scSPARKL** (Provides the Spark-parallel quality-summary routines and the `data_filter()` function used to remove unwanted cells and genes) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed engine on which the parallel QC and filtering routines execute, avoiding in-memory RAM constraints)
- **Apache Parquet** (Storage format accepted for input count matrices feeding the QC/filtering stage)

## Evaluation signals

- Filtered cell/gene quality summaries exist in the auto-generated analyses folder and are consistent with the raw summaries minus removed entities
- No cells remain that express fewer than 10 genes, and no genes remain expressed in fewer than 3 cells
- All retained cells/genes satisfy the paper's cutoffs: percent_ercc <= 10%, percent_mito <= 5%, dropout rate <= 70%
- Downstream stages succeed: normalization and PCA on the filtered table complete and clustering of the Jurkat-293T mixture reaches ARI ~0.90 against the expected species partition
- Filter package parameters match intended defaults or documented overrides (defaults changeable by directly manipulating the filter package)

## Limitations

- Cluster-mode Spark operation is not yet optimized, which constrains very large distributed filtering runs
- The data-melting step that precedes quality summaries is a workaround for RAM limitations and needs further optimization
- Default filter parameters must be changed by directly manipulating the filter package; no higher-level parameter interface is documented

## Evidence
