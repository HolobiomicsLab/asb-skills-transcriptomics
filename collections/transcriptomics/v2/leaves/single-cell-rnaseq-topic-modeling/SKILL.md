---
name: single-cell-rnaseq-topic-modeling
description: Use when the input is a cell-by-gene scRNA-seq count matrix and the goal
  is to decompose expression into interpretable topics corresponding to latent biological
  functions.
when_to_use_negative:
- Input is not raw scRNA-seq count data (e.g., already-normalized or transformed expression
  values), since LDA generatively models count data
- The analysis goal is cell-type clustering or differential expression rather than
  decomposition into latent topic-level biological functions
- Data are bulk RNA-seq or other non-single-cell expression profiles outside the pLDA
  scRNA-seq modeling scope
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: pLDA
  role: R package implementing the penalized Latent Dirichlet Allocation model; used
    to fit the model to the count matrix and extract topic-gene probabilities
  repo: github.com/wuxiaotiankevin/pLDA
- name: devtools
  role: Used to install the pLDA package from GitHub with vignettes built
- name: R
  role: Runtime environment in which the pLDA library is loaded and the model is fitted
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12561-021-09304-8
    title: Penalized Latent Dirichlet Allocation Model in Single-Cell RNA Sequencing
  source_content_hash: sha256:cbd71cfbad7bf85145c8edcbdf36d3d5ab2d0dff08ccbf1a852a2391b4510a49
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - devtools
  - pLDA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# single-cell-rnaseq-topic-modeling

## Summary

Fits a penalized Latent Dirichlet Allocation (pLDA) model to scRNA-seq count matrices, treating genes as words, cells as documents, and latent biological functions as topics, with a penalty that increases robustness of estimation.

## When to use

Use when the input is a cell-by-gene scRNA-seq count matrix and the goal is to decompose expression into interpretable topics corresponding to latent biological functions.

## When NOT to use

- Input is not raw scRNA-seq count data (e.g., already-normalized or transformed expression values), since LDA generatively models count data
- The analysis goal is cell-type clustering or differential expression rather than decomposition into latent topic-level biological functions
- Data are bulk RNA-seq or other non-single-cell expression profiles outside the pLDA scRNA-seq modeling scope

## Inputs

- cell-by-gene scRNA-seq count matrix (genes as words, cells as documents)
- simulation example count matrix from the pLDA package vignettes

## Outputs

- fitted pLDA model object containing per-topic gene-probability estimates (genes x topics matrix)
- topic-gene probability table written to plda_topic_gene_probabilities.csv

## How to apply

Install the pLDA package from GitHub with devtools::install_github('wuxiaotiankevin/pLDA', build_vignettes=TRUE), then load it with library(pLDA). Obtain the cell-by-gene count matrix (e.g., the simulation example from the package vignettes via browseVignettes('pLDA')) and fit the penalized LDA model, which maps genes to words, cells to documents, and latent biological functions to topics, applying a penalty that increases the robustness of the estimation. Extract the estimated per-topic gene-probability matrix (genes x topics) from the fitted model object and write the topic-gene probability table to a CSV file (e.g., plda_topic_gene_probabilities.csv). Topics identified by pLDA are expected to be interpretable with biological functions, so inspect top genes per topic for biological coherence.

## Related tools

- **pLDA** (R package implementing the penalized Latent Dirichlet Allocation model; used to fit the model to the count matrix and extract topic-gene probabilities) — github.com/wuxiaotiankevin/pLDA
- **devtools** (Used to install the pLDA package from GitHub with vignettes built)
- **R** (Runtime environment in which the pLDA library is loaded and the model is fitted)

## Examples

```
library(devtools); install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE); library(pLDA); browseVignettes('pLDA')
```

## Evaluation signals

- plda_topic_gene_probabilities.csv exists after the workflow completes
- Each topic's gene probabilities are non-negative
- Each topic's gene probabilities are normalized so they constitute a valid probability distribution over genes per topic
- The number of topics in the output matrix matches the model specification (genes x topics dimensions)
- Top genes per topic show interpretable association with biological functions

## Limitations

- The pLDA package is installed from GitHub (wuxiaotiankevin/pLDA) rather than CRAN, so installation requires devtools
- The method requires count-level scRNA-seq data, as it is adapted from the generative probabilistic model LDA originated in natural language processing
- As noted in the article, the manuscript was 'In preperation' at the time of the README, so documentation relies primarily on package vignettes

## Evidence
