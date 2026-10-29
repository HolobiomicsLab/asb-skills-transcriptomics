---
name: r-package-workflow-execution
description: Use when the input is a CSV data frame of significantly changed genes
  with gene identifiers and log-fold-change (LFC) values, and the goal is to group
  genes into expression-pattern clusters for downstream exploration.
when_to_use_negative:
- Input data is not a properly formatted CSV data frame with the relevant experimental
  information (e.g. gene identifiers and LFC values) — OmicInt explicitly requires
  the user to prepare data in the right format
- The goal is protein type and cellular location classification, epigenomics analysis,
  or interactome assessment via UniProt/STRINGdb — these are separate OmicInt functionalities,
  not the GMM clustering pipeline
- Genes are not significantly changed genes with log-fold-change values — the package
  builds on exploring gene networks using significantly changed genes and their LFC
  values
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: OmicInt
  role: R package providing the easy Gaussian mixture modelling pipeline used to cluster
    genes by expression pattern (v1.1.7 used in the methods)
  repo: https://github.com/AusteKan/OmicInt
- name: R
  role: Runtime environment in which the OmicInt package is executed to load the CSV
    data frame and fit the GMM
- name: mclust
  role: OmicInt dependency supplying the Gaussian mixture model implementation underlying
    the probability-based gene cluster classification
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.ailsci.2021.100025
    title: 'OmicInt package: Exploring omics data and regulatory networks using integrative
      analyses and machine learning'
  source_content_hash: sha256:f5f17304b27c26b437b70e661edcfe171f3aaef18cb3071b547aaaed4537a21f
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# r-package-workflow-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes the OmicInt R package's mclust-based Gaussian mixture modelling (GMM) pipeline, which uses probability-based classification to assign significantly changed genes to distinct expression-pattern clusters from a user-prepared CSV data frame.

## When to use

Use when the input is a CSV data frame of significantly changed genes with gene identifiers and log-fold-change (LFC) values, and the goal is to group genes into expression-pattern clusters for downstream exploration.

## When NOT to use

- Input data is not a properly formatted CSV data frame with the relevant experimental information (e.g. gene identifiers and LFC values) — OmicInt explicitly requires the user to prepare data in the right format
- The goal is protein type and cellular location classification, epigenomics analysis, or interactome assessment via UniProt/STRINGdb — these are separate OmicInt functionalities, not the GMM clustering pipeline
- Genes are not significantly changed genes with log-fold-change values — the package builds on exploring gene networks using significantly changed genes and their LFC values

## Inputs

- User-prepared gene-expression data frame in CSV format containing gene identifiers and log-fold-change (LFC) values

## Outputs

- gene_cluster_assignments.csv: one row per input gene with the assigned expression-pattern cluster label and membership probability fields

## How to apply

First verify the supplied data frame is in CSV format and contains all relevant experimental information, including gene identifiers and LFC values, as OmicInt requires the data to be in the right format. Run the OmicInt R package (v1.1.7, distributed on CRAN), which wraps mclust, to fit a Gaussian mixture model to the expression data. Rely on the GMM's probability-based classification so that each data point assignment carries a probability of belonging to one of the clusters; retain the per-cluster membership probabilities for every gene. Assemble a per-gene table mapping each gene to its assigned cluster/pattern plus its membership probabilities, and write it to gene_cluster_assignments.csv. Because the pipeline was designed to make the analyses straightforward for non-experts, no advanced machine-learning tuning is expected from the user; a pattern of interest can then be selected to further explore genes that changed their expression in a specific manner.

## Related tools

- **OmicInt** (R package providing the easy Gaussian mixture modelling pipeline used to cluster genes by expression pattern (v1.1.7 used in the methods)) — https://github.com/AusteKan/OmicInt
- **R** (Runtime environment in which the OmicInt package is executed to load the CSV data frame and fit the GMM)
- **mclust** (OmicInt dependency supplying the Gaussian mixture model implementation underlying the probability-based gene cluster classification)

## Examples

```
Rscript -e 'library(OmicInt)'
```

## Evaluation signals

- gene_cluster_assignments.csv exists after the run
- gene_cluster_assignments.csv contains one row per input gene
- Each row includes the cluster label and probability fields
- All membership probability values fall within the range 0 to 1
- Genes are grouped into distinct expression-pattern types from which a specific pattern of interest can be selected

## Limitations

- The user must ensure the supplied data is in the right format; OmicInt offers several options to prepare a data frame (CSV format) containing all the relevant experimental information, and malformed input will break the pipeline
- Cluster assignment is probability-based rather than deterministic — each data point has a different probability of belonging to one of the clusters, so membership confidence must be considered downstream
- The pipeline targets significantly changed genes with log-fold-change (LFC) values as the key analytical parameter; other data types fall outside its design

## Evidence
