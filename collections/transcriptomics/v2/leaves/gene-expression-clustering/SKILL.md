---
name: gene-expression-clustering
description: Use when a CSV gene-expression data frame with gene identifiers and log-fold-change
  (LFC) values must be partitioned into distinct expression-pattern clusters with
  per-gene membership probabilities.
when_to_use_negative:
- The input is raw counts or a feature table lacking log-fold-change (LFC) values
  — the article states LFC is the key analytical parameter of the machine learning
  pipeline and exploratory analyses.
- The goal is protein type and cellular location classification, assessment of gene
  clusters via known interactors (UniProt/STRINGdb), or epigenomics analysis rather
  than expression-pattern clustering.
- Genes are already assigned to expression-pattern clusters and only downstream exploration
  of a selected pattern is required.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: OmicInt
  role: R package (v1.1.7) providing the easy GMM pipeline that clusters genes into
    expression-pattern types via probability-based classification
  repo: https://github.com/AusteKan/OmicInt
- name: mclust
  role: Gaussian mixture model fitting dependency underlying OmicInt's probability-based
    cluster assignment of genes
- name: R
  role: Runtime language in which the OmicInt GMM clustering pipeline is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ailsci.2021.100025
    title: 'OmicInt package: Exploring omics data and regulatory networks using integrative
      analyses and machine learning'
  source_content_hash: sha256:633f132dc2eb60f4856dfafbf97a09feb7bbf11a49d0f28e9372df35ad3f00bd
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gene-expression-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

OmicInt's mclust-based Gaussian mixture modelling pipeline clusters genes from a CSV expression data frame into distinct expression-pattern types using probability-based classification, retaining per-cluster membership probabilities so a pattern of interest can be selected for downstream exploration.

## When to use

Use when a CSV gene-expression data frame with gene identifiers and log-fold-change (LFC) values must be partitioned into distinct expression-pattern clusters with per-gene membership probabilities.

## When NOT to use

- The input is raw counts or a feature table lacking log-fold-change (LFC) values — the article states LFC is the key analytical parameter of the machine learning pipeline and exploratory analyses.
- The goal is protein type and cellular location classification, assessment of gene clusters via known interactors (UniProt/STRINGdb), or epigenomics analysis rather than expression-pattern clustering.
- Genes are already assigned to expression-pattern clusters and only downstream exploration of a selected pattern is required.

## Inputs

- User-prepared gene-expression data frame in CSV format containing gene identifiers and log-fold-change (LFC) values for significantly changed genes

## Outputs

- gene_cluster_assignments.csv — per-gene table mapping each gene to its assigned expression-pattern cluster with cluster label and membership probability fields
- Genes grouped into distinct expression-pattern types from which a specific pattern of interest can be selected for further downstream exploration

## How to apply

Load the user-prepared gene-expression data frame in CSV format and verify it contains the relevant experimental information — gene identifiers and log-fold-change (LFC) values, the key analytical parameter of the pipeline. Fit a Gaussian mixture model with the mclust-based GMM pipeline of the OmicInt R package (v1.1.7); GMMs were selected because they employ a probability-based classification where each data point assignment carries a different probability of belonging to one of the clusters. Use the fitted model's probability-based classification to assign each gene to an expression-pattern cluster while retaining the per-cluster membership probabilities for every data point. Assemble a per-gene table mapping each gene to its assigned cluster/pattern and membership probabilities, then write it to gene_cluster_assignments.csv. Decision point: only proceed to select a pattern of interest for downstream exploration once the output validates (probabilities within 0–1, one row per input gene).

## Related tools

- **OmicInt** (R package (v1.1.7) providing the easy GMM pipeline that clusters genes into expression-pattern types via probability-based classification) — https://github.com/AusteKan/OmicInt
- **mclust** (Gaussian mixture model fitting dependency underlying OmicInt's probability-based cluster assignment of genes)
- **R** (Runtime language in which the OmicInt GMM clustering pipeline is executed)

## Examples

```
Rscript -e "library(mclust); fit <- Mclust(df$LFC); write.csv(data.frame(gene = df$gene, cluster = fit$classification, prob = fit$z), 'gene_cluster_assignments.csv', row.names = FALSE)"
```

## Evaluation signals

- gene_cluster_assignments.csv exists after the run.
- gene_cluster_assignments.csv contains one row per input gene (row count matches the input CSV's gene count).
- Cluster label and probability fields are present in the output table.
- All membership probability values fall within the range 0 to 1.
- Genes resolve into distinct expression-pattern types from which a specific pattern of interest can be selected for downstream exploration.

## Limitations

- The user must ensure the supplied data frame (CSV format) contains all the relevant experimental information; malformed or incomplete input is an explicit failure mode flagged in the article.
- Assignments are probabilistic — each data point has a different probability of belonging to one of the clusters — so membership probabilities must be retained and interpreted rather than treated as definitive hard labels.
- Version documentation is inconsistent: README badges show Release V1.1.5 while the methods text cites OmicInt v1.1.7, so verify the installed version before reproducing the pipeline.

## Evidence

- [abstract] Gaussian mixture models (GMMs) were selected as they employ a probability based classification where each data point assignment has a different probability of belonging to one of the clusters: "Gaussian mixture models (GMMs) were selected as they employ a probability based classification where each data point assignment has a different probability of belonging to one of the clusters"
- [abstract] The key analytical parameter in the machine learning pipeline and exploratory analyses is a specific score, namely LFC: "The key analytical parameter in the machine learning pipeline and exploratory analyses is a specific score, namely LFC"
- [abstract] There are several different options to prepare a data frame (CSV format) that contains all the relevant experimental information: "There are several different options to prepare a data frame (CSV format) that contains all the relevant experimental information"
- [abstract] A pattern of interest can be selected to further explore the genes that changed their expression in a specific manner.: "A pattern of interest can be selected to further explore the genes that changed their expression in a specific manner."
- [methods] The package has the following dependencies: ggplot2,mclust,gtools,tidyr,pheatmap: "The package has the following dependencies: ggplot2,mclust,gtools,tidyr,pheatmap"
