---
name: autoencoder-feature-selection
description: Use when a scRNA-seq expression matrix (cells x genes) is high-dimensional
  and noisy, and you must remove uninformative genes before training a stacked variational
  autoencoder or graph-based clustering.
when_to_use_negative:
- The input is already a low-dimensional feature representation or has previously
  been feature-selected, so a second non-negative-kernel autoencoder pass would be
  redundant.
- The analysis goal is supervised marker-gene selection using known cell-type labels
  rather than unsupervised noise filtering prior to clustering.
- The dataset is very large (hundreds of thousands to millions of cells, e.g. Brain
  1.3M GSE93421) without applying scCAN's Module 3 train/test split — train the autoencoder
  on the 5000-cell training set and classify remaining cells by k-NN instead.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scCAN
  role: R package implementing the non-negative-kernel autoencoder feature-selection
    module (Module 1) plus the downstream stacked VAE and network-fusion clustering
  repo: https://cran.r-project.org/package=scCAN
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:ef4687538b8f3f45ef1adabe95aacc0dd92a9a3dd5dd99f6a586630be7c1c7cc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scCAN
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# autoencoder-feature-selection

## Summary

Feature-filtering step from scCAN's Module 1: a min-max-scaled, non-negative-kernel autoencoder with a 50-dimension bottleneck ranks genes so that only informative features are carried into downstream dimensionality reduction and clustering.

## When to use

Use when a scRNA-seq expression matrix (cells x genes) is high-dimensional and noisy, and you must remove uninformative genes before training a stacked variational autoencoder or graph-based clustering.

## When NOT to use

- The input is already a low-dimensional feature representation or has previously been feature-selected, so a second non-negative-kernel autoencoder pass would be redundant.
- The analysis goal is supervised marker-gene selection using known cell-type labels rather than unsupervised noise filtering prior to clustering.
- The dataset is very large (hundreds of thousands to millions of cells, e.g. Brain 1.3M GSE93421) without applying scCAN's Module 3 train/test split — train the autoencoder on the 5000-cell training set and classify remaining cells by k-NN instead.

## Inputs

- Normalized scRNA-seq expression matrix with cells as rows and genes as columns (e.g. pollen.rds, 301 cells x genes, loaded via readRDS)
- Raw count matrix requiring log2(x+1) transformation when max(data) > 100

## Outputs

- Filtered feature matrix restricted to the top 5000 informative genes (non-zero autoencoder coefficients, highest coefficient variances)
- Latent representation matrix (e.g. written to latents.csv) passed to the stacked VAE and network-fusion clustering

## How to apply

Rescale each cell's expression profile to the 0-1 range with min-max scaling (apply log2(x+1) first if max(data) > 100, as in the scCAN README). Train a one-layer non-negative-kernel autoencoder whose encoder weights are constrained to be non-negative, with a bottleneck of dimension 50; non-negativity makes coefficients interpretable as feature importance. Keep the top 5000 genes that have non-zero coefficients, prioritizing those with the highest coefficient variances as the filtered feature set. Feed this reduced matrix (cells as rows, genes as columns) into the next module — the stacked variational autoencoder with d = 15 latent dimensions — rather than clustering on the raw genes. On large datasets, split cells into a 5000-cell random training set and a testing set, then propagate labels to the rest via k-NN to keep training tractable.

## Related tools

- **scCAN** (R package implementing the non-negative-kernel autoencoder feature-selection module (Module 1) plus the downstream stacked VAE and network-fusion clustering) — https://cran.r-project.org/package=scCAN

## Examples

```
library(scCAN); data <- t(SCE$data); if(max(data)>100) data <- log2(data + 1); result <- scCAN(data, r.seed = 1); cluster <- result$cluster
```

## Evaluation signals

- The filtered feature set contains exactly the top 5000 genes, all with non-zero encoder coefficients and ranked by highest coefficient variance.
- Encoder weights are non-negative by construction; check that no negative coefficients enter the ranking.
- Downstream ARI against published cell_type1 labels should reproduce the Pollen benchmark of 0.92 within +/- 0.05.
- The eigengap-estimated number of clusters from the downstream module matches the known number of Pollen cell types (9).
- Runtime is on the order of the reported 1.29 minutes for the 301-cell Pollen dataset; large deviations suggest misconfigured bottleneck or feature count.

## Limitations

- No software version is stated for scCAN in the article's methods or discussion sections.
- The number of clusters is user-controlled via the eigengap search range (i scanned from 2 to 15); mis-setting this range can over- or under-estimate cell types.
- Ambiguous labels ('zothers', 'dropped', 'not applicable', 'Not available') had to be removed from several benchmark datasets (Chen, Wang, Segerstolpe, Guo, Kanton, Miller) before evaluation.
- Non-negativity ties feature importance to coefficient variance, which the article does not benchmark against alternative feature-selection criteria.

## Evidence

- [intro] Here we introduce scCAN, a single-cell clustering approach that consists of three modules: "Here we introduce scCAN, a single-cell clustering approach that consists of three modules"
- [methods] the second step removes genes that are not informative: "the second step removes genes that are not informative"
- [methods] we rescale the data to a range of 0 to 1: "we rescale the data to a range of 0 to 1"
- [methods] we further reduce the size of the data by conducting an additional step of dimensional reduction using a modified version of Variational Autoencoder (VAE)44: "conducting an additional step of dimensional reduction using a modified version of Variational Autoencoder (VAE)"
- [readme] Add log transformation: if(max(data)>100) data <- log2(data + 1): "Add log transformation: `if(max(data)>100) data <- log2(data + 1)`"
