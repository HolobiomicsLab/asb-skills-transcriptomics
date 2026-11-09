---
name: variational-autoencoder-implementation
description: Use when a scRNA-seq matrix (rows = cells, columns = genes) has already
  been filtered by the non-negative kernel autoencoder and a low-dimensional compressed
  embedding is needed for clustering, visualization, classification, or pseudo-time
  inference.
when_to_use_negative:
- Input is raw unfiltered scRNA-seq counts — the non-negative kernel autoencoder (first
  module) should first remove genes or components with insignificant contributions
- The goal is dropout imputation or ultrafast clustering without a deep-learning stack
  — methods like CIDR use implicit imputation and require only R >= 3.1.0
- A single-cell toolkit with clustering, visualization, classification, and pseudo-time
  already built in is preferred over a learned embedding — the article notes scVI
  alone does not support clustering, visualization, classification, and pseudo-time
  inference
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scDHA
  role: R/Torch package whose main scDHA() function implements the modified VAE (Stacked
    Bayesian Self-learning Network) for dimension reduction and clustering
  repo: github.com/duct317/scDHA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:c5551b401109f13b0fbfcb0c31640369a026e0a53c6913119d5ae80e5de47b8a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scDHA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# variational-autoencoder-implementation

## Summary

Implements the second module of the scDHA pipeline: a Stacked Bayesian Self-learning Network, i.e. a modified Variational Autoencoder that compresses the non-negative kernel-autoencoder representation into a low-dimensional latent space while repeatedly perturbing the compressed space to reduce overfitting and learn a generalized representation.

## When to use

Use when a scRNA-seq matrix (rows = cells, columns = genes) has already been filtered by the non-negative kernel autoencoder and a low-dimensional compressed embedding is needed for clustering, visualization, classification, or pseudo-time inference.

## When NOT to use

- Input is raw unfiltered scRNA-seq counts — the non-negative kernel autoencoder (first module) should first remove genes or components with insignificant contributions
- The goal is dropout imputation or ultrafast clustering without a deep-learning stack — methods like CIDR use implicit imputation and require only R >= 3.1.0
- A single-cell toolkit with clustering, visualization, classification, and pseudo-time already built in is preferred over a learned embedding — the article notes scVI alone does not support clustering, visualization, classification, and pseudo-time inference

## Inputs

- Filtered, non-negative representation from the scDHA non-negative kernel autoencoder (matrix with rows as samples/cells and columns as genes/components)
- Log-transformed scRNA-seq expression matrix (e.g. log2(counts + 1))
- Random seed for reproducibility

## Outputs

- Latent embedding table with one row per input cell and 15 latent dimensions (e.g. scdha_latent_embeddings.csv)
- Cluster assignment per cell (result$cluster) from the scDHA main function

## How to apply

Load the filtered, non-negative representation produced by the first-module non-negative kernel autoencoder. Build the stacked Bayesian autoencoder: a modified VAE with a default 15-dimensional latent space that draws multiple realizations of the latent z via the re-parameterization trick and uses SELU activations. Train with AdamW in two stages — a warm-up phase emphasizing the reconstruction loss, followed by a phase that adds the KL-divergence loss. Repeatedly perturb the compressed latent space during training so the network learns a more generalized representation and neural-network overfitting is reduced. Finally extract one latent embedding per cell and verify the compressed space preserves cell-type structure of the input data.

## Related tools

- **scDHA** (R/Torch package whose main scDHA() function implements the modified VAE (Stacked Bayesian Self-learning Network) for dimension reduction and clustering) — github.com/duct317/scDHA

## Examples

```
result <- scDHA(data, seed = 1); cluster <- result$cluster
```

## Evaluation signals

- Output embedding file exists and contains exactly one row per input cell with 15 latent dimensions
- The compressed space qualitatively preserves the cell-type structure of the input data
- Clustering on the embedding yields a high adjusted Rand index against known labels (e.g. mclust::adjustedRandIndex(cluster, label) on the Goolam dataset)
- Results are reproducible when the same seed is set (the package states 'The result is reproducible by setting seed for these functions')
- scDHA and its second module consistently outperform scVI combined with downstream analysis methods in benchmark comparisons

## Limitations

- No explicit limitations or caveats section is acknowledged in the discussion of the article
- Accuracy claims are stated as expectations rather than demonstrated results in the discussion ('expected to be more efficient than other autoencoders')
- Comparative evaluation against scVI is deferred to supplementary materials (Supplementary Section 6 and Fig. 25–32) rather than presented in the main discussion

## Evidence

- [abstract] stacked Bayesian autoencoder projects onto low-dimensional space: "The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed)."
- [abstract] repeated perturbation for generalized representation: "we repeatedly perturb the compressed space to learn a more generalized representation of the data"
- [intro] non-negative kernel autoencoder as first-module filter: "Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation"
- [readme] scDHA input format and Torch backend: [evidence span withheld — anchor status: external_doc]
- [readme] Torch-based reproducible implementation: "The machine learning framework is changed from Tensorflow to Torch. Torch can be called directly from R. Python environment is no longer required."
