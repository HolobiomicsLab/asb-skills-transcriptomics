---
name: variational-autoencoder-dimensionality-reduction
description: Use when clustering single-cell RNA-seq data (tens of thousands of genes
  per cell) where dropout noise and high dimensionality make direct clustering unreliable,
  and a compressed latent cell representation is needed before network-fusion clustering.
when_to_use_negative:
- The input matrix is already low-dimensional (e.g. a pre-computed latent embedding
  or feature table rather than a genes-by-cells expression matrix) — the autoencoder
  feature filtering assumes raw high-dimensional gene expression.
- The data is raw/unnormalized count data that has not been log-transformed when max(data)
  > 100 — the pipeline expects normalized input and will not compensate on its own.
- The goal is supervised cell-type classification with known labels, not unsupervised
  clustering — the VAE latent space here is trained without labels.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scCAN
  role: 'R package implementing the full pipeline: min-max rescaling, non-negative-kernel
    autoencoder feature selection (bottleneck 50, top 5000 genes), stacked VAE with
    d=15 latent dimensions, and downstream SNF fusion plus spectral clustering; invoked
    as scCAN(data, r.seed = 1)'
  repo: https://cran.r-project.org/package=scCAN
- name: SHARP
  role: Provides the wMetaC weighted meta-clustering algorithm used to consolidate
    cluster assignments across the multiple latent representations before ARI-based
    selection of the best latent matrix
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-14218-6
    title: 'scCAN: single-cell clustering using autoencoder and network fusion'
  source_content_hash: sha256:9bafda741d2b94677603d0b7dda4f20eaa74a541f964679525e0ee73c2a4b4f9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SHARP
  - scCAN
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# variational-autoencoder-dimensionality-reduction

## Summary

Compresses noisy, high-dimensional scRNA-seq expression matrices into low-dimensional cell representations using scCAN's stacked variational autoencoder (latent d=15), preceded by min-max rescaling and non-negative-kernel autoencoder feature selection (top 5000 informative genes).

## When to use

Use when clustering single-cell RNA-seq data (tens of thousands of genes per cell) where dropout noise and high dimensionality make direct clustering unreliable, and a compressed latent cell representation is needed before network-fusion clustering.

## When NOT to use

- The input matrix is already low-dimensional (e.g. a pre-computed latent embedding or feature table rather than a genes-by-cells expression matrix) — the autoencoder feature filtering assumes raw high-dimensional gene expression.
- The data is raw/unnormalized count data that has not been log-transformed when max(data) > 100 — the pipeline expects normalized input and will not compensate on its own.
- The goal is supervised cell-type classification with known labels, not unsupervised clustering — the VAE latent space here is trained without labels.

## Inputs

- Normalized scRNA-seq expression matrix (rows = cells/samples, columns = genes), e.g. deposited RDS file such as pollen.rds (301 cells)
- Cell count and gene count metadata to decide whether Module 3 big-data split (n > 5000 cells) is needed
- Optional log-transformation flag: apply log2(data + 1) if max(data) > 100 before rescaling

## Outputs

- Multiple latent representation matrices of cells at d = 15 dimensions (e.g. written to latents.csv)
- Per-cell cluster assignments from the fused-network spectral clustering (e.g. cluster_assignments.csv)
- k-NN imputed cluster labels for testing-set cells when the big-data split was applied

## How to apply

First rescale the normalized expression matrix (cells as rows, genes as columns) to the 0–1 range with min-max scaling. Train a 1-layer non-negative-kernel autoencoder with encoder weights constrained non-negative and a bottleneck of dimension 50, then keep the top 5000 genes with non-zero coefficients and highest coefficient variances as the filtered feature set. Train a stacked variational autoencoder on the filtered matrix using the re-parameterization trick z = mu + sigma * N(0,1), with the latent layer limited to d = 15 dimensions, generating multiple latent representation matrices of the cells. For datasets exceeding the 5000-cell threshold, apply Module 3: split into a random training set (samp.size, default 5000; increase to 10,000 for rare cell types) and a testing set, cluster only the training cells, and classify the rest with k-NN. Feed the multiple latent matrices downstream into SNF-style network fusion (Gaussian kernel, sigma=0.5), eigengap cluster-number estimation (i scanned 2 to 15), and spectral clustering.

## Related tools

- **scCAN** (R package implementing the full pipeline: min-max rescaling, non-negative-kernel autoencoder feature selection (bottleneck 50, top 5000 genes), stacked VAE with d=15 latent dimensions, and downstream SNF fusion plus spectral clustering; invoked as scCAN(data, r.seed = 1)) — https://cran.r-project.org/package=scCAN
- **SHARP** (Provides the wMetaC weighted meta-clustering algorithm used to consolidate cluster assignments across the multiple latent representations before ARI-based selection of the best latent matrix)

## Examples

```
if(max(data)>100) data <- log2(data + 1); result <- scCAN(data, r.seed = 1); cluster <- result$cluster
```

## Evaluation signals

- Adjusted Rand index between predicted clusters and published cell_type1 labels matches the reported Pollen benchmark (ARI ≈ 0.92) within ±0.05
- Eigengap-estimated number of clusters matches the known number of true cell types (e.g. 9 for both Pollen and Zilionis)
- Each latent matrix has exactly d = 15 columns and one row per input cell after filtering to the top 5000 genes
- Feature-selection step retains only genes with non-zero encoder coefficients (target: top 5000 by coefficient variance) after the bottleneck-50 kernel autoencoder
- Runtime is on the order of the reported Pollen benchmark (~1.29 minutes) and ARI is stable across r.seed settings

## Limitations

- With the default samp.size=5000 for datasets over 5000 cells, rare cell types can be missed — scCAN failed to separate the rare tRBC cells in the 34,558-cell Zilionis dataset until samp.size was raised to 10,000 (achieving tRBC F1 ≈ 0.9).
- No software version is stated for scCAN or SHARP in the article, so exact reproducibility depends on the package snapshot; set r.seed explicitly to control VAE stochasticity.
- Benchmarks excluded cells with ambiguous labels ('zothers', 'dropped', 'not applicable', 'Not available'), so results on datasets containing such labels need equivalent filtering before comparison.

## Evidence

- [methods] Min-max scaling and VAE rationale: "we further reduce the size of the data by conducting an additional step of dimensional reduction using a modified version of Variational Autoencoder (VAE)44"
- [methods] Min-max rescaling precedes feature selection: [evidence span withheld — anchor status: not_found]
- [intro] Three-module architecture generating multiple representations: "a stacked, variational autoencoder to generate multiple low-dimensional representations"
- [methods] Latent matrices feed SNF-style network fusion: "The network fusion approach is adapted from SNF method45–47"
- [readme] README input orientation and invocation: "Generate clustering result, the input matrix has rows as samples and columns as genes: `result <- scCAN(data, r.seed = 1)`"
- [readme] README log-transformation precondition: "Add log transformation: `if(max(data)>100) data <- log2(data + 1)`"
