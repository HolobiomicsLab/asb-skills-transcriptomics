---
name: gaussian-mixture-model-fitting
description: Use when a user-supplied CSV gene-expression data frame (gene identifiers
  plus log-fold-change values) must be partitioned into distinct expression-pattern
  clusters using unsupervised, probability-based Gaussian mixture modelling.
when_to_use_negative:
- The input is not a properly formatted CSV data frame with gene identifiers and log-fold-change
  values — the article states the user must ensure the supplied data is in the right
  format before the pipeline can run
- The goal is supervised classification with labeled training data — GMMs here are
  unsupervised and assign cluster memberships probabilistically
- The analysis targets proteome-level interactome assessment or protein type / cellular
  location classification rather than expression-pattern clustering of genes
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: OmicInt
  role: Provides the easy Gaussian mixture modelling (GMM) pipeline for integrative
    analysis of gene expression data; version v1.1.7 used in the methods
  repo: https://github.com/AusteKan/OmicInt
- name: mclust
  role: Package dependency underlying the GMM fitting in the OmicInt pipeline
  repo: https://github.com/AusteKan/OmicInt
- name: R
  role: Language in which the OmicInt package and its GMM pipeline are implemented
    and executed
  repo: https://github.com/AusteKan/OmicInt
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ailsci.2021.100025
    title: 'OmicInt package: Exploring omics data and regulatory networks using integrative
      analyses and machine learning'
  source_content_hash: sha256:1b2af0e02c6720a2c3e3c719158db46b8a198bc61fba477a62678313cef17b31
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/AusteKan/OmicInt
  repo_url: https://github.com/AusteKan/OmicInt
  tools:
  - OmicInt
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gaussian-mixture-model-fitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fits a Gaussian mixture model to gene-expression data (gene identifiers with log-fold-change values) using the mclust-based GMM pipeline in the OmicInt R package, assigning each gene to an expression-pattern cluster via probability-based classification.

## When to use

Use when a user-supplied CSV gene-expression data frame (gene identifiers plus log-fold-change values) must be partitioned into distinct expression-pattern clusters using unsupervised, probability-based Gaussian mixture modelling.

## When NOT to use

- The input is not a properly formatted CSV data frame with gene identifiers and log-fold-change values — the article states the user must ensure the supplied data is in the right format before the pipeline can run
- The goal is supervised classification with labeled training data — GMMs here are unsupervised and assign cluster memberships probabilistically
- The analysis targets proteome-level interactome assessment or protein type / cellular location classification rather than expression-pattern clustering of genes

## Inputs

- User-prepared gene-expression data frame in CSV format containing gene identifiers and log-fold-change (LFC) values

## Outputs

- gene_cluster_assignments.csv: per-gene table mapping each gene to its assigned expression-pattern cluster label with per-cluster membership probability fields (values in 0–1)

## How to apply

Load the user-prepared data frame in CSV format and verify it contains the relevant experimental information (gene identifiers and LFC values), since OmicInt requires the user to ensure the supplied data is in the right format. Fit a Gaussian mixture model to the expression data using the mclust-based GMM pipeline provided by the OmicInt R package (v1.1.7); GMMs were chosen because they employ a probability-based classification where each data point assignment has a different probability of belonging to one of the clusters. Use the fitted model's probability-based classification to assign each gene to an expression-pattern cluster while retaining per-cluster membership probabilities for every data point. Assemble a per-gene table mapping each gene to its assigned cluster/pattern and membership probabilities, write it to gene_cluster_assignments.csv, and confirm every membership probability falls within 0 to 1. A pattern of interest can then be selected for further downstream exploration of the genes that changed expression in a specific manner.

## Related tools

- **OmicInt** (Provides the easy Gaussian mixture modelling (GMM) pipeline for integrative analysis of gene expression data; version v1.1.7 used in the methods) — https://github.com/AusteKan/OmicInt
- **mclust** (Package dependency underlying the GMM fitting in the OmicInt pipeline) — https://github.com/AusteKan/OmicInt
- **R** (Language in which the OmicInt package and its GMM pipeline are implemented and executed) — https://github.com/AusteKan/OmicInt

## Evaluation signals

- gene_cluster_assignments.csv exists after the run
- The output contains one row per input gene
- Each row has both a cluster label field and probability fields present
- All membership probability values fall within the range 0 to 1
- Cluster assignments are consistent with the probability-based classification retained from the fitted GMM (per-cluster membership probabilities retained for every data point)

## Limitations

- The user must ensure the supplied data is in the right format (a data frame in CSV format with all relevant experimental information) before the pipeline can be applied
- GMM clustering is probability-based: each data point has a different probability of belonging to each cluster, so assignments are probabilistic rather than hard boundaries
- The pipeline is designed to make advanced analyses straightforward for non-experts, so it offers limited tuning granularity compared with directly configuring mclust
- Version ambiguity in the documentation: the README badge shows Release V1.1.5 while the methods section states Version v1.1.7

## Evidence

- [abstract] GMM probability-based classification: "Gaussian mixture models (GMMs) were selected as they employ a probability based classification where each data point assignment has a different probability of belonging to one of the clusters"
- [abstract] Data format requirement: "the user must ensure that the supplied data is in the right format. There are several different options to prepare a data frame (CSV format) that contains all the relevant experimental information"
- [methods] GMM pipeline in OmicInt: "*OmicInt* provides an easy Gaussian mixture modelling (GMM) [3,4] pipeline for an integrative analysis"
- [readme] Non-expert usability: "*OmicInt* provides an easy Gaussian mixture modelling (GMM) [3,4] pipeline for an integrative analysis that can be used by a non-expert to explore gene expression data"
- [readme] LFC-based exploration basis: "the package builds on a previously developed method to explore gene networks using significantly changed genes, their log-fold-change values (LFC), and the predicted interactome complexity"
