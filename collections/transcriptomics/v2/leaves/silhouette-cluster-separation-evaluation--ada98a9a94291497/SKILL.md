---
name: silhouette-cluster-separation-evaluation
description: Use when comparing manifold projection methods (topoAE, VAE, AE, UMAP)
  on single-cell modalities and you need a class-separation metric for each low-dimensional
  embedding, alongside KL0.01/KL0.1/KL1 density scores.
when_to_use_negative:
- When the goal is alignment of two modalities into a shared latent space (use scTopoGAN's
  run_scTopoGAN instead of per-modality projection evaluation)
- When only density preservation matters and no cell-class labels are available —
  Silhouette cannot be computed without class structure
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Topological Autoencoder
  role: Produces the topoAE 8-dimensional latents whose Silhouette scores are computed
    in the benchmark
  repo: https://github.com/AkashCiel/scTopoGAN
- name: UMAP
  role: Non-AE baseline embedding method; in the study UMAP obtained the highest Silhouette
    scores
  repo: https://github.com/AkashCiel/scTopoGAN
- name: scTopoGAN
  role: Source framework whose TopoAE_benchmark.py script computes the benchmark metrics
    including Silhouette scores
  repo: https://github.com/AkashCiel/scTopoGAN
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad171
    title: 'scTopoGAN: unsupervised manifold alignment of single-cell data'
  source_content_hash: sha256:91fb151fcf49b34f1433f5f7268738cdb0d3f352b0c6bbadb2e54c9540430b3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AkashCiel/scTopoGAN
  tools:
  - Topological Autoencoder
  - UMAP
  - scTopoGAN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# silhouette-cluster-separation-evaluation

## Summary

Computes Silhouette scores on low-dimensional (e.g., 8-dimensional) single-cell embeddings to quantify cell-class separation, complementing KL divergence-based density-preservation metrics in the scTopoGAN topoAE vs. VAE/AE/UMAP benchmark.

## When to use

Use when comparing manifold projection methods (topoAE, VAE, AE, UMAP) on single-cell modalities and you need a class-separation metric for each low-dimensional embedding, alongside KL0.01/KL0.1/KL1 density scores.

## When NOT to use

- When the goal is alignment of two modalities into a shared latent space (use scTopoGAN's run_scTopoGAN instead of per-modality projection evaluation)
- When only density preservation matters and no cell-class labels are available — Silhouette cannot be computed without class structure

## Inputs

- 8-dimensional latent embeddings of each modality (topoAE, VAE, AE, UMAP), e.g. saved as .npy files
- Pre-processed single-cell data from the Data folder (PBMC RNA reduced by PCA to 50 components, PBMC ATAC reduced by LSI to 50 components, BM RNA/ADT matrices)
- Cell class labels per dataset for the Silhouette computation

## Outputs

- Metrics table (topoae_benchmark_results.csv) with rows = dataset x method and columns = KL0.01, KL0.1, KL1, Silhouette
- Per-modality Silhouette scores of the low-dimensional embeddings

## How to apply

For each modality (PBMC RNA PCA-50, PBMC ATAC LSI-50, BM RNA, BM ADT) and each projection method, take the trained 8-dimensional latent embedding and compute its Silhouette score, which measures how well cell classes remain separated in the low-dimensional representation. Compute this metric in the same run as the KL divergence-based density-preservation scores (at regularization weights 0.01, 0.1, and 1) so that both structure-preservation axes are captured per dataset x method. Write all metrics to a single results table (topoae_benchmark_results.csv) with rows = dataset x method and columns = KL0.01, KL0.1, KL1, Silhouette. Interpret Silhouette jointly with the KL scores: in the scTopoGAN study, UMAP obtained the highest Silhouette scores while topoAE achieved the overall lowest KL0.01, so a single metric is insufficient to declare the best method.

## Related tools

- **Topological Autoencoder** (Produces the topoAE 8-dimensional latents whose Silhouette scores are computed in the benchmark) — https://github.com/AkashCiel/scTopoGAN
- **UMAP** (Non-AE baseline embedding method; in the study UMAP obtained the highest Silhouette scores) — https://github.com/AkashCiel/scTopoGAN
- **scTopoGAN** (Source framework whose TopoAE_benchmark.py script computes the benchmark metrics including Silhouette scores) — https://github.com/AkashCiel/scTopoGAN

## Examples

```
python TopoAE_benchmark.py
```

## Evaluation signals

- topoae_benchmark_results.csv exists and contains KL0.01, KL0.1, KL1, and Silhouette fields for all four datasets (PBMC RNA, PBMC ATAC, BM RNA, BM ADT) and all four methods (topoAE, VAE, AE, UMAP)
- All Silhouette values lie within the valid range [-1, 1]
- Each dataset x method row has a non-empty Silhouette score computed from an 8-dimensional latent embedding
- Expected qualitative pattern: UMAP rows tend to show the highest Silhouette scores, while topoAE rows show the lowest KL0.01 (except PBMC ATAC where VAE performed better) — deviations should be investigated rather than silently accepted
- Each modality's trained 8-dimensional latent embeddings are saved as .npy files alongside the metrics table

## Limitations

- Silhouette only measures class separation; it does not capture density preservation, so it must be reported together with KL0.01/KL0.1/KL1 divergence scores
- The benchmark covers specific pre-processed modalities (PBMC RNA PCA-50, PBMC ATAC LSI-50, BM RNA/ADT); Silhouette outcomes on other data types may not follow the same UMAP-highest-Silhouette pattern
- The scTopoGAN repository provides no changelog, so version drift in the benchmark scripts is not traceable

## Evidence

- [readme] TopoAE benchmark scripts and comparison methods: "The ```TopoAE_benchmark.py``` script compares the performance of the Topological Autoencoder with regular Autoencoder, Variational Autoencoder and UMAP."
- [readme] Script input data: "Please note, this script takes as input the pre-processed data provided in the ```Data``` folder."
- [abstract] Per-modality latent representations: "We use topological autoencoders (topoAE) to obtain latent representations of each modality separately."
- [abstract] topoAE structure preservation finding: "the topoAE for individual modalities also showed better performance in preserving the original structure of the data in the low-dimensional representations"
