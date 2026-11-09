---
name: cell-clustering-validation-with-marker-genes
description: Use when a scRNA-seq clustering result (e.g., from scSPARKL on a mixed-species
  sample like the Jurkat–293T mixture) must be checked against known marker genes
  and a ground-truth cell partition.
when_to_use_negative:
- The dataset is multi-omics (CITE-seq, scATAC-seq, scBS-seq) — no Spark-based pipelines
  for these exist yet in scSPARKL.
- You require benchmarked clustering performance against Seurat or Scanpy — the authors
  explicitly note no such benchmarking was performed.
- Input is not .csv or .parquet — the pipeline currently supports only these formats.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scSPARKL
  role: Runs the clustering stage whose assignments are validated, and writes cluster_assignments.csv
    for the ARI/marker check
  repo: https://www.github.com/asif7adil/scSPARKL
- name: Apache Spark
  role: Distributed execution engine underlying scSPARKL's Spark-native clustering
    and PCA on the cell mixture
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-12897-5
    title: Enabling scalable single-cell transcriptomic analysis through distributed
      computing with Apache spark
  source_content_hash: sha256:5d8b26c28deb8602283031c83078b2a8aa6a68754998733494adbd92ecefeaa1
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cell-clustering-validation-with-marker-genes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates Spark-based single-cell clustering output by checking species/cell-type marker gene expression (e.g., CD3D for mouse Jurkat, XIST for human 293T) and computing the Adjusted Rand Index (ARI) against the expected partition.

## When to use

Use when a scRNA-seq clustering result (e.g., from scSPARKL on a mixed-species sample like the Jurkat–293T mixture) must be checked against known marker genes and a ground-truth cell partition.

## When NOT to use

- The dataset is multi-omics (CITE-seq, scATAC-seq, scBS-seq) — no Spark-based pipelines for these exist yet in scSPARKL.
- You require benchmarked clustering performance against Seurat or Scanpy — the authors explicitly note no such benchmarking was performed.
- Input is not .csv or .parquet — the pipeline currently supports only these formats.

## Inputs

- cluster_assignments.csv (per-cell cluster assignments from the scSPARKL clustering stage)
- Normalized count matrix (wide or tall format) retaining marker gene expression (CD3D, XIST)
- Expected cell partition (e.g., mouse Jurkat vs. human 293T labels for the 3,388-cell mixture)

## Outputs

- Marker-gene expression check confirming species-specific cluster separation
- Adjusted Rand Index (ARI) score quantifying agreement with the expected partition

## How to apply

After scSPARKL writes per-cell cluster assignments to cluster_assignments.csv, inspect expression of species markers such as CD3D (mouse Jurkat) and XIST (human 293T) to confirm clusters align with known biology. Then compute the Adjusted Rand Index (ARI) between the assigned clusters and the expected species partition; for the 3,388-cell Jurkat–293T benchmark, the paper reports an ARI of approximately 0.90 as the expected value. Validate only after the full pipeline (melting, data_filter() with thresholds like cells expressing < 10 genes, genes in < 3 cells, dropout > 70%) has run, since poor QC or normalization (global/CPM-like or quantile) will corrupt the clustering being validated.

## Related tools

- **scSPARKL** (Runs the clustering stage whose assignments are validated, and writes cluster_assignments.csv for the ARI/marker check) — https://www.github.com/asif7adil/scSPARKL
- **Apache Spark** (Distributed execution engine underlying scSPARKL's Spark-native clustering and PCA on the cell mixture)

## Evaluation signals

- ARI against the expected Jurkat–293T species partition is approximately 0.90, matching the paper's reported benchmark.
- CD3D and XIST expression patterns cleanly separate mouse and human cells across assigned clusters.
- Per-cell cluster assignments file (cluster_assignments.csv) covers all cells passing the data_filter() QC thresholds (e.g., < 10 genes per cell, < 3 cells per gene, dropout > 70% filtered).
- Clustering follows Spark-native PCA dimensionality reduction, confirming the correct pipeline stage ordering (normalize → HVG selection → PCA → cluster).

## Limitations

- Repository is not actively maintained; a new version is under development per the README warning.
- Cluster mode operations are not yet optimized and are deferred to future development.
- No benchmarking against Seurat and Scanpy was performed, so cross-tool concordance of the clustering result is uncharacterized.
- Tested on Microsoft Windows 11 only; other platforms are untested per the README.

## Evidence

- [results] scSPARKL achieved an ARI of approximately 0.90, indicating strong agreement with the expected separation between human and mouse cells: "scSPARKL achieved an ARI of approximately 0.90, indicating strong agreement with the expected separation between human and mouse cells"
- [results] in the Jurkat–293T cell mixture, scSPARKL clearly distinguished between mouse and human cells: "in the Jurkat–293T cell mixture, scSPARKL clearly distinguished between mouse and human cells"
- [methods] The quality summaries are then passed as arguments to the `data_filter()` to filter out the unwanted genes and cells.: "The quality summaries are then passed as arguments to the `data_filter()` to filter out the unwanted genes and cells."
- [readme] Download the source code and run either the Jupyter Notebook or the scSPARKL script file.: "Download the source code and run either the Jupyter Notebook or the scSPARKL script file."
