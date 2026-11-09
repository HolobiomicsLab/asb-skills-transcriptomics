---
name: single-cell-dimension-reduction
description: Use when raw single-cell RNA-seq count data (cells × genes matrix) must
  be compressed to a low-dimensional embedding for visualization, clustering, or as
  input to classification or pseudo-time inference, especially with high dropout and
  technical noise.
when_to_use_negative:
- The input is already a compact low-dimensional embedding or feature table rather
  than a cells × genes count matrix needing compression.
- A pseudo-time trajectory or cell-type label transfer is the primary goal — use scDHA.pt
  or scDHA.class on the existing embedding instead of re-running dimension reduction.
- The scVI ecosystem is required for variational inference with covariate modeling;
  the paper notes scVI alone does not support clustering, visualization, classification,
  and pseudo-time inference within one framework, but scVI may still be preferred
  for its own workflow.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scDHA
  role: 'Implements the two-module hierarchical autoencoder: scDHA main function for
    dimension reduction and clustering, scDHA.vis for 2D visualization of the embedding.'
  repo: github.com/duct317/scDHA
- name: SCANPY
  role: Competing visualization/dimension-reduction baseline benchmarked against scDHA.vis
    via silhouette index.
- name: CIDR
  role: Comparison clustering/dimensionality-reduction method using implicit imputation
    for dropouts; context for when scDHA is preferred.
  repo: VCCRI/CIDR
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:7ec3ab6d154e1c5747c29b3e3c8a40685ca3aea7cada00021e762503979e1e9c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIDR
  - SCANPY
  - scDHA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# single-cell-dimension-reduction

## Summary

Project scRNA-seq data into a low-dimensional latent space using scDHA's two-module hierarchical autoencoder: a non-negative kernel autoencoder that filters insignificant genes/components, followed by a stacked Bayesian autoencoder (modified VAE, default 15-dim latent space) whose compressed space is repeatedly perturbed to learn a generalized representation.

## When to use

Use when raw single-cell RNA-seq count data (cells × genes matrix) must be compressed to a low-dimensional embedding for visualization, clustering, or as input to classification or pseudo-time inference, especially with high dropout and technical noise.

## When NOT to use

- The input is already a compact low-dimensional embedding or feature table rather than a cells × genes count matrix needing compression.
- A pseudo-time trajectory or cell-type label transfer is the primary goal — use scDHA.pt or scDHA.class on the existing embedding instead of re-running dimension reduction.
- The scVI ecosystem is required for variational inference with covariate modeling; the paper notes scVI alone does not support clustering, visualization, classification, and pseudo-time inference within one framework, but scVI may still be preferred for its own workflow.

## Inputs

- scRNA-seq count matrix with rows as cells/samples and columns as genes (e.g. Goolam$data, log2-transformed via log2(data + 1))
- Optional ground-truth cell-type labels for validation (silhouette index)
- Dimension-reduction output (latent embedding) for scDHA.vis visualization

## Outputs

- Low-dimensional latent embedding per cell (default 15 dimensions, e.g. written to scdha_latent_embeddings.csv)
- Clustering assignment (result$cluster from scDHA)
- 2D visualization embedding from scDHA.vis for transcriptome landscape plots

## How to apply

Run scDHA's first module, a non-negative kernel autoencoder, on the log-transformed count matrix (e.g. log2(data + 1) as in the Goolam example) to obtain a non-negative, part-based representation while removing genes or components with insignificant contributions based on the encoder weight distribution. Feed this filtered representation into the second module, the Stacked Bayesian Self-learning Network: a modified VAE with a default 15-dimensional latent space that draws multiple realizations of z via the re-parameterization trick and uses SELU activations. Train with AdamW in two stages — a warm-up phase emphasizing reconstruction loss, then a phase adding KL-divergence loss — and repeatedly perturb the compressed latent space to learn a more generalized representation and reduce neural-network overfitting. Set a seed for reproducibility; for 2D visualization use scDHA.vis on the dimension-reduction output, which achieved the highest average silhouette index across 34 scRNA-seq datasets (one-sided Wilcoxon p = 1.7e-6 vs PCA, t-SNE, UMAP, SCANPY).

## Related tools

- **scDHA** (Implements the two-module hierarchical autoencoder: scDHA main function for dimension reduction and clustering, scDHA.vis for 2D visualization of the embedding.) — github.com/duct317/scDHA
- **SCANPY** (Competing visualization/dimension-reduction baseline benchmarked against scDHA.vis via silhouette index.)
- **CIDR** (Comparison clustering/dimensionality-reduction method using implicit imputation for dropouts; context for when scDHA is preferred.) — VCCRI/CIDR

## Examples

```
library(scDHA); data('Goolam'); data <- t(Goolam$data); data <- log2(data + 1); result <- scDHA(data, seed = 1); cluster <- result$cluster
```

## Evaluation signals

- Output embedding file (scdha_latent_embeddings.csv) exists with exactly one embedding row per input cell and 15 latent dimensions.
- Compressed latent space qualitatively preserves cell-type structure of the input data (e.g. ARI via mclust::adjustedRandIndex on result$cluster vs labels).
- 2D scDHA.vis embedding attains a higher silhouette index against true cell-type labels than PCA, t-SNE, UMAP, and SCANPY embeddings.
- Results are reproducible: re-running with the same seed parameter (e.g. seed = 1) yields the same embedding and clusters.
- First-module output is non-negative and genes/components with insignificant encoder-weight contributions have been removed.

## Limitations

- Accuracy claims are partly stated as expectations ('expected to be more efficient than other autoencoders') rather than demonstrated in the main discussion; comparative scVI results are deferred to Supplementary Section 6 and Fig. 25–32.
- Requires the libtorch backend (torch::install_torch()) since the framework moved from Tensorflow to Torch; Python environment is no longer required but Torch must be installed.
- Reproducibility depends on explicitly setting the seed; results are only reproducible when a seed is set for these functions.

## Evidence

- [intro] The first module is a non-negative kernel autoencoder that provides a non-negative, part-based representation of the data.: "The first module is a non-negative kernel autoencoder that provides a non-negative, part-based representation of the data."
- [abstract] The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed).: "The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed)."
- [abstract] we repeatedly perturb the compressed space to learn a more generalized representation of the data: "we repeatedly perturb the compressed space to learn a more generalized representation of the data"
- [intro] Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation: "Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation"
- [readme] Log transform the data: `data <- log2(data + 1)` / Generating clustering result: `result <- scDHA(data, seed = 1)`: "Generating clustering result: `result <- scDHA(data, seed = 1)`"
- [results] scDHA consistently and significantly outperforms the competing visualization methods, with the highest average silhouette index values (one-sided Wilcoxon test p = 1.7 × 10−6): [evidence span withheld — anchor status: not_found]
