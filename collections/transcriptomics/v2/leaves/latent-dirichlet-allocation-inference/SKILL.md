---
name: latent-dirichlet-allocation-inference
description: Use when you have a scRNA-seq count matrix (cells x genes) and want to
  decompose it into interpretable latent biological function topics via a penalized
  LDA topic model in R.
when_to_use_negative:
- Input is not raw scRNA-seq counts (e.g. already normalized or log-transformed expression
  values), since LDA-style topic models assume multinomial word-count structure over
  genes
- The goal is cell-type clustering rather than decomposition into interpretable latent
  biological function topics
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: pLDA
  role: R package providing the penalized LDA model fitting for scRNA-seq count matrices
    and the simulation example vignette
  repo: github.com/wuxiaotiankevin/pLDA
- name: devtools
  role: Used to install the pLDA package from GitHub with build_vignettes=TRUE
- name: R
  role: Runtime environment in which the pLDA library is loaded and the model is fit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s12561-021-09304-8
    title: Penalized Latent Dirichlet Allocation Model in Single-Cell RNA Sequencing
  source_content_hash: sha256:74ffefc0326a46e0688becf9b0559a02d78a58a4e57a2d7888be8e5bde3b07f1
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - devtools
  - pLDA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# latent-dirichlet-allocation-inference

## Summary

Fits the penalized Latent Dirichlet Allocation (pLDA) model to a cell-by-gene scRNA-seq count matrix, treating genes as words, cells as documents, and latent biological functions as topics, with a penalty that increases the robustness of the estimation.

## When to use

Use when you have a scRNA-seq count matrix (cells x genes) and want to decompose it into interpretable latent biological function topics via a penalized LDA topic model in R.

## When NOT to use

- Input is not raw scRNA-seq counts (e.g. already normalized or log-transformed expression values), since LDA-style topic models assume multinomial word-count structure over genes
- The goal is cell-type clustering rather than decomposition into interpretable latent biological function topics

## Inputs

- cell-by-gene scRNA-seq count matrix (integer counts)
- number of topics / penalty specification for the pLDA fit

## Outputs

- fitted pLDA model object
- topic-gene probability table (genes x topics), e.g. plda_topic_gene_probabilities.csv

## How to apply

Install the pLDA package from GitHub with devtools::install_github('wuxiaotiankevin/pLDA', build_vignettes=TRUE), then load it with library(pLDA). Take the cell-by-gene count matrix as input and fit the penalized LDA model, which adapts the LDA generative probabilistic model from natural language processing to scRNA-seq data: genes are words, cells are documents, and latent biological functions are topics. The penalty applied during fitting is what increases the robustness of the estimation relative to standard LDA. From the fitted model object, extract the estimated per-topic gene-probability matrix (genes x topics) and write it to a table such as plda_topic_gene_probabilities.csv. For a worked simulation example, consult browseVignettes('pLDA').

## Related tools

- **pLDA** (R package providing the penalized LDA model fitting for scRNA-seq count matrices and the simulation example vignette) — github.com/wuxiaotiankevin/pLDA
- **devtools** (Used to install the pLDA package from GitHub with build_vignettes=TRUE)
- **R** (Runtime environment in which the pLDA library is loaded and the model is fit)

## Examples

```
library(devtools); install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE); library(pLDA); browseVignettes('pLDA')
```

## Evaluation signals

- plda_topic_gene_probabilities.csv exists after the run
- Each topic's gene probabilities are non-negative
- Each topic's gene probabilities are normalized so they constitute a valid probability distribution over genes per topic
- The fitted model's topics are interpretable with biological functions

## Limitations

- pLDA is a package in development (the paper was 'in preparation'); it is distributed via GitHub rather than CRAN, so it must be installed with devtools
- Topic interpretability depends on the fitted topics being relatable to biological functions; the penalty increases robustness but the method's output is a probability distribution per topic, not direct functional annotations

## Evidence

- [intro] pLDA models the scRNA-seq data by considering genes as words, cells as documents, and latent biological functions as topics, with a penalty that increases the robustness of the estimation.: "pLDA models the scRNA-seq data by considering genes as words, cells as documents, and latent biological functions as topics, with a penalty that increases the robustness of the estimation."
- [methods] install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE): "install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE)"
- [readme] For a simulation example, see browseVignettes('pLDA'): "For a simulation example, see browseVignettes('pLDA')"
- [intro] The topics identified by pLDA are interpretable with biological functions.: "The topics identified by pLDA are interpretable with biological functions."
