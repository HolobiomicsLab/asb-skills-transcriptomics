---
name: correlation-of-topic-score-with-log-fold-change
description: Use when a fitted CODAL/MIRA topic (e.g., Topic 14) appears to describe
  a cell identity and its top-associated genes must be checked against HuBMAP ontology
  membership and cell-type log-fold changes.
when_to_use_negative:
- No fitted CODAL/MIRA topic model exists — the beta matrix cannot be extracted before
  topic training, so neither the correlation nor the Fisher test is computable.
- The target cell type lacks ontology gene sets and log-fold-change measurements,
  leaving nothing to enrich or correlate against.
- The goal is batch correction itself rather than biological interpretation of topics
  — use CODAL multi-batch integration as a preprocessing step instead.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: MIRA
  role: Supplies the fitted expression topic model whose beta (topic-gene association)
    matrix provides the Topic 14 gene ranking used for correlation and enrichment
  repo: https://github.com/cistrome/MIRA
- name: CODAL
  role: Batch-disentangling variational topic model that yields interpretable gene
    co-regulation modules (beta matrix) scored against cell-type log-fold changes
  repo: https://mira-multiome.readthedocs.io
- name: SciPy
  role: Runs the one-sided Fisher's exact test (scipy.stats.fisher_exact, alternative='greater')
    and the correlation computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39494-2
    title: Multi-batch single-cell comparative atlas construction by deep learning
      disentanglement
  source_content_hash: sha256:c7b0d45a232ef9956f44128dd5796e394c3df395130409e6de772fb439cbaa89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cistrome/MIRA
  tools:
  - CODAL
  - MIRA
  - SciPy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# correlation-of-topic-score-with-log-fold-change

## Summary

Validates that a CODAL/MIRA expression topic captures a cell identity program by correlating topic-gene association scores from the Ntopics x Ngenes beta matrix with gene log-fold changes in the target cell type, complemented by ontology enrichment (Topic 14 vs HuBMAP CD16+ monocytes, one-sided Fisher p = 1.5e-19).

## When to use

Use when a fitted CODAL/MIRA topic (e.g., Topic 14) appears to describe a cell identity and its top-associated genes must be checked against HuBMAP ontology membership and cell-type log-fold changes.

## When NOT to use

- No fitted CODAL/MIRA topic model exists — the beta matrix cannot be extracted before topic training, so neither the correlation nor the Fisher test is computable.
- The target cell type lacks ontology gene sets and log-fold-change measurements, leaving nothing to enrich or correlate against.
- The goal is batch correction itself rather than biological interpretation of topics — use CODAL multi-batch integration as a preprocessing step instead.

## Inputs

- Fitted CODAL/MIRA expression topic model providing the Ntopics x Ngenes topic association (beta) matrix
- Modeled gene universe (e.g., 3500 highly variable genes selected with minimum dispersion 0.3)
- HuBMAP CD16+ monocyte ontology gene-membership annotation per gene
- Log-fold-change values for genes in CD16+ monocytes (differential expression table)

## Outputs

- topic14_enrichment_results.csv containing the top-200 gene ranking, the one-sided Fisher's exact p-value, and the topic-score-vs-LFC correlation
- 2x2 contingency table of top-200 membership vs HuBMAP CD16+ monocyte ontology membership

## How to apply

Load the fitted CODAL/MIRA expression topic model and extract the Ntopics x Ngenes topic association (beta) matrix over the modeled gene universe (e.g., the 3500 highly variable genes selected at minimum dispersion 0.3). Rank all genes by their weight in the topic of interest (Topic 14) and retain the top 200, following the paper's top-200-genes-per-topic convention. Cross the top-200 set against HuBMAP CD16+ monocyte ontology membership in a 2x2 contingency table and run a one-sided Fisher's exact test (scipy.stats.fisher_exact, alternative='greater'), expecting p ≈ 1.5e-19. Then compute the Spearman/Pearson correlation between Topic 14 association scores and the log-fold-change values of those genes in CD16+ monocytes to reproduce the Fig. 3f, g comparison. Write the top-200 ranking, the Fisher p-value, and the topic-vs-LFC correlation to topic14_enrichment_results.csv.

## Related tools

- **MIRA** (Supplies the fitted expression topic model whose beta (topic-gene association) matrix provides the Topic 14 gene ranking used for correlation and enrichment) — https://github.com/cistrome/MIRA
- **CODAL** (Batch-disentangling variational topic model that yields interpretable gene co-regulation modules (beta matrix) scored against cell-type log-fold changes) — https://mira-multiome.readthedocs.io
- **SciPy** (Runs the one-sided Fisher's exact test (scipy.stats.fisher_exact, alternative='greater') and the correlation computation)

## Examples

```
from scipy.stats import fisher_exact; _, p = fisher_exact([[a, 200-a], [b, N-200-b]], alternative='greater')
```

## Evaluation signals

- One-sided Fisher's exact p-value for the top-200 x HuBMAP CD16+ monocyte contingency table reproduces ≈1.5e-19
- Topic 14 association score vs CD16+ monocyte log-fold-change correlation is positive and matches the trend displayed in Fig. 3f, g
- topic14_enrichment_results.csv contains the top-200 gene ranking, the Fisher p-value, and the topic-vs-LFC correlation
- Top-200 genes are drawn only from the modeled gene universe (e.g., 3500 highly variable genes with minimum dispersion 0.3)
- Contingency-table margins sum to 200 (top set) and to the remaining modeled genes (background set)

## Limitations

- Agreement between topic scores and log-fold change is association, not proof — the authors caution that new batch-confounded or perturbation-induced cell states should be validated using orthogonal biological evidence to rule out under-correction of technical effects.
- The result depends on the fitted model: CODAL training uses Bayesian hyperparameter optimization with trial pruning at rungs after the 8th and 16th epochs, so model refits may not exactly reproduce the beta matrix or the 1.5e-19 p-value.
- The convention fixes a top-200-genes-per-topic cutoff; sensitivity of the enrichment p-value to that threshold is not established.

## Evidence

- [intro] MIRA provides the variational topic model backend: "The CODAL model architecture extends MIRA28, our earlier method for variational topic modeling of single-cell RNA-seq and ATAC-seq data."
- [discussion] Topic modeling yields interpretable gene co-regulation modules: "CODAL’s variational autoencoder-based framework uses topic modeling to infer interpretable gene co-regulation and transcription factor binding modules"
- [methods] Modeled gene universe: 3500 highly variable genes at minimum dispersion 0.3: "highly variable genes found with a minimum dispersion threshold of 0.3, yielding 3500 highly variable genes"
- [readme] CODAL powers inference in all MIRA topic models: [evidence span withheld — anchor status: external_doc]
- [abstract] Topics yield interpretable modules of biological variation: "CODAL improves the representation of RNA-seq and ATAC-seq modalities, yields interpretable modules of biological variation"
