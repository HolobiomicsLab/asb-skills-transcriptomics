---
name: probabilistic-cluster-assignment
description: Use when a gene-expression data frame (CSV with gene identifiers and
  log-fold-change values) must be partitioned into expression-pattern types with probability-based
  cluster assignments.
when_to_use_negative:
- Input data is not a gene-expression data frame with LFC values in the right CSV
  format — the user must ensure the supplied data is in the right format before applying
  the GMM pipeline
- The goal is classification of protein type or cellular location, or epigenomics
  analysis, which are separate OmicInt utilities rather than the GMM clustering pipeline
- Hard (non-probabilistic) cluster labels are required with no per-point membership
  probabilities, contradicting the probability-based classification the GMM pipeline
  provides
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: OmicInt
  role: R package (v1.1.7) providing the easy Gaussian mixture modelling (GMM) pipeline
    for integrative analysis of gene expression data
  repo: https://github.com/AusteKan/OmicInt
- name: mclust
  role: Dependency of OmicInt underlying the Gaussian mixture model fitting for probability-based
    cluster classification
- name: R
  role: Language environment in which the OmicInt GMM pipeline is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ailsci.2021.100025
    title: 'OmicInt package: Exploring omics data and regulatory networks using integrative
      analyses and machine learning'
  source_content_hash: sha256:3de913ba5063e0735d36058f51cf6df33d6e2fdc7c4a2954f45e1699fc84282d
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# probabilistic-cluster-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns genes to expression-pattern clusters by fitting a Gaussian mixture model (GMM) via OmicInt's mclust-based pipeline in R, retaining per-gene membership probabilities so that a pattern of interest can be selected for downstream exploration.

## When to use

Use when a gene-expression data frame (CSV with gene identifiers and log-fold-change values) must be partitioned into expression-pattern types with probability-based cluster assignments.

## When NOT to use

- Input data is not a gene-expression data frame with LFC values in the right CSV format — the user must ensure the supplied data is in the right format before applying the GMM pipeline
- The goal is classification of protein type or cellular location, or epigenomics analysis, which are separate OmicInt utilities rather than the GMM clustering pipeline
- Hard (non-probabilistic) cluster labels are required with no per-point membership probabilities, contradicting the probability-based classification the GMM pipeline provides

## Inputs

- User-prepared gene-expression data frame in CSV format containing gene identifiers and log-fold-change (LFC) values

## Outputs

- gene_cluster_assignments.csv: per-gene table mapping each gene to its assigned expression-pattern cluster with membership probability fields (values in 0-1)

## How to apply

Prepare the user data frame in CSV format, verifying it contains the relevant experimental information such as gene identifiers and LFC values, since OmicInt requires the supplied data to be in the right format. Fit a Gaussian mixture model to the expression data using the mclust-based GMM pipeline provided by the OmicInt R package (v1.1.7); GMMs are chosen because they employ a probability-based classification where each data point assignment has a different probability of belonging to one of the clusters. Use the fitted model's classification to assign each gene to an expression pattern cluster, retaining the per-cluster membership probabilities for every data point. Assemble the per-gene table mapping each gene to its assigned cluster/pattern and membership probabilities, write it to gene_cluster_assignments.csv, and validate that probabilities fall within 0 to 1 before selecting a pattern of interest for further downstream exploration.

## Related tools

- **OmicInt** (R package (v1.1.7) providing the easy Gaussian mixture modelling (GMM) pipeline for integrative analysis of gene expression data) — https://github.com/AusteKan/OmicInt
- **mclust** (Dependency of OmicInt underlying the Gaussian mixture model fitting for probability-based cluster classification)
- **R** (Language environment in which the OmicInt GMM pipeline is executed)

## Examples

```
Rscript -e 'library(OmicInt); fit <- GMM_pipeline(expression_data); assignments <- fit$classification; write.csv(assignments, "gene_cluster_assignments.csv")'
```

## Evaluation signals

- gene_cluster_assignments.csv exists after the run
- The output file contains one row per input gene with cluster label and probability fields present
- All membership probability values fall within the range 0 to 1
- The input CSV was verified to contain relevant experimental information (gene identifiers and LFC values) before fitting
- Genes are grouped into distinct expression-pattern types from which a specific pattern of interest can be selected

## Limitations

- The user must ensure the supplied data is in the right format (CSV data frame with all relevant experimental information) or the pipeline cannot be applied correctly
- The pipeline is designed for non-experts, but probability-based assignment means each gene's cluster membership is probabilistic and should be retained for downstream judgement
- The article/README does not specify the number of clusters or model-selection settings; the mclust-based fit governs cluster structure

## Evidence
