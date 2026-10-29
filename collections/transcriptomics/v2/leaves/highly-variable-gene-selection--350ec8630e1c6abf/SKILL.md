---
name: highly-variable-gene-selection
description: Use when a Spark-based scRNA-seq pipeline has produced normalized (CPM-like
  or quantile) tall-format counts and you need to restrict downstream PCA to a informative
  gene subset before clustering.
when_to_use_negative:
- Input is raw (unnormalized) counts — gene variance statistics should be computed
  on scSPARKL's normalized tall-format output, not the melted raw matrix
- The analysis goal requires all genes (e.g. marker-exhaustive screens) rather than
  a variance-filtered subset prior to PCA
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scSPARKL
  role: Provides the Spark-native gene-selection module (MAD k>3 or Coefficient of
    Variance Squared top n) that runs on normalized tall-format data
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Execution engine enabling parallel, memory-efficient HVG selection on large
    scRNA-seq matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:08fa4e7c8598ae56a15ae9c8025105a03b4da0a98598045da8f35f64d3139079
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# highly-variable-gene-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Select highly variable genes (HVGs) from normalized tall-format scRNA-seq counts using scSPARKL's Spark-native gene-selection module, prior to dimensionality reduction. Two methods are supported: Median Absolute Deviation (MAD) with default threshold k > 3, or Coefficient of Variance Squared returning the top n genes.

## When to use

Use when a Spark-based scRNA-seq pipeline has produced normalized (CPM-like or quantile) tall-format counts and you need to restrict downstream PCA to a informative gene subset before clustering.

## When NOT to use

- Input is raw (unnormalized) counts — gene variance statistics should be computed on scSPARKL's normalized tall-format output, not the melted raw matrix
- The analysis goal requires all genes (e.g. marker-exhaustive screens) rather than a variance-filtered subset prior to PCA

## Inputs

- Normalized tall-format scRNA-seq count table from scSPARKL (CSV or Parquet), output of Global/Simple or Quantile Normalization
- Selection parameters: MAD threshold k (default k > 3) or top n for Coefficient of Variance Squared

## Outputs

- Selected highly-variable-gene set (top n or MAD k>3 genes) used as input to the Spark-native PCA stage

## How to apply

After scSPARKL's normalization stage (Global/Simple CPM-like or Quantile Normalization) yields wide- and tall-format normalized outputs, run the gene-selection module on the normalized tall data. Choose Median Absolute Deviation (MAD) with the default threshold k > 3, or Coefficient of Variance Squared to return the top n genes; defaults can be changed by directly manipulating the filter/selection package parameters. The selected gene subset is then fed into scSPARKL's exclusively Spark-based PCA implementation, reducing dimensionality for clustering (e.g. the Jurkat–293T mixture where CD3D and XIST separate mouse and human cells). Rationale: variance-based selection concentrates the analysis on genes driving biological structure while dropping flat genes, and keeping the operation in Spark preserves scalability on datasets of hundreds of thousands of cells.

## Related tools

- **scSPARKL** (Provides the Spark-native gene-selection module (MAD k>3 or Coefficient of Variance Squared top n) that runs on normalized tall-format data) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Execution engine enabling parallel, memory-efficient HVG selection on large scRNA-seq matrices)

## Evaluation signals

- Selected gene count matches the chosen parameter: either the requested top n genes (Coefficient of Variance Squared) or the MAD-selected set with default k > 3
- The selected set is a subset of the genes that survived data_filter() (genes expressed in < 3 cells and dropout rate > 70% already removed)
- Downstream Spark PCA runs on the reduced gene matrix without type/format errors in the tall-format input
- On the Jurkat–293T benchmark, the pipeline downstream of HVG selection still resolves CD3D (mouse) and XIST (human) separation with ARI of approximately 0.90

## Limitations

- Only two selection methods are implemented (MAD and Coefficient of Variance Squared); thresholds like the default k > 3 must be manually changed by manipulating the filter package parameters
- The repository is not actively maintained; a new version is under development, and cluster mode operations are not yet optimized
- No benchmarking against Seurat and Scanpy gene-selection strategies has been performed, so HVG choices are not cross-validated against standard in-memory tools

## Evidence
