---
name: nonnegative-kernel-autoencoder-gene-preselection
description: Use when an n × d scRNA-seq expression matrix has far more genes than
  needed (e.g., >20,000 detected genes) and a fixed-size, variance-informed gene subset
  (5000 genes) is required before filter-based feature selection or clustering.
when_to_use_negative:
- Input already contains fewer than ~5000 informative genes, or genes have already
  been pre-selected — the autoencoder cut of exactly 5000 genes would discard usable
  signal or add nothing
- The goal is supervised marker-gene discovery rather than unsupervised preprocessing
  — this filter uses no label information by design
- Data are raw counts that must not be log2-transformed (e.g., downstream tools requiring
  raw UMI counts)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scEFSC
  role: Pipeline embedding the non-negative kernel autoencoder as its gene pre-selection
    step, feeding the 5000-gene subsets into Low Variance, Laplacian Score, SPEC,
    and MCFS feature selection
  repo: https://github.com/Conan-Bian/scEFSC
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:32f75446bb29667424e8a7dcffa8ba08465c7fc28bdd0058df8984f6d6290b7e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Conan-Bian/scEFSC
  repo_url: https://github.com/Conan-Bian/scEFSC
  tools:
  - CIDR
  - Monocle
  - RaceID
  - Rphenograph
  - SC3
  - SHARP
  - SINCERA
  - Seurat
  - pcaReduce
  - scEFSC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# nonnegative-kernel-autoencoder-gene-preselection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A dimensionality-reduction pre-selection step from scEFSC that uses a single-layer non-negative kernel autoencoder to retain the 5000 most significant genes from a log2-normalized scRNA-seq matrix before downstream unsupervised feature selection and ensemble clustering.

## When to use

Use when an n × d scRNA-seq expression matrix has far more genes than needed (e.g., >20,000 detected genes) and a fixed-size, variance-informed gene subset (5000 genes) is required before filter-based feature selection or clustering.

## When NOT to use

- Input already contains fewer than ~5000 informative genes, or genes have already been pre-selected — the autoencoder cut of exactly 5000 genes would discard usable signal or add nothing
- The goal is supervised marker-gene discovery rather than unsupervised preprocessing — this filter uses no label information by design
- Data are raw counts that must not be log2-transformed (e.g., downstream tools requiring raw UMI counts)

## Inputs

- n × d scRNA-seq expression matrix (genes as rows, cells/samples as columns) from GEO accessions such as GSE36552, GSE83139, GSE81861, GSE59739, GSE81252, GSE81608, GSE71585, GSE85241, E-MTAB-5061, or GSE65525
- Known number of cell types for the dataset (used downstream, e.g. 14 cell types for GSE36552)

## Outputs

- A reduced gene × cell expression matrix restricted to the 5000 pre-selected genes
- Encoder-weight variance ranking over genes used to select the 5000-gene subset

## How to apply

First pre-process the scRNA-seq count matrix: apply a log2 transformation to normalize the data (log2 rescale if expression values exceed 100) and remove genes detected in less than 2% of the cells to filter out low-level expressed genes. Then train a single-layer non-negative kernel autoencoder on the pre-processed matrix and rank genes by the variance of their encoder weights, retaining genes with large encoder-weight variances to pre-select exactly 5000 genes for downstream analysis. The rationale is that clustering algorithms implicitly treat all features as equally significant, so insignificant genes should be removed before the four unsupervised filter selections (Low Variance, Laplacian Score, SPEC, MCFS) are applied to the reduced set.

## Related tools

- **scEFSC** (Pipeline embedding the non-negative kernel autoencoder as its gene pre-selection step, feeding the 5000-gene subsets into Low Variance, Laplacian Score, SPEC, and MCFS feature selection) — https://github.com/Conan-Bian/scEFSC

## Evaluation signals

- The pre-selected matrix contains exactly 5000 genes (rows) and all original cells (columns)
- The input matrix passed the pre-filters: log2 transformation applied and genes detected in <2% of cells removed before autoencoder training
- Selected genes correspond to the top-ranking encoder-weight variances from the non-negative kernel autoencoder
- Downstream, each of the four feature subsets derived from the 5000 genes yields non-degenerate clusterings (pairwise ARI variance across base labels is not collapsed)
- Final ensemble labels compared to published cell-type labels by NMI and ARI; scEFSC's reported behavior was best NMI on 11 of 14 datasets

## Limitations

- The 5000-gene cut is a fixed threshold applied across all fourteen datasets, which may over- or under-select for datasets with very different gene counts
- The pre-selection is unsupervised and variance-based, so rare cell-type marker genes with low overall variance may be removed before clustering
- It is only the first stage of scEFSC: results depend on the subsequent Low Variance, Laplacian Score, SPEC, and MCFS filters and the wMetaC weighted meta-clustering (δ=0.01, ward.D)

## Evidence
