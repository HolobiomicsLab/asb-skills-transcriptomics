---
name: topic-word-probability-estimation
description: Use when you have a scRNA-seq count matrix (genes × cells) and want robust,
  interpretable latent topics via penalized LDA, e.g. reproducing the pLDA vignette
  simulation and exporting topic-word distributions.
when_to_use_negative:
- Data are not scRNA-seq counts where cells can be modeled as documents of gene 'words'
  (the LDA document–word analogy does not fit)
- The goal requires annotated cell-type labels rather than unsupervised latent topics
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: pLDA
  role: Implements the penalized Latent Dirichlet Allocation model; fits the model
    and provides the simulation example vignette
  repo: github.com/wuxiaotiankevin/pLDA
- name: devtools
  role: Installs the pLDA package from GitHub with build_vignettes=TRUE so the simulation
    example is available
- name: R
  role: Runtime environment; pLDA is loaded and run as an R library
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s12561-021-09304-8
    title: Penalized Latent Dirichlet Allocation Model in Single-Cell RNA Sequencing
  source_content_hash: sha256:612643731f42d85a3cb30a4187448f8af53b6adfef90401042a2dc3a62d25976
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - devtools
  - pLDA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# topic-word-probability-estimation

## Summary

Fits a penalized Latent Dirichlet Allocation (pLDA) model to scRNA-seq count data, treating genes as words, cells as documents, and latent biological functions as topics, and exports the estimated topic-word probability matrix and per-cell topic proportions. The penalty increases robustness of the estimation so the fitted topics are interpretable with biological functions.

## When to use

Use when you have a scRNA-seq count matrix (genes × cells) and want robust, interpretable latent topics via penalized LDA, e.g. reproducing the pLDA vignette simulation and exporting topic-word distributions.

## When NOT to use

- Data are not scRNA-seq counts where cells can be modeled as documents of gene 'words' (the LDA document–word analogy does not fit)
- The goal requires annotated cell-type labels rather than unsupervised latent topics

## Inputs

- scRNA-seq count matrix with genes as words (rows) and cells as documents (columns)
- Simulated scRNA-seq count matrix from the pLDA package vignette example

## Outputs

- topic_word_matrix.csv — topic-word probability matrix (genes × topics)
- cell_topic_proportions.csv — per-cell topic proportion matrix (cells × topics)

## How to apply

Install the pLDA package from GitHub with devtools::install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE), then load it with library(pLDA). Follow the simulation example in browseVignettes('pLDA'), which generates a simulated scRNA-seq count matrix in which genes are words, cells are documents, and latent biological functions are topics. Fit the penalized LDA model to the counts, which estimates topic-word probabilities and per-cell topic proportions while a penalty increases the robustness of the estimation. Export the topic-word probability matrix as topic_word_matrix.csv and the per-cell topic proportion matrix as cell_topic_proportions.csv. Validate that both CSV files exist and are non-empty and that their dimensions match the simulated genes/topics and cells/topics from the vignette.

## Related tools

- **pLDA** (Implements the penalized Latent Dirichlet Allocation model; fits the model and provides the simulation example vignette) — github.com/wuxiaotiankevin/pLDA
- **devtools** (Installs the pLDA package from GitHub with build_vignettes=TRUE so the simulation example is available)
- **R** (Runtime environment; pLDA is loaded and run as an R library)

## Examples

```
library(devtools); install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE); library(pLDA); browseVignettes('pLDA')
```

## Evaluation signals

- Both topic_word_matrix.csv and cell_topic_proportions.csv exist and are non-empty
- Dimensions of topic_word_matrix.csv match the number of simulated genes × topics from the vignette example
- Dimensions of cell_topic_proportions.csv match the number of cells × topics from the vignette example
- Topic-word probabilities and per-cell topic proportions are valid distributions (values within [0,1], rows summing to 1 per document/topic as LDA proportions)
- Fitted topics are interpretable with biological functions, as reported by pLDA

## Limitations

- The method is developed specifically for scRNA-seq data; applicability to other count data types is not established in the provided context
- Topic interpretability relies on the penalty increasing estimation robustness; results depend on the vignette simulation setup when reproducing this example
- Package is distributed via GitHub (devtools install) rather than CRAN, and the associated paper is by Wu, Xiaotian, Wu, Hao, Wu, Zhijin (Statistics in Biosciences, 2021)

## Evidence

- [methods] Accessing the simulation example vignette: [evidence span withheld — anchor status: not_found]
