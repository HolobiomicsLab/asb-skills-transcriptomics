---
name: spectral-clustering-cell-grouping
description: Use when running Bfimpute on an unlabeled scRNA-seq count matrix and
  the number of cell types is known (e.g., k=7 for the Chu et al. 2016 dataset), so
  cells must be grouped before group-wise imputation.
when_to_use_negative:
- True cell type labels are already available — use ccluster="labeled" with the label
  parameter instead of spectral clustering, since the README notes labels 'gain more
  accuracy'
- The count matrix is already normalized and log-transformed — set normalized=TRUE
  rather than letting Bfimpute apply CPM and log10 (bias 1.01) again
- The number of cell types is unknown and cannot be estimated — the Spectrum/specc
  option requires a specific k to be supplied
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Bfimpute
  role: Consumes the ccluster specification, runs spectral clustering (or the user-supplied
    clustering function) to group cells, then imputes dropouts within each group via
    Bayesian factorization with Gibbs sampling
  repo: github.com/maiziezhoulab/Bfimpute
- name: Spectrum
  role: Spectral clustering backend selected via ccluster = c(7,"Spectrum") to assign
    cells to k groups before imputation
- name: kernlab
  role: Provides the specc spectral clustering alternative selected via ccluster =
    c(7,"specc")
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:bbb14f2d9f5a7a567d889f07e68641e9808d8a20de8d98a1e1c103994d64b7b8
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
  - kernlab
  - mnormt
  - parallel
  - rsvd
  - scImpute
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# spectral-clustering-cell-grouping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Group single-cell RNA-seq cells into cell-type clusters using spectral clustering (Spectrum or kernlab specc) so that Bfimpute can impute dropouts within each cell group, leveraging similar cells to strengthen cell-to-cell relationships.

## When to use

Use when running Bfimpute on an unlabeled scRNA-seq count matrix and the number of cell types is known (e.g., k=7 for the Chu et al. 2016 dataset), so cells must be grouped before group-wise imputation.

## When NOT to use

- True cell type labels are already available — use ccluster="labeled" with the label parameter instead of spectral clustering, since the README notes labels 'gain more accuracy'
- The count matrix is already normalized and log-transformed — set normalized=TRUE rather than letting Bfimpute apply CPM and log10 (bias 1.01) again
- The number of cell types is unknown and cannot be estimated — the Spectrum/specc option requires a specific k to be supplied

## Inputs

- Gene-by-cell raw expression count matrix (rows = genes, columns = cells), e.g., sc_qc.csv from the QC'd Chu et al. (2016) dataset
- Known number of cell types k (e.g., 7 for Chu)
- Optional custom clustering function: matrix (cells as columns) in, cluster vector out

## Outputs

- Cell cluster assignments (vector) used internally by Bfimpute to partition cells into cell-type groups
- Imputed gene expression matrix (R_calculate = G^T C), optionally with final-epoch G and C matrices when returnGC=TRUE

## How to apply

Pass ccluster = c(k, "Spectrum") or c(k, "specc") to the Bfimpute() call, where k is the known number of cell types (7 for Chu et al. 2016). Alternatively supply a custom clustering function (e.g., kmeans on transposed cells with centers = k) whose input is a matrix with cells as columns and output is a cluster vector. Bfimpute then performs CPM normalization and log10 transform with bias 1.01 (normalized=FALSE) and imputes within each cell group using Gibbs-sampled latent matrices G and C. If known labels exist, prefer ccluster="labeled" with label=cell_type for more accurate grouping than unsupervised clustering.

## Related tools

- **Bfimpute** (Consumes the ccluster specification, runs spectral clustering (or the user-supplied clustering function) to group cells, then imputes dropouts within each group via Bayesian factorization with Gibbs sampling) — github.com/maiziezhoulab/Bfimpute
- **Spectrum** (Spectral clustering backend selected via ccluster = c(7,"Spectrum") to assign cells to k groups before imputation)
- **kernlab** (Provides the specc spectral clustering alternative selected via ccluster = c(7,"specc"))

## Examples

```
library(Bfimpute); counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), ncores = 5)
```

## Evaluation signals

- The returned cluster vector has exactly k levels matching the supplied number of cell types (7 for Chu)
- Downstream spectral clustering of the imputed matrix yields higher ARI, Jaccard, NMI, and purity against known labels than clustering of the raw counts
- Same-cell-type Pearson correlation on the imputed matrix rises from ~0.70 (raw Chu) toward ~0.87 while cross-type variation is preserved better than MAGIC's ~0.97
- Cluster assignments are accepted by Bfimpute without error — the ccluster input must have cells as columns and a vector as output if a custom function is used

## Limitations

- Performance depends on clustering quality; no consensus exists on the best imputation method and results vary by dataset and downstream analysis
- Imputation can increase false positives in downstream analyses; parameters may need constraining per application
- It is premature to conclude Bfimpute (and hence the clustering-based grouping) improves downstream analysis across all datasets beyond those benchmarked

## Evidence
