---
name: labeled-cell-type-guided-imputation
description: Use when the scRNA-seq count matrix comes with trusted cell type labels
  (e.g., a published dataset like GSE118068 with annotations) and you want group-wise
  dropout imputation instead of unsupervised spectral clustering.
when_to_use_negative:
- No reliable cell type labels exist — then supply a cluster count and method (e.g.,
  c(7,"Spectrum") or c(7,"specc")) or a custom ccluster function instead of "labeled"
- Counts are already normalized — set normalized=TRUE to avoid double CPM/log10 transformation
  (though log-transform after normalization is still recommended)
- You need a single universally best imputation method — the authors caution no consensus
  exists and it is premature to conclude Bfimpute wins on all datasets
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Bfimpute
  role: Bayesian factorization imputation tool run in labeled mode (ccluster="labeled",
    label=cell_type) to impute dropouts within known cell groups
  repo: github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Comparison imputation method run in labeled mode during benchmarking
- name: DrImpute
  role: Comparison imputation method run in labeled mode during benchmarking
- name: SAVER
  role: Comparison imputation method run in labeled mode during benchmarking
- name: MAGIC
  role: Comparison imputation method (default settings) in the large-scale benchmark
- name: VIPER
  role: Comparison imputation method (default settings) in the large-scale benchmark
- name: PBLR
  role: Comparison imputation method run in labeled mode during benchmarking
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:9ead31a1ae2f957c004d8e81c00c4837fe3c36be9a46eaa94049d5336bfe0635
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
  - PBLR
  - SAVER
  - VIPER
  - scImpute
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# labeled-cell-type-guided-imputation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run Bfimpute in labeled mode, supplying ground-truth cell type labels via ccluster="labeled" and label= so imputation is performed within each known cell group rather than via clustering. This leverages similar-cell information from verified annotations to recover dropouts more accurately.

## When to use

Use when the scRNA-seq count matrix comes with trusted cell type labels (e.g., a published dataset like GSE118068 with annotations) and you want group-wise dropout imputation instead of unsupervised spectral clustering.

## When NOT to use

- No reliable cell type labels exist — then supply a cluster count and method (e.g., c(7,"Spectrum") or c(7,"specc")) or a custom ccluster function instead of "labeled"
- Counts are already normalized — set normalized=TRUE to avoid double CPM/log10 transformation (though log-transform after normalization is still recommended)
- You need a single universally best imputation method — the authors caution no consensus exists and it is premature to conclude Bfimpute wins on all datasets

## Inputs

- Raw scRNA-seq expression count matrix (rows = genes, columns = cells), e.g., CSV
- Cell type label vector or one-row/one-column data.frame aligned to the count matrix columns

## Outputs

- Imputed gene expression matrix (R_calculate), returned in R and saved to out_dir (default ./Bfimpute/) as csv/txt/rds per out_type
- Optionally the G and C latent factor matrices when returnGC=TRUE

## How to apply

Load the gene-by-cell raw count matrix and the cell type label vector (vector, data.frame, or one-row/column matrix). Call Bfimpute with ccluster="labeled" and label=cell_type so cells are grouped by annotation and imputation is carried out within each cell group, building cell-type-specific latent matrices; if the counts are unnormalized, Bfimpute internally performs CPM and log10 transform with bias 1.01 (leave normalized=FALSE). Keep default hyperparameters unless scale demands otherwise: D=32, totalepoch=300, burnin=200, sn_max=10, sn_init=1, threshold=0.5, ncores=5; on very large datasets (e.g., the 62,893-cell Vladoiu cerebellum data with 91.9% dropout) these defaults preserved feasibility. Rationale: labeled groups avoid clustering error and more accurately leverage information from similar cells, improving downstream t-SNE, TSCAN pseudotime (POS, Kendall), and clustering accuracy versus scImpute, DrImpute, SAVER, MAGIC, VIPER, and PBLR.

## Related tools

- **Bfimpute** (Bayesian factorization imputation tool run in labeled mode (ccluster="labeled", label=cell_type) to impute dropouts within known cell groups) — github.com/maiziezhoulab/Bfimpute
- **scImpute** (Comparison imputation method run in labeled mode during benchmarking)
- **DrImpute** (Comparison imputation method run in labeled mode during benchmarking)
- **SAVER** (Comparison imputation method run in labeled mode during benchmarking)
- **MAGIC** (Comparison imputation method (default settings) in the large-scale benchmark)
- **VIPER** (Comparison imputation method (default settings) in the large-scale benchmark)
- **PBLR** (Comparison imputation method run in labeled mode during benchmarking)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5)
```

## Evaluation signals

- t-SNE embeddings of the imputed matrix recover known cerebellar cell populations better than raw counts and competing imputations
- TSCAN pseudotime scores (POS and Kendall's rank correlation) on the imputed matrix exceed those of scImpute, DrImpute, SAVER, MAGIC, VIPER, and PBLR (as in Figure S6)
- Clustering metrics computed on the imputed data show the best accuracy among compared imputation methods
- Wall-clock runtime stays competitive — the benchmark ordering MAGIC < PBLR < Bfimpute < scImpute < SAVER should reproduce (DrImpute and VIPER fail on the 62,893-cell dataset)
- Sanity check on inputs: label vector length matches the count matrix column count, and counts remain unnormalized (normalized=FALSE) so CPM + log10(1.01 bias) is applied exactly once

## Limitations

- Generalizability beyond benchmarked datasets is unconfirmed — 'it is premature to conclude that Bfimpute will achieve better performance across all datasets'
- Imputation can increase false positives in downstream analyses; parameters (e.g., threshold on dropout probabilities) must be tuned per application
- Larger D, totalepoch, and burnin improve accuracy but increase runtime; smaller values save time at an accuracy cost

## Evidence

- [results] Vladoiu benchmark result: [evidence span withheld — anchor status: not_found]
