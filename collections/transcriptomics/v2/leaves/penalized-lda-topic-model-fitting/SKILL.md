---
name: penalized-lda-topic-model-fitting
description: Use when the goal is to decompose a scRNA-seq count matrix into latent
  topics whose topic-word probabilities and per-cell topic proportions yield biologically
  interpretable structure, via the penalized LDA model adapted from natural language
  processing.
when_to_use_negative:
- The input is not scRNA-seq count data structured as genes-by-cells (e.g. bulk RNA-seq
  or already-clustered/annotated cell type tables)
- The analysis goal requires supervised cell-type classification rather than unsupervised
  latent topic discovery
- The user needs topic assignments for text documents (natural language processing
  LDA) rather than gene expression counts
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: pLDA
  role: R package implementing the penalized Latent Dirichlet Allocation model used
    to fit the scRNA-seq counts and estimate latent topics
  repo: github.com/wuxiaotiankevin/pLDA
- name: devtools
  role: Used to install the pLDA package from GitHub with build_vignettes=TRUE
- name: R
  role: Runtime environment in which the pLDA library is loaded and the model is fitted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s12561-021-09304-8
    title: Penalized Latent Dirichlet Allocation Model in Single-Cell RNA Sequencing
  source_content_hash: sha256:0ce2a1dc605da69fd4072778b4fcec8a79bd0fbac9f34315b7254b08383f7531
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - devtools
  - pLDA
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# penalized-lda-topic-model-fitting

## Summary

Fit a penalized Latent Dirichlet Allocation (pLDA) model to single-cell RNA-seq count matrices using the pLDA R package, treating genes as words, cells as documents, and latent biological functions as topics to obtain interpretable topic-word distributions and per-cell topic proportions.

## When to use

Use when the goal is to decompose a scRNA-seq count matrix into latent topics whose topic-word probabilities and per-cell topic proportions yield biologically interpretable structure, via the penalized LDA model adapted from natural language processing.

## When NOT to use

- The input is not scRNA-seq count data structured as genes-by-cells (e.g. bulk RNA-seq or already-clustered/annotated cell type tables)
- The analysis goal requires supervised cell-type classification rather than unsupervised latent topic discovery
- The user needs topic assignments for text documents (natural language processing LDA) rather than gene expression counts

## Inputs

- Single-cell RNA-seq count matrix (genes as words/rows, cells as documents/columns)
- Simulated scRNA-seq count matrix from the pLDA vignette example

## Outputs

- Topic-word probability matrix (topics x genes), e.g. topic_word_matrix.csv
- Per-cell topic proportion matrix (cells x topics), e.g. cell_topic_proportions.csv
- Biologically interpretable latent topics

## How to apply

Install the pLDA package from GitHub with devtools (install_github('wuxiaotiankevin/pLDA', build_vignettes=TRUE)) so the simulation vignettes are built, then load it with library(pLDA). Prepare an scRNA-seq count matrix in which rows are genes (words) and columns are cells (documents), as modeled by pLDA where latent biological functions are topics and a penalty increases the robustness of the estimation. Fit the penalized LDA model to the counts to estimate topic-word probabilities and per-cell topic proportions; for a worked example, run the simulation code shown via browseVignettes('pLDA'). Export the estimated topic-word probability matrix (e.g., topic_word_matrix.csv) and the per-cell topic proportion matrix (e.g., cell_topic_proportions.csv), checking that dimensions match the number of genes/topics and cells/topics in the fitted model. Judge success by whether the identified topics are interpretable with biological functions, which is the stated goal of the penalty.

## Related tools

- **pLDA** (R package implementing the penalized Latent Dirichlet Allocation model used to fit the scRNA-seq counts and estimate latent topics) — github.com/wuxiaotiankevin/pLDA
- **devtools** (Used to install the pLDA package from GitHub with build_vignettes=TRUE)
- **R** (Runtime environment in which the pLDA library is loaded and the model is fitted)

## Examples

```
library(devtools); install_github("wuxiaotiankevin/pLDA", build_vignettes=TRUE); library(pLDA); browseVignettes('pLDA')
```

## Evaluation signals

- topic_word_matrix.csv and cell_topic_proportions.csv exist and are non-empty
- Dimensions of topic_word_matrix.csv match the number of simulated genes x topics from the vignette example
- Dimensions of cell_topic_proportions.csv match the number of cells x topics from the vignette example
- Topic-word probability rows and per-cell topic proportion rows are interpretable as probability distributions over genes and topics respectively
- The identified topics are interpretable with biological functions

## Limitations

- The method was developed specifically for single-cell mRNA sequencing (scRNA-seq) data; applying it outside that data type is outside the paper's scope
- A penalty is required to increase robustness of the estimation; unpenalized LDA fits on scRNA-seq counts may be less robust
- Interpretability of topics as biological functions depends on the fitted model producing topics that align with biological functions, which must be checked post hoc

## Evidence

- [intro] pLDA models the scRNA-seq data by considering genes as words, cells as documents, and latent biological functions as topics, with a penalty that increases the robustness of the estimation.: "pLDA models the scRNA-seq data by considering genes as words, cells as documents, and latent biological functions as topics, with a penalty that increases the robustness of the estimation."
