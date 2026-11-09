---
name: autoencoder-training-convergence-monitoring
description: Use when training the scDHA non-negative kernel autoencoder module on
  a genes-by-cells scRNA-seq matrix and you need to verify reconstruction-loss convergence
  before ranking genes by encoder weight variance.
when_to_use_negative:
- The goal is the second scDHA module (low-dimensional projection via the Stacked
  Bayesian Self-learning Network/VAE) rather than gene filtering in module 1
- The input matrix is already the filtered ~5000-gene set, so re-ranking encoder weights
  would be redundant
- The task is downstream pseudotime or classification (scDHA.pt / scDHA.class), which
  consume the reduced representation rather than train the filtering autoencoder
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDHA
  role: R/Torch package whose first module is the non-negative kernel autoencoder
    trained here to filter genes; installed via CRAN or devtools::install_github('duct317/scDHA')
  repo: github.com/duct317/scDHA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:26a9f3301e423cfa40bd63f0ef571a95fb6aef5a06630cf15885adf91cfc10df
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scDHA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# autoencoder-training-convergence-monitoring

## Summary

Trains scDHA's first-module non-negative kernel autoencoder — a one-layer network with a 50-node bottleneck on min-max/log2-scaled scRNA-seq count matrices — until reconstruction loss converges, producing a part-based representation whose encoder weight distribution drives gene filtering.

## When to use

Use when training the scDHA non-negative kernel autoencoder module on a genes-by-cells scRNA-seq matrix and you need to verify reconstruction-loss convergence before ranking genes by encoder weight variance.

## When NOT to use

- The goal is the second scDHA module (low-dimensional projection via the Stacked Bayesian Self-learning Network/VAE) rather than gene filtering in module 1
- The input matrix is already the filtered ~5000-gene set, so re-ranking encoder weights would be redundant
- The task is downstream pseudotime or classification (scDHA.pt / scDHA.class), which consume the reduced representation rather than train the filtering autoencoder

## Inputs

- Raw scRNA-seq count matrix (genes x cells), e.g. CSV/pandas-loadable, with rows as genes before transpose
- Target gene-set size (~5000 retained genes) and bottleneck size (50 hidden nodes) as workflow parameters

## Outputs

- selected_genes.csv — list of the top ~5000 genes ranked by encoder-weight variance
- scdha_module1_representation.csv — one row per cell with 50 columns of the compressed representation
- Training log showing reconstruction loss decreasing over epochs to convergence

## How to apply

Load the raw scRNA-seq count matrix (genes x cells) and transform it as the scDHA workflow prescribes: log2(counts + 1) per the README's Goolam example, with min-max scaling of each cell's profile to [0, 1]. Build a one-layer autoencoder with a 50-node bottleneck and constrain encoder weights to be non-negative (clamp/rectify weights after each update) so the learned representation is non-negative and part-based. Train with a reconstruction loss and treat the run as converged only when the loss has clearly decreased and stabilized across epochs; scDHA results are reproducible when a seed (e.g. seed = 1) is set. After convergence, rank genes by the variance of their encoder weights across the 50 hidden nodes and retain the top ~5000 genes, saving the gene list to selected_genes.csv and the 50-dimensional cell representation to scdha_module1_representation.csv.

## Related tools

- **scDHA** (R/Torch package whose first module is the non-negative kernel autoencoder trained here to filter genes; installed via CRAN or devtools::install_github('duct317/scDHA')) — github.com/duct317/scDHA

## Examples

```
data <- log2(data + 1); result <- scDHA(data, seed = 1)
```

## Evaluation signals

- Reconstruction loss decreased monotonically (or near-monotonically) over training and plateaued at convergence
- All encoder weights remain non-negative after rectification, yielding a part-based representation
- Exactly ~5000 genes are written to selected_genes.csv
- The compressed representation file has one row per cell and exactly 50 columns
- Re-running with the same seed (e.g. seed = 1) reproduces the same clustering/representation, per the README's reproducibility note

## Limitations

- The article's discussion contains no explicit limitations section, so convergence criteria and gene-count thresholds must be validated empirically by the practitioner
- scRNA-seq technical noise and high dropout rate motivated scDHA; extremely sparse inputs may degrade the autoencoder's reconstruction signal
- The scDHA README notes the framework moved from Tensorflow to Torch (Python environment no longer required); older instructions referencing Tensorflow may not apply
