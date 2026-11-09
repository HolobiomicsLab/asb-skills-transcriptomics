---
name: gene-network-scoring-analysis
description: Use when you have a data frame (CSV format) of significantly changed
  genes with log-fold-change values from single-cell RNA-seq or proteomics experiments
  and need per-gene LFC scores for GMM-based gene-network exploration in OmicInt.
when_to_use_negative:
- Input data is not a data frame of significantly changed genes with LFC values (e.g.
  raw counts or unfiltered omics matrices) — OmicInt requires the user to first ensure
  'the supplied data is in the right format'
- The analysis target is epigenomic regulatory-layer exploration, protein type / cellular
  location classification, or interactome assessment via UniProt/STRINGdb rather than
  LFC scoring of changed genes
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: OmicInt
  role: R package whose LFC-based scoring function and Gaussian mixture modelling
    pipeline are reconstructed and run to score genes and explore gene networks
  repo: https://github.com/AusteKan/OmicInt
- name: R
  role: runtime language in which the CSV data frame is loaded and the standalone
    LFC scoring function is implemented and executed
- name: mclust
  role: dependency providing the Gaussian mixture modelling used downstream of the
    LFC score for probabilistic cluster assignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.ailsci.2021.100025
    title: 'OmicInt package: Exploring omics data and regulatory networks using integrative
      analyses and machine learning'
  source_content_hash: sha256:93fd910d56dcc158713ec01a3b4d23ec8aabfd91e72d03bd921d69fd65eeea64
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-network-scoring-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the OmicInt LFC-based score — the key analytical parameter of the package's machine learning pipeline — which captures expression and pathology associations for significantly changed genes and feeds the Gaussian mixture modelling pipeline used to explore genes and linked interactome networks.

## When to use

Use when you have a data frame (CSV format) of significantly changed genes with log-fold-change values from single-cell RNA-seq or proteomics experiments and need per-gene LFC scores for GMM-based gene-network exploration in OmicInt.

## When NOT to use

- Input data is not a data frame of significantly changed genes with LFC values (e.g. raw counts or unfiltered omics matrices) — OmicInt requires the user to first ensure 'the supplied data is in the right format'
- The analysis target is epigenomic regulatory-layer exploration, protein type / cellular location classification, or interactome assessment via UniProt/STRINGdb rather than LFC scoring of changed genes

## Inputs

- gene-expression data frame in CSV format containing significantly changed genes and their log-fold-change (LFC) values
- OmicInt v1.1.7 package source from github.com/AusteKan/OmicInt

## Outputs

- lfc_scores.csv with one scored row per input gene (gene identifier and LFC score columns)

## How to apply

Load the supplied gene-expression data frame in CSV format into R and validate that it contains significantly changed genes and their log-fold-change (LFC) values. Inspect the OmicInt v1.1.7 source (github.com/AusteKan/OmicInt) to reconstruct the LFC-based scoring function, re-expressing it as a standalone R function with the same base R / package dependency semantics (ggplot2, mclust, gtools, tidyr, pheatmap). Run the reconstructed scoring function over all genes, computing one LFC score per gene — this score is the key analytical parameter used together with the Gaussian mixture modelling pipeline to classify each data point probabilistically into clusters. Write per-gene scores to lfc_scores.csv with gene identifier and LFC score columns. Validate by confirming one scored row per input gene and that computed scores match values from the original OmicInt package function on the same input.

## Related tools

- **OmicInt** (R package whose LFC-based scoring function and Gaussian mixture modelling pipeline are reconstructed and run to score genes and explore gene networks) — https://github.com/AusteKan/OmicInt
- **R** (runtime language in which the CSV data frame is loaded and the standalone LFC scoring function is implemented and executed)
- **mclust** (dependency providing the Gaussian mixture modelling used downstream of the LFC score for probabilistic cluster assignment)

## Examples

```
install.packages("OmicInt"); df <- read.csv("gene_expression.csv"); # compute per-gene OmicInt LFC score; write.csv(scores, "lfc_scores.csv", row.names = FALSE)
```

## Evaluation signals

- lfc_scores.csv exists and is valid CSV with gene identifier and LFC score columns
- lfc_scores.csv contains exactly one scored row per gene in the input data frame
- computed LFC scores match values obtained by running the original OmicInt package function on the same input
- input CSV was validated to contain the required columns for significantly changed genes and their expression values before scoring
- scores are usable downstream by the OmicInt GMM pipeline, where each data point assignment has a different probability of belonging to one of the clusters

## Limitations

- The exact LFC scoring formula must be reconstructed from the package source (v1.1.7 on github.com/AusteKan/OmicInt); the article and README do not state the formula explicitly
- Users must ensure the supplied data is in the right format beforehand — OmicInt offers 'several different options to prepare a data frame (CSV format)' and format errors are a user-side failure mode
- Version discrepancy exists between sources: README badges show Release V1.1.5 while methods state Version v1.1.7
- Package carries a long dependency list (ggplot2, mclust, gtools, tidyr, pheatmap, viridis, dplyr, stringr, reshape2, plotly, STRINGdb, igraph, etc.) that must be satisfied for faithful reproduction

## Evidence
