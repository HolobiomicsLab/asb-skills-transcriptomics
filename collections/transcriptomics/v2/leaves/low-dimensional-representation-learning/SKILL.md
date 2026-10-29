---
name: low-dimensional-representation-learning
description: Use when high-dimensional raw scRNA-seq count matrices (genes x cells)
  with technical noise and dropout need to be compressed into a low-dimensional cell
  representation for clustering, visualization, classification, or trajectory analysis.
when_to_use_negative:
- The scVI package alone may be preferable if only variational-inference dimension
  reduction is needed without integrated clustering/visualization/classification/pseudo-time
  (though scVI itself is not capable of these four tasks).
- Input is not single-cell high-throughput data with dropout/noise issues; the article's
  rationale targets scRNA-seq noise and dropout challenges.
- Only a standalone imputation-based clustering is needed — CIDR handles dropouts
  via implicit imputation rather than autoencoder representation learning.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDHA
  role: 'Primary R package implementing the hierarchical autoencoder: non-negative
    kernel autoencoder gene filtering and stacked Bayesian autoencoder dimension reduction,
    plus clustering, visualization, pseudo-time, and classification functions'
  repo: https://github.com/duct317/scDHA
- name: CIDR
  role: Comparison baseline using implicit imputation for dropout-robust clustering
    and dimensionality reduction
  repo: https://github.com/VCCRI/CIDR
- name: Monocle3
  role: Comparison baseline for trajectory/pseudo-time inference on scRNA-seq data
  repo: https://github.com/cole-trapnell-lab/monocle3
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:42f19e8f1d9bfec9a3ea8fba1b72eabe812a9a0118871abe42408a65ba28dd8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scDHA
  tools:
  - CIDR
  - Monocle3
  - scDHA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# low-dimensional-representation-learning

## Summary

Learn a compact, denoised low-dimensional representation of scRNA-seq data using scDHA's hierarchical autoencoder: a non-negative kernel autoencoder filters insignificant genes/components, and a stacked Bayesian autoencoder (built on a modified VAE) projects cells into a compressed space suitable for clustering, visualization, classification, and pseudo-time inference.

## When to use

Use when high-dimensional raw scRNA-seq count matrices (genes x cells) with technical noise and dropout need to be compressed into a low-dimensional cell representation for clustering, visualization, classification, or trajectory analysis.

## When NOT to use

- The scVI package alone may be preferable if only variational-inference dimension reduction is needed without integrated clustering/visualization/classification/pseudo-time (though scVI itself is not capable of these four tasks).
- Input is not single-cell high-throughput data with dropout/noise issues; the article's rationale targets scRNA-seq noise and dropout challenges.
- Only a standalone imputation-based clustering is needed — CIDR handles dropouts via implicit imputation rather than autoencoder representation learning.

## Inputs

- scRNA-seq count matrix with rows as cells/samples and columns as genes (log-transformed, e.g. log2(data + 1))
- Optional cell labels for supervised classification (scDHA.class)
- Random seed for reproducibility (e.g. seed = 1)

## Outputs

- Low-dimensional compressed representation of cells (e.g. 50-dimensional bottleneck features)
- Cluster assignments per cell (result$cluster from scDHA)
- Selected gene set filtered by encoder weight distribution (~top 5000 genes)
- 2D visualization embeddings (scDHA.vis), pseudo-time values (scDHA.pt), class predictions for new data (scDHA.class)

## How to apply

Start with an scRNA-seq count matrix (rows = samples/cells, columns = genes), log-transform it (e.g. log2(counts + 1) as in the Goolam example), and run the scDHA framework. The first module is a non-negative kernel autoencoder that yields a part-based representation; based on the weight distribution of its encoder, genes or components with insignificant contributions are removed (e.g. rank by encoder-weight variance and retain top ~5000 genes). The second module, a stacked Bayesian autoencoder built upon a Variational Autoencoder, projects the filtered data into a low-dimensional compressed space, and the compressed space is repeatedly perturbed to learn a more generalized representation. Downstream, the representation supports unsupervised clustering (result$cluster from scDHA()), visualization via scDHA.vis, pseudo-time via scDHA.pt, and classification via scDHA.class. Set a seed (e.g. seed = 1) for reproducibility; the implementation runs in R on Torch, so no Python environment is required.

## Related tools

- **scDHA** (Primary R package implementing the hierarchical autoencoder: non-negative kernel autoencoder gene filtering and stacked Bayesian autoencoder dimension reduction, plus clustering, visualization, pseudo-time, and classification functions) — https://github.com/duct317/scDHA
- **CIDR** (Comparison baseline using implicit imputation for dropout-robust clustering and dimensionality reduction) — https://github.com/VCCRI/CIDR
- **Monocle3** (Comparison baseline for trajectory/pseudo-time inference on scRNA-seq data) — https://github.com/cole-trapnell-lab/monocle3

## Examples

```
library(scDHA); data('Goolam'); data <- t(Goolam$data); data <- log2(data + 1); result <- scDHA(data, seed = 1); cluster <- result$cluster
```

## Evaluation signals

- Clustering quality measured by adjusted Rand Index against known labels, e.g. mclust::adjustedRandIndex(cluster, label) on the Goolam dataset should be high for seed = 1
- The compressed representation has one row per cell with a fixed low number of columns (e.g. 50 bottleneck nodes)
- Exactly ~5000 genes retained after filtering by encoder weight distribution, and reconstruction loss decreased during autoencoder training
- Results are reproducible when re-running with the same seed setting
- scDHA should outperform state-of-the-art baselines (SC3, Seurat, SINCERA, CIDR, SCANPY; scVI for dimension reduction) on clustering and visualization tasks

## Limitations

- The scVI package itself is not capable of clustering, visualization, classification, and pseudo-time inference, so comparisons require combining scVI with downstream analysis methods.
- Comparative claims against scVI are partly deferred to supplementary materials (Supplementary Section 6 and Fig. 25–32).
- Generality claims to other domains (cancer, obesity, aging) are stated as expectations in the discussion rather than demonstrated experiments.

## Evidence

- [abstract] First module filters insignificant genes: "The first module is a non-negative kernel autoencoder able to remove genes or components that have insignificant contributions to the part-based representation of the data."
- [intro] Encoder weight distribution drives filtering: "Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation"
- [abstract] Second module compresses the data: "The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed)."
- [abstract] Perturbation for generalized representation: "we repeatedly perturb the compressed space to learn a more generalized representation of the data"
- [readme] README: input format and log transform: [evidence span withheld — anchor status: external_doc]
- [readme] README: reproducibility via seed: [evidence span withheld — anchor status: external_doc]
