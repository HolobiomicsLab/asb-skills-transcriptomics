---
name: pca-dimensionality-reduction-of-expression-matrices
description: Use when comparing dropout-imputation methods on scRNA-seq count matrices
  and you need a low-dimensional embedding for PCA/t-SNE visualization and spectral
  clustering against known cell-type labels.
when_to_use_negative:
- When the number of cell types is unknown and no label file exists — the workflow
  assumes a known cluster count (ccluster = c(7,"Spectrum")) and labels for metric
  computation
- When the goal is differential expression or pseudotime inference rather than clustering/visualization
  benchmarking — the paper evaluates those separately and no single method wins across
  criteria
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Bfimpute
  role: Produces the Bayesian factorization-imputed matrix whose PCA embedding and
    spectral-clustering accuracy are benchmarked against raw counts and five other
    imputers
  repo: github.com/maiziezhoulab/Bfimpute
- name: Spectrum
  role: Spectral clustering on the PCA-projected cells using the known cell-type number
    to compute ARI/Jaccard/NMI/purity
- name: scImpute
  role: Comparison imputation method whose imputed matrix is PCA-projected and clustered
    with the same metrics
- name: SAVER
  role: Comparison imputation method evaluated through the same PCA/clustering/correlation
    pipeline
- name: VIPER
  role: Comparison imputation method evaluated through the same PCA/clustering/correlation
    pipeline
- name: DrImpute
  role: Comparison imputation method evaluated through the same PCA/clustering/correlation
    pipeline
- name: MAGIC
  role: Comparison imputation method evaluated through the same PCA/clustering/correlation
    pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:c462fad6b6834e5aa072334edfb8fcd03ced1f4cacb204975b40c9239699e27d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/maiziezhoulab/Bfimpute
  tools:
  - Bfimpute
  - DrImpute
  - MAGIC
  - SAVER
  - Spectrum
  - VIPER
  - scImpute
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# pca-dimensionality-reduction-of-expression-matrices

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Project raw or imputed scRNA-seq gene expression matrices (e.g., Chu GSE75748, Tang GSE100911) onto principal components to visualize cell-type separation and to feed spectral clustering, enabling quantitative comparison of imputation methods via ARI, Jaccard, NMI, purity, and same/different-cell-type Pearson correlations.

## When to use

Use when comparing dropout-imputation methods on scRNA-seq count matrices and you need a low-dimensional embedding for PCA/t-SNE visualization and spectral clustering against known cell-type labels.

## When NOT to use

- When the number of cell types is unknown and no label file exists — the workflow assumes a known cluster count (ccluster = c(7,"Spectrum")) and labels for metric computation
- When the goal is differential expression or pseudotime inference rather than clustering/visualization benchmarking — the paper evaluates those separately and no single method wins across criteria

## Inputs

- QC-filtered scRNA-seq count matrix (CSV, genes x cells, e.g., sc_qc.csv from Chu GSE75748)
- Cell type label file (cell_type.csv) with known number of cell types (7 for Chu)
- Imputed expression matrices from Bfimpute, scImpute, SAVER, VIPER, DrImpute, and MAGIC
- Tang zebrafish (GSE100911) count matrix and labels for cross-dataset validation

## Outputs

- PCA/t-SNE visualizations of cells per method
- Clustering-metric table (ARI, Jaccard, nmi, purity) per imputation method vs raw counts
- Same/different-cell-type Pearson correlation barplot data (CSV) mirroring Figures 3A-3C and S3

## How to apply

Load the QC-filtered count matrix (e.g., sc_qc.csv from Chu et al. 2016 with 7 known cell types) plus the raw and imputed variants (Bfimpute, scImpute, SAVER, VIPER, DrImpute, MAGIC outputs). For each matrix, project cells onto principal components, then run spectral clustering (Spectrum or specc) with the known number of cell types (7 for Chu). Compute clustering agreement metrics (ARI, Jaccard index, NMI, purity) against the provided cell_type.csv labels, and compute same-cell-type versus different-cell-type Pearson correlations of gene expression. On the Chu dataset expect same-type correlations of ~0.70 raw, ~0.87 Bfimpute, ~0.97 MAGIC; write metric tables and barplot data to CSV for comparison with Figures 3A-3C and S3. Rationale: PCA is the shared embedding that makes raw-vs-imputed clustering accuracy and same-cell-type correlation improvements directly comparable.

## Related tools

- **Bfimpute** (Produces the Bayesian factorization-imputed matrix whose PCA embedding and spectral-clustering accuracy are benchmarked against raw counts and five other imputers) — github.com/maiziezhoulab/Bfimpute
- **Spectrum** (Spectral clustering on the PCA-projected cells using the known cell-type number to compute ARI/Jaccard/NMI/purity)
- **scImpute** (Comparison imputation method whose imputed matrix is PCA-projected and clustered with the same metrics)
- **SAVER** (Comparison imputation method evaluated through the same PCA/clustering/correlation pipeline)
- **VIPER** (Comparison imputation method evaluated through the same PCA/clustering/correlation pipeline)
- **DrImpute** (Comparison imputation method evaluated through the same PCA/clustering/correlation pipeline)
- **MAGIC** (Comparison imputation method evaluated through the same PCA/clustering/correlation pipeline)

## Examples

```
counts = read.csv(paste0(data_dir, "sc_qc.csv"), row.names = 1, header = T); counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), ncores = 5)
```

## Evaluation signals

- Same-cell-type Pearson correlation on Chu increases from ~0.70 (raw) to ~0.87 for Bfimpute, with MAGIC at ~0.97 and scImpute at ~0.85
- Bfimpute best preserves cross-type variation in the PCA embedding while raising within-type correlation
- ARI, Jaccard, NMI, and purity tables computed with 7 spectral clusters on Chu reproduce the ordering shown in Figures 3A-3C
- Each metric table and correlation barplot-data table is written to CSV successfully for all six imputed matrices plus raw counts

## Limitations

- There is no consensus about which imputation method is the best; performance varies across evaluation criteria, datasets, and downstream analyses
- It is premature to conclude that Bfimpute will achieve better performance across all datasets; generalizability beyond the benchmarked datasets (Chu, Tang) is unconfirmed
- Imputation may increase false positives in downstream analysis; parameters (e.g., D, totalepoch, burnin) trade accuracy against runtime

## Evidence

- [results] same-cell-type Pearson correlation from 0.70 (raw) to 0.87 for Bfimpute: [evidence span withheld — anchor status: not_found]
