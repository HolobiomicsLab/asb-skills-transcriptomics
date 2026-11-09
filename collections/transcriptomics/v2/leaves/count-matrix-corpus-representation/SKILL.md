---
name: count-matrix-corpus-representation
description: Use when a cell-by-gene scRNA-seq count matrix needs to be represented
  as a topic model corpus to discover latent biological functions with pLDA.
when_to_use_negative:
- Input is not raw count data (e.g. normalized/transformed expression values), since
  LDA treats counts as word frequencies per document
- The goal is cell-type clustering rather than discovering latent biological-function
  topics modeled as an LDA corpus
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: pLDA
  role: R package implementing the penalized LDA model that treats genes as words,
    cells as documents, and latent biological functions as topics
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
  source_content_hash: sha256:10aa3a14cc1daaf7cb0ee3e938aba358747c8afc4b35cc79fdec09d2b8d6d015
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
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# count-matrix-corpus-representation

## Summary

Maps a single-cell RNA-seq count matrix onto the Latent Dirichlet Allocation corpus framework — genes as words, cells as documents, latent biological functions as topics — so that a penalized LDA (pLDA) model can extract robust, interpretable topic structure.

## When to use

Use when a cell-by-gene scRNA-seq count matrix needs to be represented as a topic model corpus to discover latent biological functions with pLDA.

## When NOT to use

- Input is not raw count data (e.g. normalized/transformed expression values), since LDA treats counts as word frequencies per document
- The goal is cell-type clustering rather than discovering latent biological-function topics modeled as an LDA corpus

## Inputs

- cell-by-gene scRNA-seq count matrix (genes as words, cells as documents)
- R environment with devtools installed

## Outputs

- fitted pLDA model object with per-topic gene-probability matrix (genes x topics)
- topic-gene probability table (e.g. plda_topic_gene_probabilities.csv)

## How to apply

Take the cell-by-gene UMI count matrix and interpret its dimensions in LDA terms: genes correspond to words, cells correspond to documents, and each latent topic corresponds to a biological function. Install the pLDA package from GitHub with devtools::install_github('wuxiaotiankevin/pLDA', build_vignettes=TRUE), load it with library(pLDA), and consult the simulation example via browseVignettes('pLDA') to confirm the expected matrix orientation. Fit the penalized LDA model, where the penalty increases the robustness of the estimation, then extract the estimated per-topic gene-probability matrix (genes x topics). Validate that each topic's gene probabilities are non-negative and normalized so they form a valid probability distribution over genes per topic.

## Related tools

- **pLDA** (R package implementing the penalized LDA model that treats genes as words, cells as documents, and latent biological functions as topics) — github.com/wuxiaotiankevin/pLDA
- **devtools** (Used to install the pLDA package from GitHub with build_vignettes=TRUE) — github.com/wuxiaotiankevin/pLDA
- **R** (Runtime environment in which the pLDA library is loaded and the model is fit)

## Examples

```
library(devtools); install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE); library(pLDA); browseVignettes('pLDA')
```

## Evaluation signals

- plda_topic_gene_probabilities.csv exists after fitting
- Each topic's gene probabilities are non-negative
- Gene probabilities are normalized so they constitute a valid probability distribution over genes per topic
- Estimated topics are interpretable with biological functions
- Simulation example from browseVignettes('pLDA') runs to completion end-to-end

## Limitations

- The pLDA package is distributed via GitHub install_github rather than CRAN, so installation depends on devtools and vignette building
- Topic interpretation as biological functions relies on the fitted model; the penalty is what increases robustness of estimation, so results may vary without it

## Evidence

- [intro] pLDA models the scRNA-seq data by considering genes as words, cells as documents, and latent biological functions as topics: "pLDA models the scRNA-seq data by considering genes as words, cells as documents, and latent biological functions as topics, with a penalty that increases the robustness of the estimation."
- [methods] install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE): "library(devtools)
install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE)"
- [intro] The topics identified by pLDA are interpretable with biological functions.: "The topics identified by pLDA are interpretable with biological functions."
- [readme] For a simulation example, see browseVignettes('pLDA'): "library(pLDA)
For a simulation example, see
browseVignettes('pLDA')"
