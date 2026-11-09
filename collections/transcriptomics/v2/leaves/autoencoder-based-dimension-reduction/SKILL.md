---
name: autoencoder-based-dimension-reduction
description: Use when scRNA-seq count matrices are noisy with high dropout and you
  need a low-dimensional, part-based cell representation to support unsupervised clustering
  (e.g., ARI benchmarking against SC3, Seurat, CIDR).
when_to_use_negative:
- The analysis goal is differential expression or marker detection on raw counts rather
  than cell segregation into a low-dimensional latent space
- The downstream tool requires its own dimensionality reduction (e.g., scVI-based
  workflows); note scDHA's comparison against scVI is deferred to supplementary material,
  so claims should not be extended blindly
- Data are not high-throughput single-cell expression matrices — generalization to
  other domains (cancer, obesity, aging) is asserted but not demonstrated in the article
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDHA
  role: Implements the non-negative kernel autoencoder and stacked Bayesian autoencoder
    for denoising, dimension reduction, and clustering of scRNA-seq data
  repo: github.com/duct317/scDHA
- name: CIDR
  role: Comparator clustering method used to benchmark the ARI of the autoencoder-derived
    clusters
  repo: VCCRI/CIDR
- name: SC3
  role: Comparator clustering method in the ARI benchmark across 34 scRNA-seq datasets
- name: SEURAT
  role: Comparator clustering method in the ARI benchmark
- name: SINCERA
  role: Comparator clustering method in the ARI benchmark
- name: SCANPY
  role: Comparator clustering method in the ARI benchmark
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:7c9c9cc0c8733b3289438e996bf4f2c577294faeae8c5323d928b98af5e0a878
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIDR
  - SC3
  - SCANPY
  - SEURAT
  - SINCERA
  - scDHA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# autoencoder-based-dimension-reduction

## Summary

A two-module hierarchical autoencoder pipeline (scDHA) that denoises scRNA-seq data with a non-negative kernel autoencoder and projects cells into a compact low-dimensional space via a stacked Bayesian autoencoder, enabling clustering, visualization, classification, and pseudo-time inference.

## When to use

Use when scRNA-seq count matrices are noisy with high dropout and you need a low-dimensional, part-based cell representation to support unsupervised clustering (e.g., ARI benchmarking against SC3, Seurat, CIDR).

## When NOT to use

- The analysis goal is differential expression or marker detection on raw counts rather than cell segregation into a low-dimensional latent space
- The downstream tool requires its own dimensionality reduction (e.g., scVI-based workflows); note scDHA's comparison against scVI is deferred to supplementary material, so claims should not be extended blindly
- Data are not high-throughput single-cell expression matrices — generalization to other domains (cancer, obesity, aging) is asserted but not demonstrated in the article

## Inputs

- scRNA-seq count matrix with rows as cells/samples and columns as genes (e.g., Goolam dataset data matrix)
- Optional known cell-type labels for external validation via ARI
- Random seed for reproducible embeddings

## Outputs

- Low-dimensional compressed representation of cells from the stacked Bayesian autoencoder
- Unsupervised cluster assignments per cell (result$cluster)
- Adjusted Rand index of clusters against known labels

## How to apply

Log-transform the cell-by-gene count matrix (e.g., log2(counts + 1)) and run scDHA's first module, a non-negative kernel autoencoder, which uses the encoder weight distribution to remove genes or components with insignificant contributions to the part-based representation. Feed the filtered representation into the second module, a stacked Bayesian autoencoder (built on a Variational Autoencoder with a Stacked Bayesian Self-learning Network), which projects data onto a compressed low-dimensional space. Repeatedly perturb this compressed space to learn a more generalized representation before clustering. Run clustering with a fixed seed for reproducibility and assess agreement with known cell labels using the adjusted Rand index (mclust::adjustedRandIndex); in the source benchmark scDHA achieved a mean ARI of 0.81 across 34 datasets versus 0.5 for the next-best method, CIDR.

## Related tools

- **scDHA** (Implements the non-negative kernel autoencoder and stacked Bayesian autoencoder for denoising, dimension reduction, and clustering of scRNA-seq data) — github.com/duct317/scDHA
- **CIDR** (Comparator clustering method used to benchmark the ARI of the autoencoder-derived clusters) — VCCRI/CIDR
- **SC3** (Comparator clustering method in the ARI benchmark across 34 scRNA-seq datasets)
- **SEURAT** (Comparator clustering method in the ARI benchmark)
- **SINCERA** (Comparator clustering method in the ARI benchmark)
- **SCANPY** (Comparator clustering method in the ARI benchmark)

## Examples

```
library(scDHA); data <- log2(t(Goolam$data) + 1); result <- scDHA(data, seed = 1); mclust::adjustedRandIndex(result$cluster, Goolam$label)
```

## Evaluation signals

- Mean ARI across benchmark datasets should be ~0.81 for scDHA (vs ~0.5 for CIDR as the second-best method)
- Per-dataset ARI computed with mclust::adjustedRandIndex(cluster, label) should exceed comparator methods on the majority of the 34 datasets
- ARI is low-variability across the 34 datasets, so per-dataset ARI spread should be small
- Re-running with the same seed reproduces identical clusters (result is reproducible by setting seed)

## Limitations

- Accuracy claims are stated as expectations in the discussion ('expected to be more efficient than other autoencoders') rather than demonstrated there
- No explicit limitations or caveats section is acknowledged in the discussion
- Claims of generality to other high-throughput domains (cancer, obesity, aging) are not supported by experiments in the article
- The comparative advantage over scVI is deferred to supplementary materials (Supplementary Section 6, Fig. 25–32) rather than presented in the main discussion

## Evidence

- [abstract] non-negative kernel autoencoder filtering rationale: "The first module is a non-negative kernel autoencoder able to remove genes or components that have insignificant contributions to the part-based representation of the data."
- [abstract] stacked Bayesian autoencoder projection: "The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed)."
- [abstract] perturbation for generalization: "we repeatedly perturb the compressed space to learn a more generalized representation of the data"
- [intro] weight-distribution-based filtering: "Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation"
- [readme] ARI clustering example from README: "Generating clustering result: `result <- scDHA(data, seed = 1)`"
- [results] benchmark result against comparators: [evidence span withheld — anchor status: not_found]
- [discussion] discussion accuracy caveat: "the hierarchical autoencoder and the modified VAE (second module of scDHA) are expected to be more efficient than other autoencoders"
