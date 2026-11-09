---
name: quantile-normalization-of-count-matrices
description: Use when tall-format filtered scRNA-seq count data must have per-cell
  expression distributions standardized across cells (e.g., to remove library-composition/distribution
  shifts) while preserving within-cell relative expression rank order, at scales requiring
  Spark.
when_to_use_negative:
- Input is still raw/unfiltered wide-format data — it must first be melted to tall
  format and passed through scSPARKL quality-summary generation and data_filter()
  before normalization
- The analysis requires CPM-style depth scaling only (Global/Simple normalization)
  or downstream HVG selection via MAD (k > 3) or Coefficient of Variance Squared,
  where quantile-forced identical distributions would be inappropriate
- Dataset is small enough for in-memory tools like Seurat or Scanpy — the article
  notes these rely on in-memory data structures, but scSPARKL is motivated by hundreds
  of thousands to millions of cells
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Modular Spark-based framework whose data_normalize.py module implements the
    Quantile Normalization routine on tall-format data
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed in-memory engine executing the rank/average/back-assign computation
    in parallel across cells
- name: Apache Parquet
  role: Columnar storage format used to write normalized_wide.parquet and normalized_tall.parquet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:70e43689379ccf3ce10d6a2f5b9dff331a79e12ae8179febc287804ee4ddc35d
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
yes_rate: 0.75
n_claims: 12
---

# Quantile normalization of count matrices

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Implements the scSPARKL data_normalize.py Quantile Normalization routine, which ranks each cell's expression values, averages values at the same rank across all cells to build a reference distribution, and back-assigns those averages by original rank so every cell shares one empirical distribution — executed distributed on Apache Spark with Parquet output.

## When to use

Use when tall-format filtered scRNA-seq count data must have per-cell expression distributions standardized across cells (e.g., to remove library-composition/distribution shifts) while preserving within-cell relative expression rank order, at scales requiring Spark.

## When NOT to use

- Input is still raw/unfiltered wide-format data — it must first be melted to tall format and passed through scSPARKL quality-summary generation and data_filter() before normalization
- The analysis requires CPM-style depth scaling only (Global/Simple normalization) or downstream HVG selection via MAD (k > 3) or Coefficient of Variance Squared, where quantile-forced identical distributions would be inappropriate
- Dataset is small enough for in-memory tools like Seurat or Scanpy — the article notes these rely on in-memory data structures, but scSPARKL is motivated by hundreds of thousands to millions of cells

## Inputs

- Filtered tall-format scRNA-seq count matrix (.csv or .parquet) loaded as an Apache Spark DataFrame
- Filtered tall-format count matrix produced upstream by scSPARKL data_filter()

## Outputs

- normalized_wide.parquet — normalized expression matrix in cells x genes wide format
- normalized_tall.parquet — normalized expression matrix in tall format

## How to apply

Load the filtered tall-format count matrix (.csv or .parquet) into a Spark DataFrame per the scSPARKL input convention. Rank each cell's expression values from lowest to highest, average the values at each rank position across all cells to form a reference distribution, then map these averaged values back to each cell according to its original rank order. Note scSPARKL's data_normalize.py offers two normalization types — Global/Simple normalization (analogous to CPM: scale each cell to a 10,000-count target by library size plus log1p with pseudocount 1) and Quantile Normalization — so pick Quantile when you need identical empirical distributions across cells rather than CPM-style depth correction. Reshape each normalized result into one wide-format matrix (cells x genes) and one tall-format matrix, since normalization takes tall data and returns both. Write the outputs as normalized_wide.parquet and normalized_tall.parquet using the Apache Parquet writer on Spark.

## Related tools

- **scSPARKL** (Modular Spark-based framework whose data_normalize.py module implements the Quantile Normalization routine on tall-format data) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed in-memory engine executing the rank/average/back-assign computation in parallel across cells)
- **Apache Parquet** (Columnar storage format used to write normalized_wide.parquet and normalized_tall.parquet)

## Examples

```
norm_wide = spark.read.parquet('normalized_wide.parquet'); norm_tall = spark.read.parquet('normalized_tall.parquet')
```

## Evaluation signals

- Both output files exist and load correctly as Apache Parquet
- Quantile-normalized cells have identical empirical distributions — sorted value vectors match across all cells
- Within-cell rank order is preserved (relative expression relationships unchanged) after back-assignment
- The companion Global/Simple normalization in the same run matches CPM scaling to a 10,000-count target followed by log1p with pseudocount 1
- Wide and tall normalized outputs contain consistent values and cover the same cells/genes as the filtered input

## Limitations

- The scSPARKL repository is not actively maintained; a new version is under development (per README)
- Pipeline inputs are currently limited to .csv and .parquet formats
- Cluster-mode Spark operations are not yet optimized, and no benchmarking against Seurat/Scanpy has been performed, so scaling behavior beyond the tested configurations is unverified

## Evidence
