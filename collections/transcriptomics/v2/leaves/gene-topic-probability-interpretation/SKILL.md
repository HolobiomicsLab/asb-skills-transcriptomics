---
name: gene-topic-probability-interpretation
description: Use when you have a cell-by-gene scRNA-seq count matrix and want to decompose
  it into latent topics (biological functions) and obtain interpretable per-topic
  gene-probability distributions.
when_to_use_negative:
- Input is raw sequencing reads or aligned FASTQ/BAM files rather than a cell-by-gene
  count matrix — pLDA models count data with genes as words
- Input is already a reduced representation such as a feature/latent table or normalized
  expression matrix where the gene-count composition has been lost
- The goal is standard LDA on text documents — pLDA is adapted specifically for scRNA-seq
  count data
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: pLDA
  role: Implements the penalized LDA model for scRNA-seq count matrices; provides
    the vignette simulation example and the fitted model object from which the genes
    x topics gene-probability matrix is extracted
  repo: https://github.com/wuxiaotiankevin/pLDA
- name: devtools
  role: Used to install the pLDA package from GitHub with vignettes built (install_github('wuxiaotiankevin/pLDA',
    build_vignettes=TRUE))
  repo: https://github.com/wuxiaotiankevin/pLDA
- name: R
  role: 'Runtime environment: library(pLDA) loads the package; model fitting and extraction
    of the topic-gene probability table are performed in R'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12561-021-09304-8
    title: Penalized Latent Dirichlet Allocation Model in Single-Cell RNA Sequencing
  source_content_hash: sha256:2df920d18804502c418563e8d8e1927a69ffeb9092a8e0ad4a36accde68bfb14
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/wuxiaotiankevin/pLDA
  tools:
  - R
  - devtools
  - pLDA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-topic-probability-interpretation

## Summary

Fits the penalized Latent Dirichlet Allocation (pLDA) model to scRNA-seq count matrices — genes as words, cells as documents, latent biological functions as topics — and extracts the per-topic gene-probability matrix for biological interpretation. The penalty increases robustness of the estimation.

## When to use

Use when you have a cell-by-gene scRNA-seq count matrix and want to decompose it into latent topics (biological functions) and obtain interpretable per-topic gene-probability distributions.

## When NOT to use

- Input is raw sequencing reads or aligned FASTQ/BAM files rather than a cell-by-gene count matrix — pLDA models count data with genes as words
- Input is already a reduced representation such as a feature/latent table or normalized expression matrix where the gene-count composition has been lost
- The goal is standard LDA on text documents — pLDA is adapted specifically for scRNA-seq count data

## Inputs

- Cell-by-gene scRNA-seq count matrix (integer UMI counts)
- Simulation example count matrix from the pLDA package vignette (browseVignettes('pLDA'))

## Outputs

- Fitted pLDA model object
- Per-topic gene-probability matrix (genes x topics), each topic normalized as a probability distribution over genes
- Topic-gene probability table written to CSV (e.g. plda_topic_gene_probabilities.csv)

## How to apply

Install the pLDA package from GitHub with devtools (install_github('wuxiaotiankevin/pLDA', build_vignettes=TRUE)) and load it with library(pLDA). Obtain a cell-by-gene count matrix — e.g. the simulation example count matrix from the pLDA vignettes accessed via browseVignettes('pLDA'). Fit the penalized LDA model to the count matrix, relying on the penalty that increases the robustness of the estimation. From the fitted model object, extract the estimated genes x topics gene-probability matrix, where each topic's probabilities over genes constitute a valid probability distribution. Write the topic-gene probability table (e.g. to plda_topic_gene_probabilities.csv) and interpret topics as biological functions, since topics identified by pLDA are interpretable with biological functions.

## Related tools

- **pLDA** (Implements the penalized LDA model for scRNA-seq count matrices; provides the vignette simulation example and the fitted model object from which the genes x topics gene-probability matrix is extracted) — https://github.com/wuxiaotiankevin/pLDA
- **devtools** (Used to install the pLDA package from GitHub with vignettes built (install_github('wuxiaotiankevin/pLDA', build_vignettes=TRUE))) — https://github.com/wuxiaotiankevin/pLDA
- **R** (Runtime environment: library(pLDA) loads the package; model fitting and extraction of the topic-gene probability table are performed in R)

## Examples

```
library(devtools); install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE); library(pLDA); browseVignettes('pLDA')
```

## Evaluation signals

- plda_topic_gene_probabilities.csv exists and is parseable
- Each topic's gene probabilities are non-negative
- Each topic's gene probabilities are normalized so they constitute a valid probability distribution over genes (e.g. sum to 1 within a topic)
- The output matrix has one column per estimated topic and one row per gene, matching the input count matrix's gene dimension
- Fitting the model succeeds without error on the vignette's simulation example count matrix

## Limitations

- The package is maintained by Xiaotian (Kevin) Wu (contact xiaotian_wu at brown dot edu); the accompanying manuscript was 'In preparation' at the time of the README, so documentation is largely vignette-based
- Topics are interpretable with biological functions, but mapping topics to functions requires downstream interpretation of the topic-gene probabilities
- The method is adapted from the generative probabilistic model LDA originated in natural language processing, so assumptions of that model apply

## Evidence

- [intro] pLDA models the scRNA-seq data by considering genes as words, cells as documents, and latent biological functions as topics, with a penalty that increases the robustness of the estimation.: "pLDA models the scRNA-seq data by considering genes as words, cells as documents, and latent biological functions as topics, with a penalty that increases the robustness of the estimation."
- [intro] The topics identified by pLDA are interpretable with biological functions.: "The topics identified by pLDA are interpretable with biological functions."
- [methods] install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE): "install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE)"
- [readme] Single cell RNA sequencing (scRNA-seq) is a recently developed technology that allows quantification of RNA transcripts at individual cell level.: "Single cell RNA sequencing (scRNA-seq) is a recently developed technology that allows quantification of RNA transcripts at individual cell level, providing cellular level resolution of gene"
