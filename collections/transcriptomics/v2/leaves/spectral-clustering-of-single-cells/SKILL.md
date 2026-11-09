---
name: spectral-clustering-of-single-cells
description: Use when an scRNA-seq count matrix must be partitioned into known or
  estimated cell-type groups before Bfimpute imputation, and the number of cell types
  is known (e.g., 7 for Chu et al. 2016).
when_to_use_negative:
- Cell type labels are already provided — use Bfimpute's `ccluster = "labeled"` with
  `label` instead of spectral clustering
- The clustering is for standalone cell-type discovery/downstream analysis rather
  than as the cell-grouping stage preceding Bfimpute imputation
- The preferred clustering method is not spectral (Spectrum/specc) — Bfimpute expects
  you to wrap a custom function whose output is a cluster vector
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Bfimpute
  role: Hosts the clustering step via its `ccluster` parameter; imputation is performed
    within each spectral cluster using full Bayesian inference and MCMC
  repo: github.com/maiziezhoulab/Bfimpute
- name: Spectrum
  role: Spectral clustering engine selectable inside Bfimpute (`ccluster = c(7, "Spectrum")`)
- name: specc
  role: Alternative spectral clustering implementation selectable inside Bfimpute
    (`ccluster = c(7, "specc")`)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:bd4eb5e490cee5612bb450ce5387f6a8b145e24a532480d1c3c4edd0da79b90a
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
  - Leafcutter
  - SCASL
  - SCRABBLE
  - Spectrum
  - netNMF-sc
  - specc
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# spectral-clustering-of-single-cells

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Groups scRNA-seq cells with spectral clustering (Spectrum or kernlab specc) as the pre-imputation cell-grouping step of Bfimpute, so that latent gene/cell factorization is performed within each cell-type group to strengthen cell-to-cell relationships.

## When to use

Use when an scRNA-seq count matrix must be partitioned into known or estimated cell-type groups before Bfimpute imputation, and the number of cell types is known (e.g., 7 for Chu et al. 2016).

## When NOT to use

- Cell type labels are already provided — use Bfimpute's `ccluster = "labeled"` with `label` instead of spectral clustering
- The clustering is for standalone cell-type discovery/downstream analysis rather than as the cell-grouping stage preceding Bfimpute imputation
- The preferred clustering method is not spectral (Spectrum/specc) — Bfimpute expects you to wrap a custom function whose output is a cluster vector

## Inputs

- scRNA-seq expression count matrix (rows = genes, columns = cells), e.g. sc_qc.csv from the Chu et al. (2016) QC'd dataset
- number of cell types (e.g., 7 in the Chu et al. dataset) or pre-existing cell type labels from cell_type.csv

## Outputs

- Cell cluster assignments (vector of cluster labels per cell) consumed internally by Bfimpute
- Cell-type-grouped imputed expression matrix returned/saved by Bfimpute (default out_name 'Bfimpute' as Bfimpute.csv, Bfimpute.txt, Bfimpute.rds)

## How to apply

Pass the cell-grouping directly into Bfimpute via the `ccluster` parameter: when the number of cell types N is known, supply `ccluster = c(N, "Spectrum")` or `c(N, "specc")` to run spectral clustering on the counts matrix. If cell type labels are already available (e.g., cell_type.csv from Chu et al. QC'd data), skip clustering with `ccluster = "labeled"` and `label = cell_type` for more accuracy. A custom clustering function is also supported: it must take a matrix with cells as columns and features as rows and return a vector of cluster assignments (e.g., a kmeans wrapper). Clustering is the first stage of Bfimpute; subsequent imputation, including optional gene/cell side information via S_G/S_C, runs within each resulting cell group. Rationale: Bfimpute builds cell-type-specific latent matrices so imputation leverages information from similar cells.

## Related tools

- **Bfimpute** (Hosts the clustering step via its `ccluster` parameter; imputation is performed within each spectral cluster using full Bayesian inference and MCMC) — github.com/maiziezhoulab/Bfimpute
- **Spectrum** (Spectral clustering engine selectable inside Bfimpute (`ccluster = c(7, "Spectrum")`))
- **specc** (Alternative spectral clustering implementation selectable inside Bfimpute (`ccluster = c(7, "specc")`))

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), ncores = 5)
```

## Evaluation signals

- The number of clusters equals the known cell-type count (7 for the Chu et al. dataset) and cluster assignments align with cell_type.csv labels where available
- Bfimpute accepts the `ccluster` argument without error and completes imputation within each cell group (e.g., counts_imputed returned with ncores = 5)
- PCA projection of the imputed matrix shows cells separating by the labeled cell types, consistent with the article's PCA/correlation validation
- Cell-to-cell correlation against labeled cell types after imputation shows no degradation relative to running Bfimpute without the clustering specification

## Limitations

- Bfimpute's clustering step is limited to spectral approaches (Spectrum, specc) or user-supplied functions; other methods require building a custom function with a matrix input (cells as columns) and cluster vector output
- There is no consensus on which imputation method is best, and clustering-based grouping is one of several factors affecting downstream clustering and trajectory quality
- Using `ccluster = c(7, "Spectrum")` presumes the number of cell types is known a priori

## Evidence
