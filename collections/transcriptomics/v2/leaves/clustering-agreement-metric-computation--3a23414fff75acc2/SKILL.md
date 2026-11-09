---
name: clustering-agreement-metric-computation
description: 'Use when benchmarking imputed versus raw scRNA-seq expression matrices:
  after clustering cells, you need ARI, Jaccard, NMI, and purity against known cell
  type labels to compare clustering accuracy across methods.'
when_to_use_negative:
- When cell labels are available during imputation itself, Bfimpute can use them directly
  via ccluster='labeled' rather than clustering; metrics are still needed for evaluation
  but the clustering step inside imputation is replaced
- When the number of cell types is unknown — spectral clustering here requires specifying
  the number of clusters (e.g., c(7,"Spectrum"))
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Bfimpute
  role: Produces the imputed expression matrix whose cluster agreement against raw-data
    and competing-imputation clusterings is being evaluated
  repo: https://github.com/maiziezhoulab/Bfimpute
- name: Spectrum
  role: Spectral clustering used to assign cells to the known number of types before
    computing ARI/Jaccard/NMI/purity
- name: scImpute
  role: Comparison imputation method whose output matrix is clustered and scored with
    the same metrics
- name: SAVER
  role: Comparison imputation method scored with the same clustering agreement metrics
- name: VIPER
  role: Comparison imputation method scored with the same clustering agreement metrics
- name: DrImpute
  role: Comparison imputation method scored with the same clustering agreement metrics
- name: MAGIC
  role: Comparison imputation method scored with the same clustering agreement metrics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:f8e3fe29ff637507e55dc2ab45d2a8d4ded21a0d455c64d88104b6ec19cf6df6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/maiziezhoulab/Bfimpute
  repo_url: https://github.com/maiziezhoulab/Bfimpute
  tools:
  - Bfimpute
  - DrImpute
  - MAGIC
  - SAVER
  - Spectrum
  - VIPER
  - scImpute
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# clustering-agreement-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes agreement metrics (ARI, Jaccard index, NMI, and purity) between spectral-clustering assignments and known cell type labels to quantify how well an scRNA-seq imputation method (e.g., Bfimpute) recovers cell-to-cell structure on datasets such as Chu (GSE75748) and Tang (GSE100911).

## When to use

Use when benchmarking imputed versus raw scRNA-seq expression matrices: after clustering cells, you need ARI, Jaccard, NMI, and purity against known cell type labels to compare clustering accuracy across methods.

## When NOT to use

- When cell labels are available during imputation itself, Bfimpute can use them directly via ccluster='labeled' rather than clustering; metrics are still needed for evaluation but the clustering step inside imputation is replaced
- When the number of cell types is unknown — spectral clustering here requires specifying the number of clusters (e.g., c(7,"Spectrum"))

## Inputs

- Imputed or raw gene-by-cell expression count matrix (e.g., sc_qc.csv from Chu et al 2016 QC-filtered dataset)
- Known cell type labels (cell_type.csv)
- Known number of cell types (e.g., 7 for Chu dataset)

## Outputs

- Table of ARI, Jaccard index, NMI, and purity per method/dataset (CSV)
- Cluster assignments from spectral clustering for each expression matrix

## How to apply

Take each expression matrix (raw counts or the matrix imputed by Bfimpute, scImpute, SAVER, VIPER, DrImpute, or MAGIC), project cells onto principal components, and run spectral clustering with the known number of cell types (7 for the Chu dataset) using Spectrum. Compare the resulting cluster assignments against the known cell type labels by computing adjusted Rand index, Jaccard index, normalized mutual information, and purity. Higher values indicate that imputation better recovered cell-to-cell relationships, the key weakness of prior methods. Write the resulting metric tables to CSV for comparison against Figures 3A–3C and S3, alongside same- versus different-cell-type Pearson correlations (raw ~0.70 vs Bfimpute ~0.87 on Chu).

## Related tools

- **Bfimpute** (Produces the imputed expression matrix whose cluster agreement against raw-data and competing-imputation clusterings is being evaluated) — https://github.com/maiziezhoulab/Bfimpute
- **Spectrum** (Spectral clustering used to assign cells to the known number of types before computing ARI/Jaccard/NMI/purity)
- **scImpute** (Comparison imputation method whose output matrix is clustered and scored with the same metrics)
- **SAVER** (Comparison imputation method scored with the same clustering agreement metrics)
- **VIPER** (Comparison imputation method scored with the same clustering agreement metrics)
- **DrImpute** (Comparison imputation method scored with the same clustering agreement metrics)
- **MAGIC** (Comparison imputation method scored with the same clustering agreement metrics)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), ncores = 5)
```

## Evaluation signals

- ARI, Jaccard, NMI, and purity computed against known labels should be reported for every method on the same PC/spectral-clustering setup (7 clusters for Chu)
- Bfimpute-imputed data should improve clustering accuracy over raw data and the five comparison methods on the benchmark datasets
- Same-cell-type Pearson correlation should rise from ~0.70 (raw Chu) to ~0.87 for Bfimpute, with Bfimpute best preserving cross-type variation
- Metric tables and Pearson-correlation barplot data should reproduce the values shown in Figures 3A–3C and S3 when written to CSV

## Limitations

- There is no consensus about which imputation method is the best; performance varies across evaluation criteria, datasets, and downstream analyses
- It is premature to conclude Bfimpute will achieve better performance across all datasets; generalizability beyond the benchmarked datasets is unconfirmed
- Imputation may increase false positives in downstream analysis; parameters may need tuning depending on the application

## Evidence

- [methods] Workflow step on metrics: [evidence span withheld — anchor status: not_found]
- [results] Research question on clustering accuracy: [evidence span withheld — anchor status: not_found]
- [results] Chu Pearson correlation finding: [evidence span withheld — anchor status: not_found]
