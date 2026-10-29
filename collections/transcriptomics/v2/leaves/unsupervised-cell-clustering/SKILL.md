---
name: unsupervised-cell-clustering
description: Use when clustering raw single-cell RNA-seq count data (genes x cells
  matrix) with unknown cell labels, or when benchmarking scDHA against SC3, SEURAT,
  SINCERA, CIDR, SCANPY, or k-means on datasets with known cell types using ARI.
when_to_use_negative:
- The task is supervised cell classification of new labeled data — scDHA.class, not
  the unsupervised clustering path, is the appropriate function
- The analysis goal is pseudo-time or trajectory inference — use scDHA.pt on the dimension-reduction
  output instead of the clustering output
- The input is already a low-dimensional embedding; re-running the non-negative kernel
  autoencoder for gene/component filtering is unnecessary
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDHA
  role: 'Primary clustering framework: non-negative kernel autoencoder for filtering
    insignificant genes/components plus stacked Bayesian autoencoder for low-dimensional
    projection and unsupervised clustering'
  repo: https://github.com/duct317/scDHA
- name: CIDR
  role: Comparator clustering method using implicit imputation and dimensionality
    reduction, run with default settings; reported second-best with mean ARI of 0.5
  repo: https://github.com/VCCRI/CIDR
- name: SC3
  role: Comparator single-cell clustering method run on the same 34 datasets with
    default settings
- name: SEURAT
  role: Comparator single-cell clustering method run on the same 34 datasets with
    default settings
- name: SINCERA
  role: Comparator single-cell clustering method run on the same 34 datasets with
    default settings
- name: SCANPY
  role: Comparator single-cell clustering method run on the same 34 datasets with
    default settings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:f8d370a37e927fabac7a3b933b76b8d15ce030d4894e2beb5e43cc0b408f0eac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scDHA
  tools:
  - CIDR
  - SC3
  - SCANPY
  - SEURAT
  - SINCERA
  - scDHA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# unsupervised-cell-clustering

## Summary

Unsupervised cell segregation of scRNA-seq data by projecting the expression matrix through scDHA's two-module autoencoder (non-negative kernel autoencoder + stacked Bayesian autoencoder) and assigning clusters, benchmarked against SC3, Seurat, SINCERA, CIDR, Scanpy, and k-means using adjusted Rand index.

## When to use

Use when clustering raw single-cell RNA-seq count data (genes x cells matrix) with unknown cell labels, or when benchmarking scDHA against SC3, SEURAT, SINCERA, CIDR, SCANPY, or k-means on datasets with known cell types using ARI.

## When NOT to use

- The task is supervised cell classification of new labeled data — scDHA.class, not the unsupervised clustering path, is the appropriate function
- The analysis goal is pseudo-time or trajectory inference — use scDHA.pt on the dimension-reduction output instead of the clustering output
- The input is already a low-dimensional embedding; re-running the non-negative kernel autoencoder for gene/component filtering is unnecessary

## Inputs

- scRNA-seq count matrix with rows as cells/samples and columns as genes (e.g., scDHA's Goolam dataset, accessed via data('Goolam'))
- Known cell-type labels for evaluation datasets (e.g., Goolam$label as character vector)
- Optional: the 34 scRNA-seq datasets with known cell types listed in Table 1, retrieved via GEO/SCP accession numbers

## Outputs

- Per-cell unsupervised cluster assignments (result$cluster from scDHA)
- Low-dimensional compressed representation of cells from the stacked Bayesian autoencoder
- Per-dataset adjusted Rand index values against known labels, aggregated into per-method ARI means (target: ~0.81 for scDHA, ~0.5 for CIDR)

## How to apply

Log-transform the input count matrix as log2(counts + 1), with rows as cells/samples and columns as genes, then run the scDHA main function: the first module, a non-negative kernel autoencoder, removes genes or components with insignificant contributions based on the encoder weight distribution, and the second module, a stacked Bayesian autoencoder, projects data onto a low-dimensional compressed space that is repeatedly perturbed to learn a generalized representation. Extract the cluster assignment from the result object (result$cluster). For comparator runs, execute SC3, Seurat, SINCERA, CIDR, and Scanpy with default settings on the same datasets. Evaluate all methods against known cell labels using the adjusted Rand index (mclust::adjustedRandIndex) per dataset, and aggregate to per-method means; scDHA was reported to reach a mean ARI of 0.81 across 34 datasets versus 0.5 for the second-best method, CIDR. Set a seed to make results reproducible.

## Related tools

- **scDHA** (Primary clustering framework: non-negative kernel autoencoder for filtering insignificant genes/components plus stacked Bayesian autoencoder for low-dimensional projection and unsupervised clustering) — https://github.com/duct317/scDHA
- **CIDR** (Comparator clustering method using implicit imputation and dimensionality reduction, run with default settings; reported second-best with mean ARI of 0.5) — https://github.com/VCCRI/CIDR
- **SC3** (Comparator single-cell clustering method run on the same 34 datasets with default settings)
- **SEURAT** (Comparator single-cell clustering method run on the same 34 datasets with default settings)
- **SINCERA** (Comparator single-cell clustering method run on the same 34 datasets with default settings)
- **SCANPY** (Comparator single-cell clustering method run on the same 34 datasets with default settings)

## Examples

```
library(scDHA); data('Goolam'); data <- log2(t(Goolam$data) + 1); result <- scDHA(data, seed = 1); mclust::adjustedRandIndex(result$cluster, as.character(Goolam$label))
```

## Evaluation signals

- Mean adjusted Rand index over the 34 benchmark datasets should approximate 0.81 for scDHA and 0.5 for CIDR when comparators run with defaults
- ARI computed per dataset with mclust::adjustedRandIndex(cluster, known_labels) should be high relative to all six comparators on most individual datasets
- Variability of scDHA's per-dataset ARI should be very low compared to competing methods
- Re-running scDHA with the same seed (e.g., seed = 1) should reproduce identical cluster assignments, since results are reproducible by setting the seed

## Limitations

- Computational challenges from the ever-increasing number of cells, technical noise, and high dropout rate in scRNA-seq data affect unsupervised clustering accuracy
- The comparison against scVI for clustering is deferred to supplementary materials rather than the main text
- Accuracy claims in the discussion are stated as expectations (scDHA 'is expected to be more accurate than existing autoencoders') rather than independently re-demonstrated there

## Evidence

- [results] scDHA achieves an average ARI of 0.81 across all 34 data sets with very low variability, while the second best method, CIDR, has an average ARI of only 0.5.: [evidence span withheld — anchor status: not_found]
- [abstract] non-negative kernel autoencoder removes insignificant genes/components: "The first module is a non-negative kernel autoencoder able to remove genes or components that have insignificant contributions to the part-based representation of the data."
- [abstract] stacked Bayesian autoencoder low-dimensional projection: "The second module is a stacked Bayesian autoencoder that projects the data onto a low-dimensional space (compressed)."
- [readme] README log-transform and ARI calculation with scDHA on Goolam: "Log transform the data: `data <- log2(data + 1)` - Generating clustering result: `result <- scDHA(data, seed = 1)`"
- [readme] README ARI via mclust adjustedRandIndex: "Calculating adjusted Rand Index using mclust package: `mclust::adjustedRandIndex(cluster,label)`"
