---
name: penalized-probabilistic-estimation
description: Use when you have a cell-by-gene scRNA-seq count matrix and want an interpretable
  topic-model decomposition into latent biological functions with robust (penalized)
  estimation.
when_to_use_negative:
- Input is not a count matrix (e.g. normalized or log-transformed expression values),
  since LDA assumes generative count data with genes as words and cells as documents
- The goal is standard topic modeling of text documents rather than scRNA-seq count
  decomposition
- A pretrained or reference-based cell annotation is needed instead of de novo latent
  topic discovery
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: pLDA
  role: R package that implements the penalized LDA model for scRNA-seq count matrices;
    provides model fitting and the simulation example vignette
  repo: github.com/wuxiaotiankevin/pLDA
- name: devtools
  role: Used to install the pLDA package from GitHub with build_vignettes=TRUE
  repo: github.com/wuxiaotiankevin/pLDA
- name: R
  role: Runtime environment in which the pLDA library is loaded and the model is fit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12561-021-09304-8
    title: Penalized Latent Dirichlet Allocation Model in Single-Cell RNA Sequencing
  source_content_hash: sha256:ecd4794123e72c3433f6f93389d8f23afc5be40bc13787bf8beeed146fdb4622
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - devtools
  - pLDA
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# penalized-probabilistic-estimation

## Summary

Fit a penalized Latent Dirichlet Allocation (pLDA) model to single-cell RNA-seq count matrices using the pLDA R package, treating genes as words, cells as documents, and latent biological functions as topics, with a penalty that increases the robustness of the estimation.

## When to use

Use when you have a cell-by-gene scRNA-seq count matrix and want an interpretable topic-model decomposition into latent biological functions with robust (penalized) estimation.

## When NOT to use

- Input is not a count matrix (e.g. normalized or log-transformed expression values), since LDA assumes generative count data with genes as words and cells as documents
- The goal is standard topic modeling of text documents rather than scRNA-seq count decomposition
- A pretrained or reference-based cell annotation is needed instead of de novo latent topic discovery

## Inputs

- Cell-by-gene scRNA-seq count matrix (integer counts per cell per gene)
- R environment with devtools installed

## Outputs

- Fitted pLDA model object with per-cell topic proportions
- Genes x topics per-topic gene-probability matrix (exportable as plda_topic_gene_probabilities.csv)

## How to apply

Install the pLDA package from GitHub with devtools::install_github('wuxiaotiankevin/pLDA', build_vignettes=TRUE) and load it via library(pLDA). Supply a cell-by-gene scRNA-seq count matrix as the input document-term representation (cells = documents, genes = words). Fit the penalized LDA model so that the penalty increases the robustness of the estimation relative to standard LDA. Extract the estimated per-topic gene-probability matrix (genes x topics) from the fitted model object, where each topic corresponds to a latent biological function. Validate that each topic's gene probabilities are non-negative and normalized to form a valid probability distribution over genes.

## Related tools

- **pLDA** (R package that implements the penalized LDA model for scRNA-seq count matrices; provides model fitting and the simulation example vignette) — github.com/wuxiaotiankevin/pLDA
- **devtools** (Used to install the pLDA package from GitHub with build_vignettes=TRUE) — github.com/wuxiaotiankevin/pLDA
- **R** (Runtime environment in which the pLDA library is loaded and the model is fit)

## Examples

```
library(devtools); install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE); library(pLDA); browseVignettes('pLDA')
```

## Evaluation signals

- plda_topic_gene_probabilities.csv exists after the workflow completes
- Each topic's gene probabilities are non-negative
- Each topic's gene probabilities are normalized so they constitute a valid probability distribution over genes per topic
- The fitted model recovers topics that are interpretable with biological functions
- The simulation example from browseVignettes('pLDA') runs successfully as a sanity check

## Limitations

- Interpretability of topics as biological functions must be assessed by the analyst; the model identifies latent functions but does not guarantee a one-to-one mapping to known cell types
- The penalty is what increases robustness of estimation; without it (vanilla LDA), estimates may be less robust on scRNA-seq data
- The paper is noted as 'In preparation' in the README, so documentation relies on the package vignettes

## Evidence
