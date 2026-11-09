---
name: log-fold-change-gene-scoring
description: Use when you have a CSV data frame of significantly changed genes with
  expression values / log-fold-change values and need per-gene LFC scores to drive
  OmicInt's GMM clustering or gene-network exploration.
when_to_use_negative:
- Input data is not a CSV data frame containing significantly changed genes and their
  expression/log-fold-change values — the user must ensure data is in the right format
  before scoring
- Goal is integrated scRNA-seq/proteomics expression profiles rather than per-gene
  LFC scoring — use the OmicInt integration pipeline instead
- Gene identifiers are not aligned across omics layers, making per-gene LFC computation
  ambiguous
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: OmicInt
  role: Source of the LFC-based scoring function that is reconstructed or called to
    compute one score per gene from significantly changed genes and their log-fold-change
    values
  repo: https://github.com/AusteKan/OmicInt
- name: R
  role: Runtime language in which the CSV data frame is loaded, the LFC score is computed
    per gene, and lfc_scores.csv is written
- name: tidyr
  role: Declared OmicInt dependency supporting the data-frame reshaping around the
    scoring computation
- name: dplyr
  role: Declared OmicInt dependency supporting per-gene manipulation of the scored
    data frame
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1016/j.ailsci.2021.100025
    title: 'OmicInt package: Exploring omics data and regulatory networks using integrative
      analyses and machine learning'
  source_content_hash: sha256:eded64ff83449a961a9dd4792074c6511078b122569547a22833c7b0697aa0bf
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
  - dplyr
  - reshape2
  - tidyr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# log-fold-change-gene-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the OmicInt LFC-based gene score — a novel scoring function capturing expression and pathology associations — over significantly changed genes supplied as a CSV data frame. The score is the key analytical parameter of OmicInt's Gaussian mixture modelling pipeline for exploring gene networks and interactome complexity.

## When to use

Use when you have a CSV data frame of significantly changed genes with expression values / log-fold-change values and need per-gene LFC scores to drive OmicInt's GMM clustering or gene-network exploration.

## When NOT to use

- Input data is not a CSV data frame containing significantly changed genes and their expression/log-fold-change values — the user must ensure data is in the right format before scoring
- Goal is integrated scRNA-seq/proteomics expression profiles rather than per-gene LFC scoring — use the OmicInt integration pipeline instead
- Gene identifiers are not aligned across omics layers, making per-gene LFC computation ambiguous

## Inputs

- CSV data frame of significantly changed genes with expression values (gene identifiers aligned across omics layers)
- OmicInt v1.1.7 R package installed from github.com/AusteKan/OmicInt with its declared dependencies (tidyr, dplyr, reshape2, mclust, ggplot2, gtools, pheatmap)

## Outputs

- lfc_scores.csv with gene identifier and LFC score columns, one scored row per input gene

## How to apply

Load the gene-expression data frame in CSV format into R and validate that it contains columns for significantly changed genes and their expression values. Reconstruct or call OmicInt's (v1.1.7, github.com/AusteKan/OmicInt) LFC-based scoring function, which explores gene networks from significantly changed genes and their log-fold-change (LFC) values together with predicted interactome complexity. Run the function over all genes so that exactly one LFC score is produced per input gene; this score is the key analytical parameter that the downstream Gaussian mixture modelling pipeline (mclust) uses to classify genes into probability-based clusters. Persist the per-gene scores as a CSV with gene identifier and LFC score columns (e.g. lfc_scores.csv) and verify they match the values obtained by running the original OmicInt package function on the same input.

## Related tools

- **OmicInt** (Source of the LFC-based scoring function that is reconstructed or called to compute one score per gene from significantly changed genes and their log-fold-change values) — https://github.com/AusteKan/OmicInt
- **R** (Runtime language in which the CSV data frame is loaded, the LFC score is computed per gene, and lfc_scores.csv is written)
- **tidyr** (Declared OmicInt dependency supporting the data-frame reshaping around the scoring computation)
- **dplyr** (Declared OmicInt dependency supporting per-gene manipulation of the scored data frame)

## Examples

```
Rscript -e 'df <- read.csv("gene_expression.csv"); scores <- lfc_score(df); write.csv(scores, "lfc_scores.csv", row.names=FALSE)'
```

## Evaluation signals

- lfc_scores.csv exists, parses as valid CSV, and contains exactly one scored row per input gene
- Output has an explicit gene identifier field plus an LFC score column
- Computed LFC scores match values obtained by running the original OmicInt v1.1.7 package function on the same input data frame
- Every significantly changed gene in the input data frame receives a finite LFC score (no dropped or NA rows)
- Scores are reproducible when the same reconstructed function is re-run on the same CSV input

## Limitations

- Users must ensure the supplied data is in the right format — a CSV data frame containing all relevant experimental information — before scoring
- The README badge reports Release V1.1.5 while the methods text specifies v1.1.7, so version semantics of the scoring function may differ between releases
- The scoring function captures expression and pathology associations but is intended to be used together with the Gaussian mixture modelling pipeline, not as a standalone endpoint

## Evidence
